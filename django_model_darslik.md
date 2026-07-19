# Django Model — MVT arxitekturasidagi "M" haqida to'liq darslik

## Mundarija
1. Kirish: MVT va Model o'rni
2. Model nima va u nima uchun kerak
3. Birinchi Model yaratish
4. Field (maydon) turlari
5. Field parametrlari (options)
6. Meta klassi
7. Model metodlari (`__str__`, custom metodlar, `save()` override)
8. Migratsiyalar (Migrations)
9. Munosabatlar (Relationships): ForeignKey, ManyToMany, OneToOne
10. QuerySet va Manager — ORM asoslari
11. Validatsiya (`clean()`, `full_clean()`)
12. Abstract Base Class va meros olish
13. Eng yaxshi amaliyotlar (Best Practices)
14. Xulosa

---

## 1. Kirish: MVT va Model o'rni

Django **MVT** (Model-View-Template) arxitekturasidan foydalanadi. Bu klassik MVC ning Django'ga moslashtirilgan varianti:

- **Model** — ma'lumotlar bilan ishlaydi: bazadagi jadvallar tuzilishini, ular orasidagi munosabatlarni va biznes-logikani belgilaydi.
- **View** — so'rovni qabul qilib, Model orqali ma'lumot oladi va Template'ga uzatadi (siz avval View haqida darslik olgan edingiz).
- **Template** — foydalanuvchiga ko'rsatiladigan HTML'ni shakllantiradi.

Model — bu tizimning "yuragi". U qanday ma'lumot saqlanishini, qanday tekshiruvlardan o'tishini va boshqa ma'lumotlar bilan qanday bog'lanishini belgilaydi. View va Template Model ustida "qurilgan" hisoblanadi.

```
Foydalanuvchi so'rovi → URL → View → Model (baza bilan ishlaydi) → View → Template → Javob
```

---

## 2. Model nima va u nima uchun kerak

Django Model — bu Python klassi bo'lib, u `django.db.models.Model` klassidan meros oladi. Har bir Model odatda bitta baza jadvaliga mos keladi:

- Model klassi → baza jadvali
- Model atributi (field) → jadval ustuni
- Model instansi → jadvaldagi bitta qator (row)

Bu yondashuv **ORM** (Object-Relational Mapping) deb ataladi — siz SQL yozish o'rniga Python kodi orqali baza bilan ishlaysiz. Django ORM sizning Python kodingizni avtomatik ravishda SQL so'rovlarga aylantiradi, shu bilan birga PostgreSQL, MySQL, SQLite, Oracle kabi turli bazalar bilan bir xil kod orqali ishlash imkonini beradi.

**Nima uchun ORM kerak?**
- SQL ni qo'lda yozish shart emas — kamroq xato.
- Baza turini almashtirish oson (masalan, SQLite'dan PostgreSQL'ga).
- Python obyektlari orqali ishlash — kod o'qilishi osonroq.
- Xavfsizlik: SQL Injection kabi hujumlardan avtomatik himoya.

---

## 3. Birinchi Model yaratish

Har bir Django ilovasi (`app`) o'zining `models.py` fayliga ega bo'ladi. Kitob do'koni misolida:

```python
# books/models.py
from django.db import models

class Author(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)
    birth_date = models.DateField(null=True, blank=True)

    def __str__(self):
        return f"{self.first_name} {self.last_name}"


class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name="books")
    price = models.DecimalField(max_digits=8, decimal_places=2)
    published_date = models.DateField()
    is_available = models.BooleanField(default=True)

    def __str__(self):
        return self.title
```

Bu yerda ikkita Model bor: `Author` va `Book`. `Book` modeli `Author` bilan **ForeignKey** orqali bog'langan — bu "bitta muallif ko'p kitob yozishi mumkin" degan munosabatni bildiradi.

Modelni yaratgandan so'ng, uni albatta **migratsiya** qilish kerak (buni pastda ko'rib chiqamiz), aks holda baza jadvali yaratilmaydi.

---

## 4. Field (maydon) turlari

Django juda ko'p tayyor field turlarini taqdim etadi. Eng ko'p ishlatiladiganlari:

### Matn va son turlari
| Field | Tavsifi |
|---|---|
| `CharField(max_length=N)` | Qisqa matn (majburiy `max_length`) |
| `TextField()` | Uzun matn, cheklovsiz |
| `IntegerField()` | Butun son |
| `FloatField()` | Kasr son (float) |
| `DecimalField(max_digits, decimal_places)` | Aniq kasr son (pul uchun tavsiya etiladi) |
| `BooleanField()` | True/False |
| `SlugField()` | URL-ga mos matn (masalan, `my-article-title`) |
| `EmailField()` | Email formatini tekshiradi |
| `URLField()` | URL formatini tekshiradi |

### Sana va vaqt turlari
| Field | Tavsifi |
|---|---|
| `DateField()` | Faqat sana |
| `DateTimeField()` | Sana + vaqt |
| `TimeField()` | Faqat vaqt |
| `DateTimeField(auto_now_add=True)` | Obyekt yaratilgandagi vaqtni avtomatik saqlaydi |
| `DateTimeField(auto_now=True)` | Har safar saqlanganda vaqtni yangilaydi |

### Fayl va rasm
| Field | Tavsifi |
|---|---|
| `FileField(upload_to='files/')` | Fayl yuklash |
| `ImageField(upload_to='images/')` | Rasm yuklash (Pillow kutubxonasi kerak) |

### Munosabat turlari (keyingi bo'limda batafsil)
| Field | Tavsifi |
|---|---|
| `ForeignKey` | Ko'p-ga-bir (many-to-one) |
| `ManyToManyField` | Ko'p-ga-ko'p (many-to-many) |
| `OneToOneField` | Bir-ga-bir (one-to-one) |

**Misol — turli field'larni birlashtirib ishlatish:**

```python
class Product(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.PositiveIntegerField(default=0)
    image = models.ImageField(upload_to="products/", blank=True, null=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

---

## 5. Field parametrlari (options)

Har bir field o'zining xatti-harakatini boshqaradigan parametrlarni qabul qiladi:

- **`null=True`** — bazada bu ustun `NULL` qiymatni qabul qiladimi (baza darajasida).
- **`blank=True`** — forma validatsiyasida bu maydon bo'sh qoldirilishi mumkinmi (Python/forma darajasida).
- **`default=value`** — standart qiymat.
- **`unique=True`** — bu ustundagi barcha qiymatlar takrorlanmasligi kerak.
- **`choices=[...]`** — maydon uchun tanlov ro'yxati beradi.
- **`db_index=True`** — bu ustunga indeks qo'shadi (qidiruvni tezlashtiradi).
- **`verbose_name="..."`** — admin panelda va formalarda ko'rsatiladigan chiroyli nom.
- **`help_text="..."`** — foydalanuvchiga yordam matni.
- **`editable=False`** — admin va formalarda tahrirlab bo'lmaydi.

**Muhim farq: `null` vs `blank`**

Bu ikkisi ko'pincha adashtiriladi:
- `null=True` — bazaga bog'liq, ustun `NULL` bo'lishi mumkinligini bildiradi.
- `blank=True` — validatsiyaga bog'liq, forma/admin'da bo'sh qoldirish mumkinligini bildiradi.

**Qoida:** `CharField` va `TextField` uchun odatda faqat `blank=True` ishlatiladi, `null=True` ishlatilmaydi — chunki Django bo'sh matnni `NULL` emas, bo'sh satr (`""`) sifatida saqlaydi. Sonli va sana maydonlari uchun esa ikkalasini birga ishlatish mumkin:

```python
class Event(models.Model):
    title = models.CharField(max_length=100)          # majburiy
    description = models.TextField(blank=True)          # ixtiyoriy matn
    end_date = models.DateField(null=True, blank=True)  # ixtiyoriy sana
```

**`choices` misoli:**

```python
class Order(models.Model):
    STATUS_CHOICES = [
        ("pending", "Kutilmoqda"),
        ("shipped", "Jo'natildi"),
        ("delivered", "Yetkazildi"),
        ("cancelled", "Bekor qilindi"),
    ]
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default="pending")
```

---

## 6. Meta klassi

Har bir Model ichida `Meta` degan ichki klass e'lon qilish mumkin — bu Model'ning o'zi haqida emas, balki uning **metama'lumotlarini** (metadata) belgilaydi:

```python
class Book(models.Model):
    title = models.CharField(max_length=200)
    published_date = models.DateField()
    price = models.DecimalField(max_digits=8, decimal_places=2)

    class Meta:
        ordering = ["-published_date"]          # standart saralash tartibi
        verbose_name = "Kitob"
        verbose_name_plural = "Kitoblar"
        unique_together = [["title", "published_date"]]  # kombinatsiya unique bo'lishi kerak
        indexes = [models.Index(fields=["title"])]
        db_table = "books_book_custom"           # jadval nomini o'zgartirish
```

Eng ko'p ishlatiladigan `Meta` parametrlari:
- **`ordering`** — natijalarning standart tartibini belgilaydi (`-` belgisi kamayish tartibini bildiradi).
- **`verbose_name` / `verbose_name_plural`** — admin panelda ko'rsatiladigan nomlar.
- **`unique_together`** (yoki yangi Django'larda `constraints` bilan `UniqueConstraint`) — bir nechta ustunlar kombinatsiyasi unique bo'lishini ta'minlaydi.
- **`db_table`** — Django avtomatik yaratadigan jadval nomi (`app_modelname`) o'rniga o'zingiz belgilagan nom.
- **`abstract`** — Model'ni abstrakt qiladi (7-bo'limda ko'ramiz).

---

## 7. Model metodlari

Model — oddiy Python klassi bo'lgani uchun, unga o'zingizning metodlaringizni qo'shishingiz mumkin.

### `__str__` metodi

Bu metod Model obyektining matn ko'rinishini belgilaydi — admin panelda, shell'da va debugging paytida obyekt qanday ko'rinishini aniqlaydi:

```python
class Author(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name
```

`__str__` yozilmasa, Django obyektni `Author object (1)` kabi noqulay formatda ko'rsatadi.

### Custom metodlar

Biznes-logikani Model ichida saqlash — "fat models, thin views" tamoyiliga mos keladi:

```python
class Order(models.Model):
    quantity = models.PositiveIntegerField()
    unit_price = models.DecimalField(max_digits=8, decimal_places=2)

    def total_price(self):
        return self.quantity * self.unit_price

    @property
    def is_expensive(self):
        return self.total_price() > 1000
```

`@property` orqali metodni oddiy atribut kabi chaqirish mumkin: `order.is_expensive` (qavs `()` kerak emas).

### `save()` metodini override qilish

Ba'zida obyekt saqlanishidan oldin qo'shimcha ish bajarish kerak bo'ladi:

```python
from django.utils.text import slugify

class Article(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True, blank=True)

    def save(self, *args, **kwargs):
        if not self.slug:
            self.slug = slugify(self.title)
        super().save(*args, **kwargs)  # asosiy save() ni chaqirish MAJBURIY
```

**Muhim:** `super().save(*args, **kwargs)` chaqirilmasa, obyekt bazaga umuman saqlanmaydi.

---

## 8. Migratsiyalar (Migrations)

Model — bu Python kodi, lekin baza jadvali fizik ravishda mavjud bo'lishi kerak. Bu ikkisini sinxronlashtirish uchun **migratsiyalar** ishlatiladi.

```bash
# Model o'zgarishlariga asosan migratsiya fayllarini yaratish
python manage.py makemigrations

# Migratsiyalarni bazaga qo'llash (jadval yaratish/o'zgartirish)
python manage.py migrate

# Qanday SQL bajarilishini oldindan ko'rish
python manage.py sqlmigrate books 0001

# Migratsiyalar holatini ko'rish
python manage.py showmigrations
```

**Ish jarayoni:**
1. `models.py` faylida Model yaratasiz yoki o'zgartirasiz.
2. `makemigrations` — Django o'zgarishlarni aniqlab, `migrations/0001_initial.py` kabi fayl yaratadi (bu SQL emas, Python kodi — o'zgarishlar "retsepti").
3. `migrate` — bu retsept asosida haqiqiy SQL buyruqlarini bazada bajaradi.

**Nima uchun ikki bosqich?** Chunki `makemigrations` faqat fayl yaratadi, hali bazaga hech narsa qilmaydi — bu jamoaviy ishlashda muhim: har bir dasturchi migratsiya fayllarini Git orqali oladi va o'zining bazasida `migrate` qiladi.

---

## 9. Munosabatlar (Relationships)

### ForeignKey (Ko'p-ga-bir)

Bitta obyekt boshqa bir obyektga tegishli bo'lganda ishlatiladi (masalan, ko'p kitob bitta muallifga tegishli):

```python
class Book(models.Model):
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,
        related_name="books"
    )
```

`on_delete` — bog'langan obyekt o'chirilganda nima bo'lishini belgilaydi (**majburiy parametr**):
- `CASCADE` — muallif o'chirilsa, uning barcha kitoblari ham o'chadi.
- `PROTECT` — agar muallifga bog'langan kitoblar bo'lsa, uni o'chirishga yo'l qo'ymaydi.
- `SET_NULL` — muallif o'chirilsa, `author` maydoni `NULL` bo'ladi (`null=True` bilan birga ishlatiladi).
- `SET_DEFAULT` — standart qiymatga o'rnatadi.
- `DO_NOTHING` — hech narsa qilmaydi (xavfli, ehtiyot bo'lish kerak).

`related_name` — teskari bog'lanish uchun nom beradi: `author.books.all()` orqali muallifning barcha kitoblarini olish mumkin.

### ManyToManyField (Ko'p-ga-ko'p)

Ikki tomonlama ko'p bog'lanish uchun (masalan, bitta kitob bir nechta janrga tegishli, bitta janrda ko'p kitob bo'lishi mumkin):

```python
class Genre(models.Model):
    name = models.CharField(max_length=50)

class Book(models.Model):
    title = models.CharField(max_length=200)
    genres = models.ManyToManyField(Genre, related_name="books")
```

Django avtomatik ravishda oraliq (junction) jadval yaratadi. Foydalanish:

```python
book.genres.add(genre1, genre2)
book.genres.remove(genre1)
book.genres.all()
genre.books.all()  # teskari tomondan
```

### OneToOneField (Bir-ga-bir)

Har bir obyekt aniq bitta boshqa obyektga bog'langanda (masalan, foydalanuvchi va uning profili):

```python
from django.contrib.auth.models import User

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name="profile")
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to="avatars/", blank=True, null=True)
```

Bu, mohiyatan, `ForeignKey(unique=True)` bilan bir xil, lekin niyatni aniqroq ifodalaydi.

---

## 10. QuerySet va Manager — ORM asoslari

Har bir Model avtomatik ravishda `objects` degan **Manager** ga ega bo'ladi — bu bazadan ma'lumot olish uchun kirish nuqtasi:

```python
# Barcha obyektlarni olish
Book.objects.all()

# Filtrlash
Book.objects.filter(price__lt=50)          # narxi 50 dan kichik
Book.objects.filter(author__last_name="Tolstoy")  # bog'langan jadval orqali filtrlash
Book.objects.exclude(is_available=False)

# Bitta obyekt olish
Book.objects.get(id=1)          # topilmasa yoki bir nechta topilsa xatolik beradi
Book.objects.first()
Book.objects.last()

# Saralash
Book.objects.order_by("-published_date")

# Sanash va agregatsiya
Book.objects.count()
from django.db.models import Avg, Sum
Book.objects.aggregate(Avg("price"))

# Yangi obyekt yaratish
Book.objects.create(title="Yangi kitob", author=author, price=25000)

# Yangilash
Book.objects.filter(author=author).update(is_available=True)

# O'chirish
Book.objects.filter(id=1).delete()
```

**QuerySet'ning muhim xususiyati — "lazy evaluation":** QuerySet yozilganda bazaga so'rov yuborilmaydi, u faqat natija haqiqatan kerak bo'lganda (masalan, `for` sikli, `list()`, yoki `print()` chaqirilganda) bajariladi. Bu ko'p filtrlarni zanjir shaklida bog'lashga imkon beradi:

```python
qs = Book.objects.filter(price__lt=100).exclude(is_available=False).order_by("title")
# Bu yergacha bazaga hech qanday so'rov ketmagan
for book in qs:   # aynan shu yerda SQL so'rov bajariladi
    print(book.title)
```

### `select_related` va `prefetch_related` — N+1 muammosi

Bog'langan obyektlarga murojaat qilganda ko'p ortiqcha so'rov yubormaslik uchun:

```python
# ForeignKey / OneToOne uchun (SQL JOIN orqali)
books = Book.objects.select_related("author").all()

# ManyToMany yoki teskari ForeignKey uchun
books = Book.objects.prefetch_related("genres").all()
```

---

## 11. Validatsiya

Model darajasida validatsiya `full_clean()` orqali amalga oshiriladi (bu avtomatik `save()` chaqirilganda ishlamaydi — buni alohida chaqirish kerak, odatda formalar orqali avtomatik chaqiriladi):

```python
from django.core.exceptions import ValidationError

class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=8, decimal_places=2)

    def clean(self):
        if self.price <= 0:
            raise ValidationError("Narx musbat son bo'lishi kerak.")

# Foydalanish:
product = Product(name="Noutbuk", price=-100)
product.full_clean()  # ValidationError ko'taradi
```

**Validator'lar** — field darajasidagi qayta ishlatiladigan tekshiruvlar:

```python
from django.core.validators import MinValueValidator, MaxValueValidator

class Product(models.Model):
    rating = models.IntegerField(validators=[MinValueValidator(1), MaxValueValidator(5)])
```

---

## 12. Abstract Base Class va meros olish

Bir nechta Model'da takrorlanadigan field'larni umumiy klassga chiqarish mumkin:

```python
class TimeStampedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True   # bu klass uchun jadval yaratilmaydi

class Article(TimeStampedModel):
    title = models.CharField(max_length=200)
    content = models.TextField()

class Comment(TimeStampedModel):
    text = models.TextField()
```

`Article` va `Comment` ikkalasi ham `created_at` va `updated_at` field'lariga ega bo'ladi, lekin `TimeStampedModel` uchun alohida jadval yaratilmaydi — bu **kod takrorlanishining oldini oladi** (DRY tamoyili).

---

## 13. Eng yaxshi amaliyotlar (Best Practices)

1. **Har doim `__str__` metodini yozing** — bu debugging va admin panelni ancha qulaylashtiradi.
2. **Biznes-logikani Model ichida saqlang, View'da emas** ("fat models, thin views").
3. **`on_delete` parametrini ongli tanlang** — `CASCADE` har doim to'g'ri variant emas; muhim ma'lumotlar uchun `PROTECT` yoki `SET_NULL` xavfsizroq bo'lishi mumkin.
4. **Pul uchun `DecimalField` ishlating, `FloatField` emas** — `float` yaxlitlash xatolariga olib kelishi mumkin.
5. **Индекслардан фойдаланинг** (`db_index=True` yoki `Meta.indexes`) tez-tez qidiriladigan ustunlar uchun.
6. **N+1 muammosidan saqlaning** — `select_related` / `prefetch_related` dan foydalaning.
7. **Migratsiyalarni Git'ga qo'shing** va jamoada muvofiqlashtiring — hech qachon migratsiya fayllarini qo'lda tahrirlamang, agar aniq nima qilayotganingizni bilmasangiz.
8. **`related_name` bering** — ayniqsa bitta Model'ga bir nechta ForeignKey bog'langanda, aks holda Django xatolik beradi (`related_name` ziddiyati).

---

## 14. Xulosa

Django Model — MVT arxitekturasining asosi bo'lib, u:
- Ma'lumotlar tuzilishini (field'lar orqali) belgilaydi,
- Obyektlar orasidagi munosabatlarni (`ForeignKey`, `ManyToMany`, `OneToOne`) tavsiflaydi,
- Biznes-logika va validatsiyani o'z ichiga oladi,
- ORM orqali SQL yozmasdan baza bilan ishlash imkonini beradi,
- Migratsiyalar orqali kod va baza tuzilishini sinxronlab turadi.

Model'ni yaxshi loyihalash — butun ilovaning barqarorligi va kengaytiriluvchanligi uchun hal qiluvchi ahamiyatga ega. Keyingi qadam sifatida **Django Forms** yoki **Django Admin** mavzularini o'rganish, yoki QuerySet'lar va agregatsiyalarni chuqurroq mashq qilish tavsiya etiladi.
