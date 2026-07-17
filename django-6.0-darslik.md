# Django 6.0 — To'liq Darslik (Uzbek)

> Manba: [Django rasmiy hujjatlari (docs.djangoproject.com/en/6.0)](https://docs.djangoproject.com/en/6.0/)
> Bu darslik Django 6.0 versiyasiga asoslangan. Django 6.0 — Python 3.12, 3.13 va 3.14 versiyalarini qo'llab-quvvatlaydi (minimal versiya — Python 3.12).

---

## 1-QISM: Kirish, o'rnatish va birinchi loyiha

### 1.1. Django nima?

Django — Python tilida yozilgan, yuqori darajali (high-level) veb-freymvork. U "batteries-included" (o'rnatilgan holda hamma narsa bor) falsafasiga asoslangan: autentifikatsiya, admin panel, ORM (ma'lumotlar bazasi bilan ishlash), forma validatsiyasi, xavfsizlik mexanizmlari — bularning barchasi qutidan chiqishi bilanoq mavjud.

Django **MTV (Model–Template–View)** arxitekturasiga asoslanadi. Bu klassik MVC (Model–View–Controller) ning Django'cha nomlanishi, xolos:

| Klassik MVC | Django MTV | Vazifasi |
|---|---|---|
| Model | Model | Ma'lumotlar bazasi strukturasi va biznes logika |
| Controller | View | So'rovni qabul qilib, javob qaytarish logikasi |
| View | Template | Foydalanuvchiga ko'rsatiladigan HTML qatlami |

Bu yerda chalkashlik chiqmasligi uchun eslab qoling: **Django'da "View" — bu logika (Controller), "Template" — bu HTML (klassik View)**.

### 1.2. O'rnatish

Avval virtual muhit (virtual environment) yaratish tavsiya etiladi — bu loyihaning kutubxonalarini tizimning boshqa loyihalaridan izolyatsiya qiladi:

```bash
python -m venv venv
source venv/bin/activate      # Linux/macOS
venv\Scripts\activate         # Windows

pip install django
```

O'rnatilgan versiyani tekshirish:

```bash
python -m django --version
```

### 1.3. Birinchi loyiha yaratish

```bash
django-admin startproject mysite
cd mysite
```

Bu buyruq quyidagi strukturani yaratadi:

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

- **manage.py** — loyiha bilan komandalar orqali ishlash uchun yordamchi skript (server ishga tushirish, migratsiya qilish va h.k.).
- **settings.py** — loyihaning barcha sozlamalari: ma'lumotlar bazasi, o'rnatilgan ilovalar, til, vaqt zonasi va boshqalar.
- **urls.py** — URL manzillarini view'larga bog'laydigan "yo'l xaritasi".
- **asgi.py / wsgi.py** — production serverlar bilan ishlash uchun kirish nuqtalari (entry point).

Serverni ishga tushirish:

```bash
python manage.py runserver
```

Brauzerda `http://127.0.0.1:8000/` manzilini ochsangiz, Django'ning "muvaffaqiyatli o'rnatildi" sahifasini ko'rasiz.

### 1.4. Ilova (App) yaratish

Django'da **loyiha (project)** — butun sayt, **ilova (app)** — muayyan funksionallikni bajaruvchi mustaqil modul (masalan, blog, do'kon, foydalanuvchilar boshqaruvi). Bitta loyihada bir nechta ilova bo'lishi mumkin:

```bash
python manage.py startapp blog
```

Yaratilgan ilovani `settings.py` faylidagi `INSTALLED_APPS` ro'yxatiga qo'shish shart:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',   # bizning yangi ilovamiz
]
```

**Muhim izoh (mendan):** Ko'p boshlang'ich dasturchilar `startapp` qilingandan so'ng ilovani `INSTALLED_APPS`ga qo'shishni unutib qo'yishadi — natijada modellar migratsiyaga tushmaydi. Bu Django'dagi eng ko'p uchraydigan xatolardan biri, shuning uchun har doim shu qadamni tekshiring.

---

## 2-QISM: Modellar va ORM asoslari

### 2.1. Model nima?

Model — Python klassi orqali ma'lumotlar bazasi jadvalini tasvirlash usuli. Har bir model — bitta jadval, har bir atribut — jadvalning ustuni (column) hisoblanadi. Bu yondashuv **ORM (Object-Relational Mapping)** deb ataladi — siz SQL yozmasdan, oddiy Python kodi orqali ma'lumotlar bazasi bilan ishlaysiz.

`blog/models.py` faylida:

```python
from django.db import models

class Category(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name

class Post(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    body = models.TextField()
    category = models.ForeignKey(Category, on_delete=models.CASCADE, related_name='posts')
    published = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title
```

### 2.2. Eng ko'p ishlatiladigan field turlari

| Field turi | Tavsifi |
|---|---|
| `CharField(max_length=...)` | Qisqa matn (majburiy `max_length`) |
| `TextField()` | Uzun matn, chegarasiz |
| `IntegerField()` | Butun son |
| `DecimalField(max_digits, decimal_places)` | Aniq kasr son (pul uchun ideal) |
| `BooleanField()` | To'g'ri/Noto'g'ri (True/False) |
| `DateField()` / `DateTimeField()` | Sana / Sana+vaqt |
| `EmailField()` | Email validatsiyasi bilan CharField |
| `SlugField()` | URL uchun qulay matn (masalan, `mening-postim`) |
| `ForeignKey(Model, on_delete=...)` | Boshqa modelga ko'p-birga bog'lanish |
| `ManyToManyField(Model)` | Ko'p-ko'pga bog'lanish |
| `OneToOneField(Model, on_delete=...)` | Bir-birga bog'lanish |

`on_delete` parametri majburiy va bog'langan obyekt o'chirilganda nima bo'lishini belgilaydi: `CASCADE` (bog'liqlarni ham o'chiradi), `PROTECT` (o'chirishga yo'l qo'ymaydi), `SET_NULL` (bo'sh qiladi, `null=True` talab qiladi) va h.k.

### 2.3. Migratsiyalar

Model kodini o'zgartirganingizdan so'ng, bu o'zgarishni haqiqiy ma'lumotlar bazasiga qo'llash uchun **migratsiya** kerak bo'ladi. Bu ikki bosqichli jarayon:

```bash
python manage.py makemigrations   # o'zgarishlarni "reja" fayliga yozadi
python manage.py migrate          # rejani ma'lumotlar bazasiga qo'llaydi
```

`makemigrations` — model bilan haqiqiy DB o'rtasidagi farqni aniqlab, `blog/migrations/0001_initial.py` kabi Python fayllarini yaratadi. `migrate` esa shu fayllardagi SQL buyruqlarini bajaradi.

**Nega bu ikki bosqichga bo'lingan?** Chunki migratsiya fayllari versiya nazorati tizimida (git) saqlanadi va jamoa a'zolari bir xil DB strukturasini olishlari uchun xizmat qiladi — bu shunchaki "avtomatik SQL generatori" emas, balki DB tarixini kuzatish mexanizmi.

### 2.4. Django shell orqali sinash

```bash
python manage.py shell
```

```python
from blog.models import Post, Category

c = Category.objects.create(name="Texnologiya")
p = Post.objects.create(title="Birinchi post", slug="birinchi-post", body="Matn...", category=c)

Post.objects.all()
Post.objects.filter(published=True)
Post.objects.get(id=1)
```

---

## 3-QISM: Views va URL routing

### 3.1. URLConf — yo'l xaritasi

Loyihaning asosiy `mysite/urls.py` fayli har bir ilovaning o'z `urls.py`siga yo'naltiradi (bu — **include()** naqshi, katta loyihalarni tartibli saqlash uchun tavsiya etiladi):

```python
# mysite/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('blog/', include('blog.urls')),
]
```

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('', views.post_list, name='post_list'),
    path('<slug:slug>/', views.post_detail, name='post_detail'),
]
```

`<slug:slug>` — **path converter**. Django URLdagi qismni avtomatik tekshiradi va view funksiyaga argument sifatida uzatadi. Eng ko'p ishlatiladiganlari: `str`, `int`, `slug`, `uuid`, `path`.

### 3.2. Function-Based Views (FBV)

```python
from django.shortcuts import render, get_object_or_404

def post_list(request):
    posts = Post.objects.filter(published=True)
    return render(request, 'blog/post_list.html', {'posts': posts})

def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug, published=True)
    return render(request, 'blog/post_detail.html', {'post': post})
```

`get_object_or_404` — obyekt topilmasa, avtomatik 404 xatosini qaytaradi; bu try/except yozishdan qutqaradi.

### 3.3. Class-Based Views (CBV)

Django katta miqdorda tayyor generic view'lar taqdim etadi — takroriy kod (CRUD operatsiyalari) yozishning oldini oladi:

```python
from django.views.generic import ListView, DetailView

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10

    def get_queryset(self):
        return Post.objects.filter(published=True)

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
```

```python
# urls.py
path('', views.PostListView.as_view(), name='post_list'),
path('<slug:slug>/', views.PostDetailView.as_view(), name='post_detail'),
```

**FBV yoki CBV — qaysi birini tanlash kerak?** Oddiy, bir martalik logika uchun FBV ko'proq o'qilishi oson. Standart CRUD (ro'yxat, detail, create, update, delete) uchun CBV kod hajmini sezilarli kamaytiradi, chunki `ListView`, `DetailView`, `CreateView`, `UpdateView`, `DeleteView` kabi klasslar shu ishni deyarli avtomatik bajaradi. Katta loyihalarda ikkalasi birga qo'llaniladi.

---

## 4-QISM: Shablonlar (Templates)

### 4.1. Django Template Language (DTL)

Shablonlar `templates/` papkasida joylashadi. DTL — HTML ichida dinamik ma'lumotni ko'rsatish uchun maxsus sintaksis:

```django
{# blog/templates/blog/post_list.html #}
{% extends "base.html" %}

{% block content %}
  <h1>Barcha postlar</h1>
  <ul>
    {% for post in posts %}
      <li>
        <a href="{% url 'blog:post_detail' post.slug %}">{{ post.title }}</a>
        — {{ post.created_at|date:"d.m.Y" }}
      </li>
    {% empty %}
      <li>Hozircha postlar yo'q.</li>
    {% endfor %}
  </ul>
{% endblock %}
```

- `{{ variable }}` — o'zgaruvchini chiqarish
- `{% tag %}` — mantiqiy operatorlar (`if`, `for`, `block`, `url` va h.k.)
- `|filter` — qiymatni formatlash (`date`, `lower`, `truncatewords`, `default` va h.k.)

### 4.2. Shablon merosxo'rligi (Template Inheritance)

Kod takrorlanishining oldini olish uchun bitta bazaviy shablon yaratiladi, qolganlari undan meros oladi:

```django
{# templates/base.html #}
<!DOCTYPE html>
<html>
<head><title>{% block title %}Mening saytim{% endblock %}</title></head>
<body>
  <nav>...</nav>
  {% block content %}{% endblock %}
  <footer>...</footer>
</body>
</html>
```

### 4.3. Yangilik: Template Partials (Django 6.0)

Django 6.0'da qo'shilgan asosiy yangiliklardan biri — **template partials**. Bu shablon ichida kichik, nomlangan fragmentlarni belgilash va ularni alohida qayta ishlatish imkonini beradi (masalan, HTMX bilan faqat bitta HTML bo'lagini qaytarish kerak bo'lganda juda foydali):

```django
{% partialdef post-card %}
  <div class="card">
    <h3>{{ post.title }}</h3>
    <p>{{ post.body|truncatewords:20 }}</p>
  </div>
{% endpartialdef %}
```

Bu fragmentni boshqa joydan chaqirish yoki alohida so'rovga javob sifatida render qilish mumkin — avvalgi versiyalarda buning uchun butun shablonni alohida faylga chiqarish kerak edi.

### 4.4. Statik fayllar (CSS, JS, rasmlar)

```python
# settings.py
STATIC_URL = 'static/'
```

```django
{% load static %}
<link rel="stylesheet" href="{% static 'blog/style.css' %}">
```

Production muhitida `python manage.py collectstatic` buyrug'i barcha statik fayllarni bitta joyga (`STATIC_ROOT`) yig'adi.

---

## 5-QISM: Formalar va validatsiya

### 5.1. Oddiy Form klassi

```python
# blog/forms.py
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(max_length=100)
    email = forms.EmailField()
    message = forms.CharField(widget=forms.Textarea)
```

### 5.2. ModelForm — modeldan avtomatik forma yasash

Ko'p hollarda forma to'g'ridan-to'g'ri modeldan generatsiya qilinadi — bu maydonlarni ikki marta yozishdan qutqaradi:

```python
class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'slug', 'body', 'category', 'published']
```

### 5.3. View ichida formani qayta ishlash

```python
def contact_view(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            cleaned = form.cleaned_data
            # cleaned['name'], cleaned['email'] ...
            return redirect('blog:post_list')
    else:
        form = ContactForm()
    return render(request, 'blog/contact.html', {'form': form})
```

`is_valid()` chaqirilgach, Django avtomatik ravishda har bir maydonni tekshiradi (masalan, `EmailField` email formatini, `CharField` uzunlikni). Xato bo'lsa, `form.errors` orqali xabarlarni olish mumkin, va shablonda `{{ form }}` yozilganda xatolar avtomatik ko'rsatiladi.

### 5.4. CSRF himoyasi

Har qanday POST formada `{% csrf_token %}` tegi bo'lishi **shart**:

```django
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Yuborish</button>
</form>
```

Bu Cross-Site Request Forgery hujumlaridan himoyalanish uchun Django'ning o'rnatilgan mexanizmi — token bo'lmasa, Django so'rovni rad etadi (403 xatosi).

---

## 6-QISM: Admin panel

Django'ning eng mashhur xususiyatlaridan biri — bir necha qatorlik kod bilan to'liq funksional boshqaruv panelini olish.

### 6.1. Superuser yaratish

```bash
python manage.py createsuperuser
```

`/admin/` manzili orqali kirish mumkin bo'ladi.

### 6.2. Modellarni admin'ga ro'yxatdan o'tkazish

```python
# blog/admin.py
from django.contrib import admin
from .models import Post, Category

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name']

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'category', 'published', 'created_at']
    list_filter = ['published', 'category']
    search_fields = ['title', 'body']
    prepopulated_fields = {'slug': ('title',)}
```

- `list_display` — jadvalda ko'rsatiladigan ustunlar
- `list_filter` — o'ng tomondagi filtr paneli
- `search_fields` — qidiruv maydoni
- `prepopulated_fields` — `slug`ni `title`dan avtomatik generatsiya qilish (JavaScript orqali)

**Muhim izoh (mendan):** Admin panel — bu tez ichki (internal) vositalar yaratish uchun ajoyib, lekin uni to'g'ridan-to'g'ri end-user'lar uchun ochiq interfeys sifatida ishlatish tavsiya etilmaydi; u ko'proq jamoangiz yoki kontent-menejerlar uchun mo'ljallangan.

---

## 7-QISM: Chuqurroq ORM — QuerySet API

### 7.1. Filtrlash va qidiruv

```python
Post.objects.all()
Post.objects.filter(published=True)
Post.objects.exclude(category__name="Arxiv")
Post.objects.get(pk=1)                       # aynan bitta obyekt, topilmasa xato beradi
Post.objects.filter(title__icontains="django")   # katta-kichik harfga sezgir emas
Post.objects.filter(created_at__year=2026)
Post.objects.order_by('-created_at')          # kamayish tartibida
```

`__` (double underscore) — **lookup** sintaksisi: `field__lookup=value`. Eng ko'p ishlatiladiganlar: `exact`, `icontains`, `gt`, `lt`, `gte`, `lte`, `in`, `isnull`, `startswith`.

### 7.2. Bog'lanishlar orqali so'rov (relation lookups)

```python
Post.objects.filter(category__name="Texnologiya")
Category.objects.filter(posts__published=True).distinct()
```

### 7.3. Agregatsiya

```python
from django.db.models import Count, Avg

Category.objects.annotate(post_count=Count('posts'))
Post.objects.aggregate(Avg('id'))
```

### 7.4. QuerySet'larning "lazy" tabiati

Muhim tushuncha: `Post.objects.filter(...)` chaqirilganda ma'lumotlar bazasiga darhol so'rov yuborilmaydi. QuerySet — **lazy** (dangasa) obyekt: u faqat haqiqatan kerak bo'lganda (masalan, `for` sikli, `list()`, `print()`) bajariladi. Bu Django'ga bir nechta `.filter()` va `.exclude()` chaqiruvlarini zanjir (chain) qilib, faqat bitta optimallashtirilgan SQL so'rovi yuborish imkonini beradi:

```python
qs = Post.objects.filter(published=True)   # hali SQL yuborilmagan
qs = qs.filter(category__name="Texnologiya")  # hali ham yuborilmagan
list(qs)   # faqat shu yerda haqiqiy SQL so'rov bajariladi
```

### 7.5. `select_related` va `prefetch_related`

Bog'langan obyektlarni olishda "N+1 muammosi" degan keng tarqalgan xatoning oldini olish uchun:

```python
Post.objects.select_related('category')       # ForeignKey / OneToOne uchun (JOIN orqali)
Post.objects.prefetch_related('tags')          # ManyToMany yoki teskari FK uchun (alohida so'rov)
```

`select_related` bitta SQL JOIN orqali bog'liq obyektni oldindan yuklab oladi; `prefetch_related` esa alohida so'rov yuborib, natijalarni Python darajasida birlashtiradi. Bularsiz, sikl ichida har safar `post.category.name` chaqirilganda alohida SQL so'rov ketadi — bu ishlash tezligini sezilarli pasaytiradi.

---

## 8-QISM: Xavfsizlik, sozlamalar va joylashtirish (Deployment)

### 8.1. `settings.py` asosiy sozlamalari

```python
DEBUG = False                       # production'da HAR DOIM False bo'lishi shart
ALLOWED_HOSTS = ['example.com']     # DEBUG=False bo'lganda majburiy
SECRET_KEY = '...'                  # maxfiy, git'ga tushmasligi kerak (environment variable orqali)
```

**Muhim izoh (mendan):** `DEBUG = True` holatida ishlayotgan sayt xato yuz berganda to'liq stack trace, kod va sozlamalarni ochiq holda ko'rsatadi — bu production serverda katta xavfsizlik teshigi hisoblanadi. Loyihani serverga chiqarishdan oldin bu sozlamani tekshirish — eng birinchi qadam bo'lishi kerak.

### 8.2. Content Security Policy — Django 6.0'dagi yangilik

Django 6.0'da birinchi marta **o'rnatilgan (built-in) Content Security Policy (CSP)** middleware qo'shildi — bu oldin faqat tashqi kutubxonalar (`django-csp`) orqali qilinardi. CSP — XSS va ma'lumot in'ektsiyasi hujumlaridan himoyalanish uchun brauzerga qaysi manbalardan skript/stil yuklash mumkinligini aytadigan HTTP sarlavha:

```python
# settings.py
MIDDLEWARE = [
    # ...
    "django.middleware.csp.ContentSecurityPolicyMiddleware",
]

SECURE_CSP = {
    "DIRECTIVES": {
        "default-src": ["'self'"],
        "script-src": ["'self'", "cdn.example.com"],
    }
}
```

### 8.3. Fon vazifalari (Background Tasks) — Django 6.0'dagi yangilik

Django 6.0 birinchi marta **o'rnatilgan background tasks freymvorkini** taqdim etdi — oldin bu maqsadda odatda Celery kabi tashqi vositalar ishlatilardi. Bu engil vazifalar (masalan, email yuborish) uchun qo'shimcha infratuzilmasiz yechim beradi.

### 8.4. Ma'lumotlar bazasini production uchun sozlash

Standart holatda Django SQLite bilan ishlaydi (kichik loyihalar uchun yetarli), lekin production'da PostgreSQL tavsiya etiladi:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydb',
        'USER': 'myuser',
        'PASSWORD': 'mypassword',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

### 8.5. Joylashtirish (deployment) uchun asosiy qadamlar

1. `DEBUG = False`, `ALLOWED_HOSTS` sozlash
2. `SECRET_KEY`ni environment variable orqali saqlash
3. `python manage.py collectstatic` — statik fayllarni yig'ish
4. WSGI/ASGI server ishlatish (Gunicorn, Daphne/Uvicorn) — `runserver` faqat development uchun, production uchun mos emas
5. Nginx yoki shunga o'xshash reverse proxy orqali statik fayllarni va HTTPS'ni boshqarish
6. `python manage.py check --deploy` — Django'ning o'zi xavfsizlik sozlamalarini avtomatik tekshiradigan buyrug'i

---

## Yakuniy so'z

Ushbu darslik Django'ning asosiy skeleti — loyiha tuzilishidan tortib production'ga chiqarishgacha bo'lgan yo'lni qamrab oldi. Keyingi qadam sifatida quyidagilarni chuqurroq o'rganish tavsiya etiladi: Django REST Framework (API yaratish), autentifikatsiya va ruxsatlar tizimi (`django.contrib.auth`), testlash (`django.test.TestCase`) va signal'lar (`post_save`, `pre_delete` kabi hodisalar).

Rasmiy hujjat: https://docs.djangoproject.com/en/6.0/
