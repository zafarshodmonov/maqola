# Amaliy Django Model: Algoritmik Platforma (AtCoder / LeetCode / Codeforces uslubida)

Bu darslikda oldingi Model darslikidagi nazariyani **amaliy misol** orqali mustahkamlaymiz. Loyiha sifatida AtCoder, LeetCode, Codeforces kabi platformalarning soddalashtirilgan bazasini quramiz: **Contest**, **Problem**, **Tag**, **Submission**, **Profile**. Bu misol barcha uchta asosiy munosabat turini (`ForeignKey`, `ManyToMany`, `OneToOne`) o'zida real vaziyatda namoyish etadi.

## Mundarija
1. Loyiha tuzilishini rejalashtirish (ER diagram mantig'i)
2. `Tag` modeli — eng oddiy Model
3. `Contest` modeli
4. `Problem` modeli — ForeignKey va ManyToMany birga
5. `Submission` modeli — bir nechta ForeignKey
6. `Profile` modeli — OneToOne
7. To'liq `models.py`
8. Munosabatlarni amalda sinash (Django shell)
9. Real so'rovlar: reyting, statistika, filtrlash
10. Xulosa

---

## 1. Loyiha tuzilishini rejalashtirish

Avval qog'ozda (yoki miyada) qanday obyektlar borligini va ular orasidagi munosabatni belgilab olamiz — bu haqiqiy loyihalashda ham birinchi qadam:

```
Contest (Musobaqa)
  └── bitta Contest'da ko'p Problem bo'ladi         → ForeignKey (Problem → Contest)

Problem (Masala)
  └── bitta Problem bir nechta Tag'ga ega bo'lishi mumkin
      va bitta Tag ko'p Problem'da ishlatiladi        → ManyToMany (Problem ↔ Tag)

Tag (masalan: "dp", "graph", "greedy", "math")
  └── mustaqil, oddiy spravochnik modeli

User (Django'ning tayyor foydalanuvchi modeli)
  └── har bir User aynan bitta Profile'ga ega          → OneToOne (Profile → User)

Submission (Yechim yuborish)
  └── bitta Submission — bitta User va bitta Problem'ga tegishli → ikkita ForeignKey
```

Bu — real loyihalarda ishlatiladigan tipik "ER diagram" mantig'i: avval obyektlarni (entity) aniqlaysiz, keyin ular orasidagi munosabat turini (1-ga-ko'p, ko'p-ga-ko'p, 1-ga-1) belgilaysiz, va shundan keyingina kodga o'tasiz.

---

## 2. `Tag` modeli — eng oddiy Model

`Tag` — hech kimga bog'liq bo'lmagan, mustaqil "spravochnik" (lookup table):

```python
# problems/models.py
from django.db import models

class Tag(models.Model):
    name = models.CharField(max_length=30, unique=True)  # masalan: "dp", "graph"
    slug = models.SlugField(max_length=30, unique=True)   # URL uchun: "dynamic-programming"

    class Meta:
        ordering = ["name"]

    def __str__(self):
        return self.name
```

`unique=True` — bir xil nomli tag ikki marta yaratilmasligi uchun. Bu — Codeforces'dagi "dp", "greedy", "graphs" kabi teglarga to'g'ridan-to'g'ri mos keladi.

---

## 3. `Contest` modeli

```python
class Contest(models.Model):
    PLATFORM_CHOICES = [
        ("atcoder", "AtCoder"),
        ("codeforces", "Codeforces"),
        ("leetcode", "LeetCode"),
    ]

    title = models.CharField(max_length=150)          # masalan: "AtCoder Beginner Contest 350"
    platform = models.CharField(max_length=20, choices=PLATFORM_CHOICES)
    start_time = models.DateTimeField()
    duration_minutes = models.PositiveIntegerField(default=120)

    class Meta:
        ordering = ["-start_time"]  # eng yangi musobaqa birinchi

    def __str__(self):
        return f"{self.title} ({self.get_platform_display()})"
```

**Diqqat:** `get_platform_display()` — `choices` ishlatilgan har qanday field uchun Django avtomatik yaratadigan metod. U bazadagi qiymatni (`"atcoder"`) emas, foydalanuvchiga ko'rsatiladigan chiroyli nomni (`"AtCoder"`) qaytaradi.

---

## 4. `Problem` modeli — ForeignKey va ManyToMany birga

Bu modelda ikkita munosabat turi bir vaqtda ishlatiladi:

```python
class Problem(models.Model):
    DIFFICULTY_CHOICES = [
        ("easy", "Oson"),
        ("medium", "O'rta"),
        ("hard", "Qiyin"),
    ]

    contest = models.ForeignKey(
        Contest,
        on_delete=models.CASCADE,
        related_name="problems"
    )
    index_letter = models.CharField(max_length=2)   # masalan: "A", "B", "C", "D"
    title = models.CharField(max_length=200)
    difficulty = models.CharField(max_length=10, choices=DIFFICULTY_CHOICES)
    rating = models.PositiveIntegerField(null=True, blank=True)  # Codeforces uslubidagi rating (masalan 1600)
    time_limit_ms = models.PositiveIntegerField(default=2000)
    tags = models.ManyToManyField(Tag, related_name="problems", blank=True)

    class Meta:
        ordering = ["contest", "index_letter"]
        unique_together = [["contest", "index_letter"]]  # bitta contestda "B" masala bitta bo'ladi

    def __str__(self):
        return f"{self.contest.title} - {self.index_letter}. {self.title}"
```

**Nega bu yerda ForeignKey, nega ManyToMany?**

- `contest = ForeignKey(Contest, ...)` — chunki bitta Problem faqat **bitta** Contest'ga tegishli, lekin bitta Contest'da **ko'p** Problem bo'ladi (1-ga-ko'p). Bu xuddi AtCoder'dagi "ABC350 A", "ABC350 B" kabi tuzilishga mos keladi.
- `tags = ManyToManyField(Tag, ...)` — chunki bitta Problem bir nechta tag'ga ega bo'lishi mumkin ("dp" + "graph" birga kelishi mumkin), va bitta tag ("dp") minglab masalalarda ishlatiladi. Bu — ikki tomonlama ko'p bog'lanish, ya'ni **ko'p-ga-ko'p**.

`unique_together` esa real hayotdagi cheklovni bazaga majburlaydi: bitta musobaqada ikkita "B" masala bo'lishi mumkin emas.

---

## 5. `Submission` modeli — bir nechta ForeignKey

Foydalanuvchi masalaga yechim yuborganda yoziladigan yozuv:

```python
from django.contrib.auth.models import User

class Submission(models.Model):
    STATUS_CHOICES = [
        ("AC", "Accepted"),
        ("WA", "Wrong Answer"),
        ("TLE", "Time Limit Exceeded"),
        ("RE", "Runtime Error"),
        ("CE", "Compile Error"),
    ]

    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name="submissions")
    problem = models.ForeignKey(Problem, on_delete=models.CASCADE, related_name="submissions")
    language = models.CharField(max_length=30)          # "C++17", "Python3", "Java" va h.k.
    status = models.CharField(max_length=5, choices=STATUS_CHOICES)
    execution_time_ms = models.PositiveIntegerField(null=True, blank=True)
    submitted_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ["-submitted_at"]

    def __str__(self):
        return f"{self.user.username} → {self.problem} [{self.status}]"
```

Bu yerda **ikkita** `ForeignKey` bor — `user` va `problem`. Bu, aslida, `User` bilan `Problem` orasidagi ko'p-ga-ko'p munosabatni "oraliq model" (through model) orqali ifodalash usuli ham hisoblanadi: bitta User ko'p marta submit qiladi, bitta Problem'ga ko'p User yechim yuboradi, lekin har bir Submission qo'shimcha ma'lumot (til, status, vaqt) bilan boyitilgan — shu sababli oddiy `ManyToManyField` emas, alohida Model sifatida ifodalangan.

---

## 6. `Profile` modeli — OneToOne

Har bir foydalanuvchi **aynan bitta** qo'shimcha profilga ega bo'lishi kerak (Django'ning ichki `User` modelini o'zgartira olmaymiz, shuning uchun uni "kengaytiramiz"):

```python
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name="profile")
    rating = models.IntegerField(default=1500)
    country = models.CharField(max_length=50, blank=True)
    avatar = models.ImageField(upload_to="avatars/", blank=True, null=True)

    def __str__(self):
        return f"{self.user.username} (rating: {self.rating})"
```

`OneToOneField` — mohiyatan `ForeignKey(unique=True)` bilan bir xil, lekin bu yerda niyat aniq: har bir User uchun aynan bitta Profile, ortiqcha emas.

---

## 7. To'liq `models.py`

Barchasini birlashtirsak:

```python
from django.db import models
from django.contrib.auth.models import User


class Tag(models.Model):
    name = models.CharField(max_length=30, unique=True)
    slug = models.SlugField(max_length=30, unique=True)

    class Meta:
        ordering = ["name"]

    def __str__(self):
        return self.name


class Contest(models.Model):
    PLATFORM_CHOICES = [
        ("atcoder", "AtCoder"),
        ("codeforces", "Codeforces"),
        ("leetcode", "LeetCode"),
    ]
    title = models.CharField(max_length=150)
    platform = models.CharField(max_length=20, choices=PLATFORM_CHOICES)
    start_time = models.DateTimeField()
    duration_minutes = models.PositiveIntegerField(default=120)

    class Meta:
        ordering = ["-start_time"]

    def __str__(self):
        return f"{self.title} ({self.get_platform_display()})"


class Problem(models.Model):
    DIFFICULTY_CHOICES = [
        ("easy", "Oson"),
        ("medium", "O'rta"),
        ("hard", "Qiyin"),
    ]
    contest = models.ForeignKey(Contest, on_delete=models.CASCADE, related_name="problems")
    index_letter = models.CharField(max_length=2)
    title = models.CharField(max_length=200)
    difficulty = models.CharField(max_length=10, choices=DIFFICULTY_CHOICES)
    rating = models.PositiveIntegerField(null=True, blank=True)
    time_limit_ms = models.PositiveIntegerField(default=2000)
    tags = models.ManyToManyField(Tag, related_name="problems", blank=True)

    class Meta:
        ordering = ["contest", "index_letter"]
        unique_together = [["contest", "index_letter"]]

    def __str__(self):
        return f"{self.contest.title} - {self.index_letter}. {self.title}"


class Submission(models.Model):
    STATUS_CHOICES = [
        ("AC", "Accepted"),
        ("WA", "Wrong Answer"),
        ("TLE", "Time Limit Exceeded"),
        ("RE", "Runtime Error"),
        ("CE", "Compile Error"),
    ]
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name="submissions")
    problem = models.ForeignKey(Problem, on_delete=models.CASCADE, related_name="submissions")
    language = models.CharField(max_length=30)
    status = models.CharField(max_length=5, choices=STATUS_CHOICES)
    execution_time_ms = models.PositiveIntegerField(null=True, blank=True)
    submitted_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ["-submitted_at"]

    def __str__(self):
        return f"{self.user.username} → {self.problem} [{self.status}]"


class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name="profile")
    rating = models.IntegerField(default=1500)
    country = models.CharField(max_length=50, blank=True)
    avatar = models.ImageField(upload_to="avatars/", blank=True, null=True)

    def __str__(self):
        return f"{self.user.username} (rating: {self.rating})"
```

Migratsiya qilish:

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## 8. Munosabatlarni amalda sinash (Django shell)

```bash
python manage.py shell
```

```python
from problems.models import Contest, Problem, Tag, Submission, Profile
from django.contrib.auth.models import User
from django.utils import timezone

# Contest yaratish
contest = Contest.objects.create(
    title="AtCoder Beginner Contest 350",
    platform="atcoder",
    start_time=timezone.now()
)

# Tag'lar yaratish
dp_tag = Tag.objects.create(name="dp", slug="dp")
graph_tag = Tag.objects.create(name="graph", slug="graph")

# Problem yaratish (avval ForeignKey orqali contest bilan bog'laymiz)
problem = Problem.objects.create(
    contest=contest,
    index_letter="D",
    title="Shortest Path with Twist",
    difficulty="medium",
    rating=1200
)

# ManyToMany bog'lanish — obyekt saqlangandan KEYIN qo'shiladi
problem.tags.add(dp_tag, graph_tag)

# Teskari tomondan ko'rish (related_name orqali)
contest.problems.all()          # bu contest'dagi barcha masalalar
dp_tag.problems.all()           # "dp" tegiga ega barcha masalalar
problem.tags.all()              # bu masalaning barcha teglari

# User va Profile (OneToOne)
user = User.objects.create_user(username="zafar", password="12345")
profile = Profile.objects.create(user=user, rating=1650, country="Uzbekistan")

user.profile                    # OneToOne orqali to'g'ridan-to'g'ri kirish
profile.user.username           # teskari yo'nalish

# Submission yaratish
Submission.objects.create(
    user=user,
    problem=problem,
    language="C++20",
    status="AC",
    execution_time_ms=340
)

user.submissions.all()          # bu foydalanuvchining barcha yuborishlari
problem.submissions.all()       # bu masalaga yuborilgan barcha yechimlar
```

**Muhim qoida:** `ManyToManyField`ni ishlatishdan oldin ikkala obyekt ham bazada saqlangan (ya'ni `id` ga ega) bo'lishi shart — shuning uchun `problem.tags.add(...)` faqat `problem.save()` (yoki `objects.create()`) dan **keyin** chaqiriladi.

---

## 9. Real so'rovlar: reyting, statistika, filtrlash

Endi bu tuzilma ustida real platformalarga xos so'rovlarni yozamiz:

```python
# 1. Ma'lum tag bo'yicha barcha masalalarni topish (masalan, "dp" mavzusidagi masalalar)
Problem.objects.filter(tags__name="dp")

# 2. Rating oralig'ida masalalarni filtrlash (Codeforces uslubida)
Problem.objects.filter(rating__gte=1200, rating__lte=1600)

# 3. Foydalanuvchi qancha masalani AC qilgani (unique)
user.submissions.filter(status="AC").values("problem").distinct().count()

# 4. Eng ko'p ishlatiladigan tag'larni aniqlash
from django.db.models import Count
Tag.objects.annotate(problem_count=Count("problems")).order_by("-problem_count")

# 5. Muayyan contest ichida qiyinlik darajasi bo'yicha saralangan masalalar
contest.problems.order_by("rating")

# 6. Foydalanuvchining reytingi bo'yicha top-10 ro'yxati
Profile.objects.order_by("-rating")[:10]

# 7. select_related va prefetch_related bilan optimallashtirish
# (N+1 muammosining oldini olish uchun — submission ro'yxatini chiqarganda)
Submission.objects.select_related("user", "problem").prefetch_related("problem__tags").all()
```

5-misolda `contest.problems` — bu `related_name="problems"` orqali ishlagan **teskari ForeignKey** so'rovi; alohida `Problem.objects.filter(contest=contest)` yozishga hojat qolmaydi.

4-misolda `annotate(Count(...))` — bu ManyToMany munosabat orqali agregatsiya qilishning klassik namunasi: "qaysi tag qanchta masalada ishlatilgan" degan savolga bitta so'rov bilan javob beradi.

---

## 10. Xulosa

Bu amaliy misolda uchta asosiy munosabat turi bir tizim ichida qo'llanildi:

| Munosabat | Qayerda ishlatildi | Nega aynan shu tur |
|---|---|---|
| **ForeignKey** (1-ga-ko'p) | `Problem → Contest`, `Submission → User`, `Submission → Problem` | Bitta tomon ko'p, ikkinchi tomon bitta |
| **ManyToManyField** (ko'p-ga-ko'p) | `Problem ↔ Tag` | Ikki tomon ham bir-biriga ko'p marta bog'lanadi |
| **OneToOneField** (1-ga-1) | `Profile → User` | Har bir tomon aynan bittadan bog'lanadi |

Real algoritmik platformalarni loyihalashda aynan shu uchta munosabat turi va `related_name`, `unique_together`, `choices` kabi vositalar orqali butun tizimning mantiqiy tuzilishi ORM darajasida ta'minlanadi — bu esa View va Template qatlamlarida SQL haqida deyarli o'ylamasdan ishlash imkonini beradi.
