# Django Views: To'liq Darslik

## Mundarija

1. [Kirish: View nima va u nima uchun kerak](#1-kirish)
2. [MVT arxitekturasi va View-ning o'rni](#2-mvt-arxitekturasi)
3. [Function-Based Views (FBV)](#3-function-based-views-fbv)
4. [HttpRequest va HttpResponse obyektlari](#4-httprequest-va-httpresponse)
5. [URL routing bilan bog'lash](#5-url-routing-bilan-boglash)
6. [Templates bilan ishlash — render()](#6-templates-bilan-ishlash)
7. [Class-Based Views (CBV)](#7-class-based-views-cbv)
8. [Generic (tayyor) View klasslari](#8-generic-view-klasslari)
9. [Mixins](#9-mixins)
10. [Decorators](#10-decorators)
11. [FBV vs CBV: qachon qaysi birini tanlash](#11-fbv-vs-cbv-taqqoslash)
12. [Amaliy loyiha: Blog CRUD (FBV va CBV bilan)](#12-amaliy-loyiha-blog-crud)
13. [Xatoliklarni boshqarish (404, 403, 500)](#13-xatoliklarni-boshqarish)
14. [Best Practices](#14-best-practices)
15. [Xulosa](#15-xulosa)

---

## 1. Kirish

**View** — Django'da HTTP so'rovini (`request`) qabul qilib, unga javob (`response`) qaytaruvchi Python funksiyasi yoki klassidir. Foydalanuvchi brauzerda biror URL'ga kirganda, Django o'sha URL'ga mos view'ni chaqiradi, view esa kerakli ma'lumotni (masalan, bazadan) oladi va HTML sahifa, JSON, fayl yoki boshqa turdagi javobni qaytaradi.

Oddiy qilib aytganda:

```
Foydalanuvchi -> URL so'raydi -> Django URL'ni view'ga yo'naltiradi -> View ishlaydi -> Response qaytadi -> Foydalanuvchi ko'radi
```

View — bu Django ilovasining "miya markazi": biznes-logika shu yerda joylashadi (ma'lumotlarni olish, validatsiya qilish, formalarni qayta ishlash va h.k.).

---

## 2. MVT Arxitekturasi

Django **MVT** (Model-View-Template) arxitekturasidan foydalanadi. Bu klassik MVC'ning Django'dagi ko'rinishi:

| Qism | Vazifasi | Klassik MVC'dagi mosi |
|---|---|---|
| **Model** | Ma'lumotlar bazasi tuzilishi va logikasi | Model |
| **View** | So'rovni qayta ishlash, biznes-logika | Controller |
| **Template** | Foydalanuvchiga ko'rinadigan HTML | View |

Ya'ni Django'dagi **View** aslida boshqa freymvorklardagi **Controller**ga to'g'ri keladi — chalkashmaslik uchun buni yodda tutish kerak.

Oqim quyidagicha:

```
urls.py --> views.py --> models.py (ma'lumot kerak bo'lsa)
                |
                v
          templates/*.html --> foydalanuvchiga HTML qaytadi
```

---

## 3. Function-Based Views (FBV)

Eng oddiy view turi — bu oddiy Python funksiyasi. U kamida bitta argument — `request` (HttpRequest obyekti) qabul qiladi va HttpResponse obyektini qaytarishi **shart**.

### 3.1. Eng sodda misol

```python
# views.py
from django.http import HttpResponse

def salom(request):
    return HttpResponse("Salom, Django dunyosi!")
```

Bu yerda:
- `request` — kiruvchi HTTP so'rovi haqidagi barcha ma'lumotni o'zida saqlaydi (GET/POST parametrlar, foydalanuvchi, cookie'lar va h.k.)
- `HttpResponse` — brauzerga qaytariladigan matn/HTML

### 3.2. Dinamik kontent qaytarish

```python
from django.http import HttpResponse
import datetime

def hozirgi_vaqt(request):
    hozir = datetime.datetime.now()
    html = f"<h1>Hozirgi vaqt: {hozir}</h1>"
    return HttpResponse(html)
```

### 3.3. URL parametrlari bilan ishlash

View funksiyasiga URL'dan kelgan parametrlarni qo'shimcha argument sifatida uzatish mumkin:

```python
def maqola_korish(request, maqola_id):
    return HttpResponse(f"Siz {maqola_id}-maqolani ko'ryapsiz.")
```

Bunga mos URL:

```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('maqola/<int:maqola_id>/', views.maqola_korish, name='maqola_korish'),
]
```

`<int:maqola_id>` qismi — **path converter** deb ataladi. Django avtomatik ravishda URL'dagi qiymatni `int` turiga o'giradi va funksiyaga uzatadi. Boshqa converter turlari:

| Converter | Nima ushlaydi | Misol |
|---|---|---|
| `str` | `/` dan tashqari har qanday matn (default) | `<str:nomi>` |
| `int` | Manfiy bo'lmagan butun son | `<int:id>` |
| `slug` | Harflar, raqamlar, `-`, `_` | `<slug:slug>` |
| `uuid` | UUID formatidagi qiymat | `<uuid:id>` |
| `path` | `/` belgisini ham o'z ichiga oladigan matn | `<path:manzil>` |

---

## 4. HttpRequest va HttpResponse

### 4.1. HttpRequest obyekti

Har bir view funksiyasi birinchi argument sifatida `request` obyektini oladi. Uning eng muhim atributlari:

```python
def request_malumotlari(request):
    print(request.method)       # 'GET', 'POST', 'PUT', 'DELETE' ...
    print(request.GET)          # GET parametrlari (QueryDict)
    print(request.POST)         # POST orqali yuborilgan forma ma'lumotlari
    print(request.user)         # Joriy foydalanuvchi (auth yoqilgan bo'lsa)
    print(request.path)         # So'ralgan URL yo'li, masalan: '/blog/1/'
    print(request.headers)      # HTTP headerlar
    print(request.COOKIES)      # Cookie'lar lug'ati
    print(request.FILES)        # Yuklangan fayllar
    return HttpResponse("OK")
```

### 4.2. GET va POST farqini ushlash

```python
def forma_korish(request):
    if request.method == 'POST':
        ism = request.POST.get('ism')
        return HttpResponse(f"Rahmat, {ism}!")
    return HttpResponse("""
        <form method="POST">
            <input type="text" name="ism">
            <button type="submit">Yuborish</button>
        </form>
    """)
```

> **Diqqat:** POST so'rovlarida CSRF himoyasi ishlaydi. Haqiqiy loyihada `{% csrf_token %}` tegini formada ishlatish shart, aks holda Django `403 Forbidden` xatosini qaytaradi.

### 4.3. HttpResponse turlari

Django turli maqsadlar uchun tayyor Response klasslarini beradi:

```python
from django.http import (
    HttpResponse,
    HttpResponseRedirect,
    HttpResponseNotFound,
    JsonResponse,
)

def json_misol(request):
    data = {"ism": "Zafar", "yosh": 22}
    return JsonResponse(data)   # avtomatik Content-Type: application/json

def redirect_misol(request):
    return HttpResponseRedirect('/muvaffaqiyatli/')

def notfound_misol(request):
    return HttpResponseNotFound("Sahifa topilmadi")
```

Amalda `HttpResponseRedirect('/url/')` o'rniga qulayroq `redirect()` funksiyasi ishlatiladi (pastda ko'ramiz).

---

## 5. URL Routing bilan bog'lash

View o'zi ishlashi uchun albatta biror URL manziliga bog'lanishi kerak. Bu `urls.py` faylida amalga oshiriladi.

### 5.1. Loyihaning asosiy urls.py

```python
# project_nomi/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),   # ilova ichidagi urls.py'ga yo'naltirish
]
```

### 5.2. Ilova ichidagi urls.py

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'   # namespace — {% url 'blog:detail' %} kabi ishlatish uchun

urlpatterns = [
    path('', views.maqolalar_royxati, name='royxat'),
    path('<int:pk>/', views.maqola_detail, name='detail'),
    path('yangi/', views.maqola_yaratish, name='yaratish'),
]
```

`name=` parametri juda muhim — u orqali template va Python kodida URL'ni **qattiq yozilgan matn (hardcode)** o'rniga nom bilan chaqirish mumkin bo'ladi:

```python
from django.urls import reverse
url = reverse('blog:detail', args=[5])   # -> '/blog/5/'
```

```html
<!-- template ichida -->
<a href="{% url 'blog:detail' maqola.pk %}">Batafsil</a>
```

Bu yondashuv URL manzili keyinchalik o'zgarsa ham, kodning boshqa joylarini o'zgartirishga hojat qoldirmaydi.

---

## 6. Templates bilan ishlash

Har safar HTML'ni Python string sifatida yozish noqulay. Shu sabab Django **template** tizimidan foydalanadi.

### 6.1. render() funksiyasi

```python
from django.shortcuts import render

def maqolalar_royxati(request):
    maqolalar = ['Birinchi maqola', 'Ikkinchi maqola', 'Uchinchi maqola']
    context = {'maqolalar': maqolalar}
    return render(request, 'blog/royxat.html', context)
```

`render()` uchta ishni birdaniga bajaradi:
1. Template faylini topadi (`templates/blog/royxat.html`)
2. `context` lug'atidagi ma'lumotlarni shablonga uzatadi
3. Natijada tayyor `HttpResponse` obyektini qaytaradi

### 6.2. Mos template fayli

```html
<!-- templates/blog/royxat.html -->
<h1>Maqolalar ro'yxati</h1>
<ul>
{% for maqola in maqolalar %}
    <li>{{ maqola }}</li>
{% endfor %}
</ul>
```

### 6.3. get_object_or_404

Bazadan bitta obyektni olib, topilmasa avtomatik 404 xato qaytarish uchun qulay yordamchi funksiya:

```python
from django.shortcuts import render, get_object_or_404
from .models import Maqola

def maqola_detail(request, pk):
    maqola = get_object_or_404(Maqola, pk=pk)
    return render(request, 'blog/detail.html', {'maqola': maqola})
```

Bu quyidagi kodga teng, lekin ancha qisqa va toza:

```python
try:
    maqola = Maqola.objects.get(pk=pk)
except Maqola.DoesNotExist:
    raise Http404("Maqola topilmadi")
```

---

## 7. Class-Based Views (CBV)

Loyiha kattalashgani sari, bir xil turdagi amallar (ro'yxat ko'rsatish, yaratish, tahrirlash, o'chirish) ko'p marta takrorlanadi. Shu muammoni yechish uchun Django **Class-Based Views** tizimini taklif qiladi — bu yerda view funksiya emas, balki klass sifatida yoziladi va HTTP metodlariga mos klass metodlari bo'ladi.

### 7.1. Eng asosiy `View` klassi

```python
from django.views import View
from django.http import HttpResponse

class SalomView(View):
    def get(self, request):
        return HttpResponse("Bu GET so'roviga javob")

    def post(self, request):
        return HttpResponse("Bu POST so'roviga javob")
```

`urls.py`da CBV `as_view()` metodi orqali ulanadi:

```python
from .views import SalomView

urlpatterns = [
    path('salom/', SalomView.as_view(), name='salom'),
]
```

### 7.2. Nega bu ishlaydi — dispatch mexanizmi

`View` klassining ichida `dispatch()` metodi bor. So'rov kelganda u avtomatik chaqiriladi va `request.method` qiymatiga qarab mos metodga (`get`, `post`, `put`, `delete`, ...) yo'naltiradi:

```python
# Django ichida (soddalashtirilgan)
def dispatch(self, request, *args, **kwargs):
    handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
    return handler(request, *args, **kwargs)
```

Ya'ni GET so'rovi kelsa — `self.get()`, POST kelsa — `self.post()` chaqiriladi. Agar mos metod klassda yozilmagan bo'lsa, Django avtomatik `405 Method Not Allowed` qaytaradi.

### 7.3. FBV kodini CBV'ga o'girish misoli

FBV:

```python
def maqola_detail(request, pk):
    maqola = get_object_or_404(Maqola, pk=pk)
    return render(request, 'blog/detail.html', {'maqola': maqola})
```

CBV:

```python
class MaqolaDetailView(View):
    def get(self, request, pk):
        maqola = get_object_or_404(Maqola, pk=pk)
        return render(request, 'blog/detail.html', {'maqola': maqola})
```

Bu yerda hali unchalik farq sezilmaydi, lekin CBV'ning kuchi — **generic view'lar** orqali ochiladi.

---

## 8. Generic View Klasslari

Django ko'plab keng tarqalgan vazifalar (ro'yxat chiqarish, detail sahifa, forma orqali yaratish/tahrirlash/o'chirish) uchun tayyor CBV klasslarini beradi. Bular `django.views.generic` modulida joylashgan.

### 8.1. TemplateView

Faqat statik shablonni ko'rsatish uchun:

```python
from django.views.generic import TemplateView

class AsosiySahifa(TemplateView):
    template_name = 'blog/asosiy.html'

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['sarlavha'] = 'Bosh sahifa'
        return context
```

### 8.2. ListView — ro'yxat ko'rsatish

```python
from django.views.generic import ListView
from .models import Maqola

class MaqolaListView(ListView):
    model = Maqola
    template_name = 'blog/royxat.html'   # default: blog/maqola_list.html
    context_object_name = 'maqolalar'    # default: object_list
    paginate_by = 10                     # sahifalash (pagination)
    ordering = ['-yaratilgan_sana']

    def get_queryset(self):
        # ixtiyoriy: standart queryset'ni filtrlash mumkin
        return Maqola.objects.filter(chop_etilgan=True)
```

`ListView` ichki mexanizmi:
1. `get_queryset()` chaqiriladi — bazadan obyektlar ro'yxati olinadi
2. Agar `paginate_by` berilgan bo'lsa — natija sahifalarga bo'linadi
3. `context_object_name` bilan berilgan nom ostida template'ga uzatiladi

### 8.3. DetailView — bitta obyektni ko'rsatish

```python
from django.views.generic import DetailView

class MaqolaDetailView(DetailView):
    model = Maqola
    template_name = 'blog/detail.html'
    context_object_name = 'maqola'
```

`DetailView` URL'dan `pk` yoki `slug` parametrini avtomatik oladi va `get_object_or_404` mantig'iga o'xshash tarzda obyektni topadi.

```python
# urls.py
path('<int:pk>/', MaqolaDetailView.as_view(), name='detail'),
```

### 8.4. CreateView — yangi obyekt yaratish

```python
from django.views.generic.edit import CreateView
from django.urls import reverse_lazy
from .models import Maqola

class MaqolaCreateView(CreateView):
    model = Maqola
    fields = ['sarlavha', 'matn', 'chop_etilgan']
    template_name = 'blog/forma.html'
    success_url = reverse_lazy('blog:royxat')
```

> **Nima uchun `reverse_lazy`, `reverse` emas?** Klass darajasidagi atributlar Django ilova yuklanayotganda baholanadi — o'sha paytda URL konfiguratsiyasi hali to'liq tayyor bo'lmasligi mumkin. `reverse_lazy` URL'ni faqat kerak bo'lganda (so'rov kelganda) hisoblaydi.

### 8.5. UpdateView va DeleteView

```python
from django.views.generic.edit import UpdateView, DeleteView

class MaqolaUpdateView(UpdateView):
    model = Maqola
    fields = ['sarlavha', 'matn', 'chop_etilgan']
    template_name = 'blog/forma.html'
    success_url = reverse_lazy('blog:royxat')


class MaqolaDeleteView(DeleteView):
    model = Maqola
    template_name = 'blog/ochirish_tasdiq.html'
    success_url = reverse_lazy('blog:royxat')
```

`DeleteView` ishlash tartibi: GET so'rovida tasdiqlash sahifasini ko'rsatadi, POST so'rovida esa obyektni o'chiradi va `success_url`ga yo'naltiradi.

### 8.6. To'liq CRUD uchun urls.py

```python
from django.urls import path
from .views import (
    MaqolaListView, MaqolaDetailView,
    MaqolaCreateView, MaqolaUpdateView, MaqolaDeleteView,
)

app_name = 'blog'

urlpatterns = [
    path('', MaqolaListView.as_view(), name='royxat'),
    path('<int:pk>/', MaqolaDetailView.as_view(), name='detail'),
    path('yangi/', MaqolaCreateView.as_view(), name='yaratish'),
    path('<int:pk>/tahrirlash/', MaqolaUpdateView.as_view(), name='tahrirlash'),
    path('<int:pk>/ochirish/', MaqolaDeleteView.as_view(), name='ochirish'),
]
```

Shu 5 qatorli kod bilan to'liq CRUD (Create, Read, Update, Delete) funksionalligi tayyor bo'ladi — FBV bilan bu ishni bajarish uchun kamida 40-50 qator kod yozish kerak bo'lardi.

---

## 9. Mixins

**Mixin** — bu CBV'larga qo'shimcha xatti-harakat (funksionallik) qo'shish uchun ishlatiladigan yordamchi klass. Ular ko'p meros olish (multiple inheritance) orqali asosiy view klassiga "aralashtiriladi".

### 9.1. LoginRequiredMixin

Faqat tizimga kirgan foydalanuvchilarga ruxsat berish uchun:

```python
from django.contrib.auth.mixins import LoginRequiredMixin

class MaqolaCreateView(LoginRequiredMixin, CreateView):
    model = Maqola
    fields = ['sarlavha', 'matn']
    login_url = '/login/'
    redirect_field_name = 'keyingi'
```

> **Diqqat: Mixin tartibi muhim!** `LoginRequiredMixin` har doim asosiy generic view klassidan (`CreateView`, `ListView` va h.k.) **oldin** yozilishi kerak, chunki Python MRO (Method Resolution Order) chapdan o'ngga qarab ishlaydi va mixin o'z tekshiruvini asosiy klass ishga tushishidan oldin bajarishi kerak.

### 9.2. UserPassesTestMixin

Murakkabroq ruxsat tekshiruvlari uchun:

```python
from django.contrib.auth.mixins import UserPassesTestMixin

class MaqolaUpdateView(UserPassesTestMixin, UpdateView):
    model = Maqola
    fields = ['sarlavha', 'matn']

    def test_func(self):
        maqola = self.get_object()
        return self.request.user == maqola.muallif   # faqat muallif tahrirlay oladi
```

### 9.3. PermissionRequiredMixin

```python
from django.contrib.auth.mixins import PermissionRequiredMixin

class MaqolaDeleteView(PermissionRequiredMixin, DeleteView):
    model = Maqola
    permission_required = 'blog.delete_maqola'
```

---

## 10. Decorators

FBV'larda ruxsat va cheklovlarni **decorator** yordamida qo'shish odatiy holdir.

### 10.1. login_required

```python
from django.contrib.auth.decorators import login_required

@login_required(login_url='/login/')
def maqola_yaratish(request):
    ...
```

### 10.2. require_http_methods / require_POST

```python
from django.views.decorators.http import require_http_methods, require_POST

@require_http_methods(['GET', 'POST'])
def maqola_forma(request):
    ...

@require_POST
def maqola_ochirish(request, pk):
    ...
```

### 10.3. CBV'da decorator ishlatish — method_decorator

CBV klasslarida oddiy decorator to'g'ridan-to'g'ri ishlamaydi, chunki decorator funksiya kutadi, klass metodini emas. Shu sababli `method_decorator` ishlatiladi:

```python
from django.utils.decorators import method_decorator
from django.contrib.auth.decorators import login_required
from django.views import View

@method_decorator(login_required, name='dispatch')
class MaqolaCreateView(View):
    ...
```

`name='dispatch'` — decorator klassning aynan `dispatch` metodiga qo'llanishini bildiradi, ya'ni har qanday HTTP so'rovi kelishidan oldin tekshiruv ishga tushadi.

> **Amaliy maslahat:** Agar CBV bilan ishlayotgan bo'lsangiz, `method_decorator` o'rniga imkon qadar tayyor **Mixin**lardan foydalaning (`LoginRequiredMixin` va h.k.) — bu Django'ning o'zi tavsiya qiladigan, ancha "Django-native" yondashuv.

---

## 11. FBV vs CBV Taqqoslash

| Jihat | Function-Based View | Class-Based View |
|---|---|---|
| **O'qilishi** | Chiziqli, yuqoridan pastga o'qiladi, tushunish oson | Meros (inheritance) tufayli ba'zan qaysi metod qayerdan kelayotganini kuzatish qiyinroq |
| **Kod takrori** | Ko'p takrorlanadi (CRUD har safar qayta yoziladi) | Generic view'lar orqali kod deyarli yozilmaydi |
| **Moslashuvchanlik** | To'liq erkin — istagan mantiqni yozish oson | Override qilish kerak bo'ladi, ba'zida "sehrli" (implicit) xatti-harakatni tushunish qiyin |
| **Kengaytirish** | Decorator orqali | Mixin orqali (ko'proq qayta ishlatiladigan) |
| **Kimga mos** | Oddiy, bir martalik yoki noodatiy logika uchun | Standart CRUD, takrorlanuvchi patternlar uchun |

**Umumiy tavsiya:**
- Oddiy, bir martalik yoki juda o'ziga xos logika kerak bo'lsa → **FBV**
- Standart CRUD amallari (ro'yxat, detail, yaratish, tahrirlash, o'chirish) kerak bo'lsa → **CBV (generic views)**

Django jamoasi rasmiy hujjatlarida ham shuni ta'kidlaydi: ikkalasi ham "to'g'ri" yechim, tanlov loyihaning ehtiyojiga bog'liq.

---

## 12. Amaliy Loyiha: Blog CRUD

Quyida to'liq ishlaydigan, minimal Blog ilovasi misoli — avval FBV, keyin xuddi shu funksionallik CBV bilan.

### 12.1. models.py

```python
from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse

class Maqola(models.Model):
    sarlavha = models.CharField(max_length=200)
    matn = models.TextField()
    muallif = models.ForeignKey(User, on_delete=models.CASCADE)
    yaratilgan_sana = models.DateTimeField(auto_now_add=True)
    chop_etilgan = models.BooleanField(default=True)

    def __str__(self):
        return self.sarlavha

    def get_absolute_url(self):
        return reverse('blog:detail', kwargs={'pk': self.pk})
```

### 12.2. FBV variant — views_fbv.py

```python
from django.shortcuts import render, get_object_or_404, redirect
from django.contrib.auth.decorators import login_required
from .models import Maqola
from .forms import MaqolaForm

def maqola_royxati(request):
    maqolalar = Maqola.objects.filter(chop_etilgan=True).order_by('-yaratilgan_sana')
    return render(request, 'blog/royxat.html', {'maqolalar': maqolalar})

def maqola_detail(request, pk):
    maqola = get_object_or_404(Maqola, pk=pk)
    return render(request, 'blog/detail.html', {'maqola': maqola})

@login_required
def maqola_yaratish(request):
    if request.method == 'POST':
        form = MaqolaForm(request.POST)
        if form.is_valid():
            yangi = form.save(commit=False)
            yangi.muallif = request.user
            yangi.save()
            return redirect('blog:detail', pk=yangi.pk)
    else:
        form = MaqolaForm()
    return render(request, 'blog/forma.html', {'form': form})

@login_required
def maqola_tahrirlash(request, pk):
    maqola = get_object_or_404(Maqola, pk=pk, muallif=request.user)
    if request.method == 'POST':
        form = MaqolaForm(request.POST, instance=maqola)
        if form.is_valid():
            form.save()
            return redirect('blog:detail', pk=maqola.pk)
    else:
        form = MaqolaForm(instance=maqola)
    return render(request, 'blog/forma.html', {'form': form})

@login_required
def maqola_ochirish(request, pk):
    maqola = get_object_or_404(Maqola, pk=pk, muallif=request.user)
    if request.method == 'POST':
        maqola.delete()
        return redirect('blog:royxat')
    return render(request, 'blog/ochirish_tasdiq.html', {'maqola': maqola})
```

### 12.3. CBV variant — views_cbv.py

```python
from django.views.generic import ListView, DetailView
from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin
from django.urls import reverse_lazy
from .models import Maqola

class MaqolaListView(ListView):
    model = Maqola
    template_name = 'blog/royxat.html'
    context_object_name = 'maqolalar'
    ordering = ['-yaratilgan_sana']

    def get_queryset(self):
        return Maqola.objects.filter(chop_etilgan=True)

class MaqolaDetailView(DetailView):
    model = Maqola
    template_name = 'blog/detail.html'
    context_object_name = 'maqola'

class MaqolaCreateView(LoginRequiredMixin, CreateView):
    model = Maqola
    fields = ['sarlavha', 'matn', 'chop_etilgan']
    template_name = 'blog/forma.html'

    def form_valid(self, form):
        form.instance.muallif = self.request.user
        return super().form_valid(form)

class MaqolaUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    model = Maqola
    fields = ['sarlavha', 'matn', 'chop_etilgan']
    template_name = 'blog/forma.html'

    def test_func(self):
        return self.get_object().muallif == self.request.user

class MaqolaDeleteView(LoginRequiredMixin, UserPassesTestMixin, DeleteView):
    model = Maqola
    template_name = 'blog/ochirish_tasdiq.html'
    success_url = reverse_lazy('blog:royxat')

    def test_func(self):
        return self.get_object().muallif == self.request.user
```

E'tibor bering: CBV variantida `form_valid()` metodi orqali `muallif` maydoni avtomatik joriy foydalanuvchi bilan to'ldiriladi — bu FBV'dagi `commit=False` mantig'iga to'g'ri keladi, lekin ancha ozroq kod bilan.

---

## 13. Xatoliklarni Boshqarish

### 13.1. 404 — topilmadi

```python
from django.http import Http404

def maqola_detail(request, pk):
    try:
        maqola = Maqola.objects.get(pk=pk)
    except Maqola.DoesNotExist:
        raise Http404("Bunday maqola mavjud emas")
    return render(request, 'blog/detail.html', {'maqola': maqola})
```

Amalda bu deyarli har doim `get_object_or_404()` bilan almashtiriladi (7.3-bo'limda ko'rsatilgan).

### 13.2. Maxsus xato sahifalari

Loyiha ildizidagi `urls.py`da (yoki `settings.py`da `DEBUG = False` bo'lganda) Django avtomatik quyidagi shablonlarni izlaydi:

```
templates/404.html   -> sahifa topilmadi
templates/403.html   -> ruxsat yo'q
templates/500.html   -> server xatosi
templates/400.html   -> noto'g'ri so'rov
```

### 13.3. PermissionDenied

```python
from django.core.exceptions import PermissionDenied

def maqola_ochirish(request, pk):
    maqola = get_object_or_404(Maqola, pk=pk)
    if maqola.muallif != request.user:
        raise PermissionDenied("Sizga bu maqolani o'chirishga ruxsat yo'q")
    maqola.delete()
    return redirect('blog:royxat')
```

---

## 14. Best Practices

1. **"Fat models, thin views"** — biznes-logikani iloji boricha model metodlariga yoki alohida service funksiyalarga chiqaring, view'ni faqat "orkestratsiya" (so'rovni qabul qilish, chaqirish, javob qaytarish) uchun ishlating.
2. **`reverse()` / `{% url %}`dan foydalaning** — URL manzillarini hech qachon view yoki template ichida qo'lda yozmang.
3. **`get_object_or_404` va `get_list_or_404`** — bazadan obyekt olishda doim shulardan foydalaning, `try/except` yozishga hojat qoldirmaydi.
4. **CBV uchun generic view'larni tanlang**, faqat generic view yetarli bo'lmasa `View` klassidan qo'lda meros oling.
5. **Permission tekshiruvlarini mixinlar orqali qiling** (`LoginRequiredMixin`, `UserPassesTestMixin`) — decorator emas, chunki mixin CBV bilan tabiiyroq ishlaydi.
6. **`success_url` uchun `reverse_lazy` ishlating**, `reverse` emas — sabab 8.4-bo'limda tushuntirilgan.
7. **View'larni testlang** — Django'ning `TestCase` va `Client` obyektlari orqali har bir view uchun status kod va kontent tekshiruvlarini yozing.
8. **Katta loyihalarda view'larni bo'lib tashlang** — bitta `views.py` fayl juda katta bo'lib ketsa, `views/` papkasiga bo'lib, har bir modelning view'larini alohida faylga chiqaring.

---

## 15. Xulosa

- **View** — Django'da so'rovni qabul qilib, javob qaytaruvchi mantiqiy birlik; MVT arxitekturasida "Controller" vazifasini bajaradi.
- **Function-Based View (FBV)** — oddiy, tushunarli, to'liq nazorat beradi, lekin CRUD kabi standart amallarda ko'p kod takrorlanishiga olib keladi.
- **Class-Based View (CBV)** — ayniqsa **generic view**lar (`ListView`, `DetailView`, `CreateView`, `UpdateView`, `DeleteView`) orqali standart amallarni juda qisqa kod bilan amalga oshirishga yordam beradi.
- **Mixin**lar CBV'ga qo'shimcha xatti-harakat (masalan, autentifikatsiya tekshiruvi) qo'shish uchun ishlatiladi; **decorator**lar esa asosan FBV bilan ishlatiladi (CBV'da `method_decorator` orqali).
- To'g'ri yondashuv — loyihaning ehtiyojiga qarab FBV va CBV'ni **aralashtirib** ishlatish: oddiy/o'ziga xos sahifalar uchun FBV, standart CRUD uchun CBV.

Keyingi qadam sifatida **Django Forms** va **Django REST Framework**'dagi `APIView` / `ViewSet`larni o'rganish tavsiya qilinadi — ular xuddi shu View konsepsiyasining forma va API kontekstidagi davomidir.
