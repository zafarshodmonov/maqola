# C Dasturlash Tilida Type Casting (Tur Aylantirish)

## Mundarija
1. Kirish — Casting nima va nima uchun kerak
2. Implicit Conversion (Yashirin/Avtomatik aylantirish)
3. Integer Promotion (Butun son ko'tarilishi)
4. Usual Arithmetic Conversions (Odatiy arifmetik aylantirish)
5. Explicit Casting (Aniq/Qo'lda aylantirish)
6. Sonli turlar orasidagi cast (int, float, double)
7. Kichik va katta turlar orasidagi cast (truncation, overflow)
8. Signed va Unsigned orasidagi cast
9. Pointer casting
10. `void*` va uning ahamiyati
11. Function pointer casting
12. Keng tarqalgan xatolar va ulardan qochish
13. Best practices (Yaxshi amaliyotlar)
14. Xulosa va mashqlar

---

## 1. Kirish — Casting nima va nima uchun kerak

**Type casting (tur aylantirish)** — bu bir ma'lumot turidagi qiymatni boshqa ma'lumot turiga o'girish jarayoni. C tili **statik tiplashtirilgan (statically typed)** til bo'lgani uchun, har bir o'zgaruvchi qat'iy belgilangan turga ega, va turlar orasida operatsiya bajarilganda til qanday harakat qilishni aniq bilishi kerak.

Masalan:
```c
int a = 5;
double b = 2.0;
double c = a + b;  // 'a' avtomatik double'ga aylantiriladi
```

Casting ikki xil bo'ladi:

| Turi | Tavsif | Kim bajaradi |
|---|---|---|
| **Implicit (yashirin)** | Kompilyator avtomatik bajaradi | Kompilyator |
| **Explicit (aniq)** | Dasturchi maxsus sintaksis bilan buyuradi | Dasturchi |

Nima uchun bu muhim? Chunki noto'g'ri yoki e'tiborsiz casting **ma'lumot yo'qotilishiga (data loss)**, **kutilmagan natijalarga**, hatto **xavfsiz bo'lmagan xotira xatolariga** olib kelishi mumkin (masalan, pointer casting noto'g'ri qo'llanilsa).

---

## 2. Implicit Conversion (Yashirin/Avtomatik aylantirish)

Kompilyator ba'zi holatlarda o'zi turlarni avtomatik moslashtiradi. Bu quyidagi hollarda sodir bo'ladi:

### 2.1. Assignment (tenglashtirish) paytida
```c
int x = 10;
double y = x;      // int -> double, avtomatik, xavfsiz (kengaytirish)
int z = 3.99;       // double -> int, avtomatik, XAVFLI (qisqartirish, kasr qismi yo'qoladi)
printf("%d\n", z);  // 3 chiqadi (kesib tashlanadi, yaxlitlanmaydi!)
```

**Muhim:** `double` dan `int`ga o'tishda C **yaxlitlamaydi (round)**, balki **kesib tashlaydi (truncate)** — ya'ni kasr qismi butunlay tashlab yuboriladi, manfiy sonlarda ham nolga qarab kesiladi:
```c
int a = 3.99;   // 3
int b = -3.99;  // -3 (pastga emas, nolga qarab kesiladi)
```

### 2.2. Function chaqiruvida (argument passing)
```c
void foo(double d) {
    printf("%f\n", d);
}

int main() {
    foo(5);  // int -> double avtomatik aylantiriladi
}
```

### 2.3. Arifmetik ifodalarda
Bu eng ko'p uchraydigan holat bo'lib, quyidagi bo'limda batafsil ko'rib chiqamiz.

---

## 3. Integer Promotion (Butun son ko'tarilishi)

C standartiga ko'ra, arifmetik yoki mantiqiy operatsiyada qatnashuvchi **`int`dan kichik** turlar (`char`, `short`, va bit-fieldlar) avtomatik ravishda **`int`ga ko'tariladi** (agar `int` ularning barcha qiymatlarini sig'dira olsa) yoki `unsigned int`ga.

```c
char a = 100;
char b = 100;
char c = a + b;  // XATO EMAS, lekin diqqat qiling!
```

Bu yerda `a + b` bajarilishidan oldin ikkalasi ham `int`ga ko'tariladi: `100 + 100 = 200` (int sifatida hisoblanadi, overflow bo'lmaydi chunki int kattaroq). Keyin natija `char`ga qaytarib cast qilinadi (`c` ga tenglashtirilganda). Agar `char` **signed** bo'lsa va odatda 8 bit (-128...127 oralig'ida) bo'lsa, `200` bu oraliqqa sig'maydi va **implementation-defined** (kompilyatorga bog'liq) natija olamiz — ko'pincha `-56` chiqadi (`200 - 256 = -56`).

**Nega bu muhim?** Chunki kichik turlar bilan ishlaganda ham, oraliq hisoblashlar aslida `int` darajasida bo'ladi — bu performance uchun foydali (protsessorlar `int` bilan tezroq ishlaydi), lekin natijani qayta kichik turga sig'dirishda ehtiyot bo'lish kerak.

---

## 4. Usual Arithmetic Conversions (Odatiy arifmetik aylantirish)

Ikki xil turdagi operandlar arifmetik operatorda (`+`, `-`, `*`, `/`, `%`) ishtirok etganda, C **"kichikroq" turni "kattaroq" turga** aylantiradi (bu "type hierarchy" yoki "conversion rank" deb ataladi). Umumiy tartib (kattadan kichikka):

```
long double > double > float > unsigned long long > long long 
> unsigned long > long > unsigned int > int
```

**Qoida oddiy:** Ikki operanddan qaysi biri "kattaroq" bo'lsa, ikkinchisi o'sha turga aylantiriladi, va natija ham o'sha turda bo'ladi.

### Misollar:

```c
int a = 5;
float b = 2.0f;
auto natija = a / b;   // 'a' -> float ga aylantiriladi, natija float: 2.5

int x = 7;
double y = 2.0;
// x avtomatik double'ga aylantiriladi -> 7.0 / 2.0 = 3.5
```

### Signed va unsigned aralashganda (juda muhim!)

Agar bir xil rangdagi (masalan ikkalasi ham `int` darajasida) `signed` va `unsigned` turlar aralashsa, **`signed` `unsigned`ga aylantiriladi** — bu ko'plab xatolarning manbai:

```c
int a = -1;
unsigned int b = 1;

if (a < b) {
    printf("a kichik\n");
} else {
    printf("a katta yoki teng\n");  // AYNAN SHU CHIQADI!
}
```

**Nima uchun?** `a = -1` qiymati `unsigned int`ga aylantirilganda, u manfiy son sifatida emas, balki **bit pattern** sifatida talqin qilinadi — natijada `-1` → `UINT_MAX` (masalan, 32-bitli sistemada `4294967295`) ga aylanadi. Shuning uchun `4294967295 < 1` — noto'g'ri, va `else` blok ishga tushadi.

Bu — C dasturlashda eng ko'p uchraydigan "silent bug"lardan biri, ayniqsa `size_t` (bu odatda `unsigned`) bilan ishlaganda:

```c
for (size_t i = 10; i >= 0; i--) {  // CHEKSIZ SIKL!
    printf("%zu\n", i);
}
```

`size_t` unsigned bo'lgani uchun `i` hech qachon manfiy bo'la olmaydi; `0`dan keyin `i--` uni yana `UINT_MAX`ga aylantiradi, va sikl to'xtamaydi.

---

## 5. Explicit Casting (Aniq/Qo'lda aylantirish)

Dasturchi turni o'zi ko'rsatib aylantirishni xohlasa, **cast operator** ishlatiladi:

```c
(tur) ifoda
```

Misollar:
```c
double pi = 3.14159;
int butun_qism = (int)pi;       // 3

int a = 7, b = 2;
double natija = (double)a / b;  // 3.5 — to'g'ri usul

float f = (float)3.14159265358979;  // double -> float, aniqlik yo'qoladi
```

Cast operatorining **ustuvorligi (precedence)** juda yuqori — unar operatorlar darajasida, shuning uchun u faqat o'zidan keyin kelgan **bitta operand**ga ta'sir qiladi:

```c
(double)a / b     // (double)a keyin / b  -> to'g'ri
(double)(a / b)   // avval a/b (int bo'linish), keyin double'ga cast -> noto'g'ri natija bersa ham mumkin
```

Bu ikkalasi orasidagi farq — bu oldingi suhbatimizda ko'rgan `a/b == 9` muammosining aynan yechimi edi.

### C99+ da murakkabroq cast: Compound Literal

```c
int *p = (int[]){1, 2, 3};  // Vaqtinchalik massivga pointer
```

Bu kamdan-kam ishlatiladi, lekin bilish foydali.

---

## 6. Sonli turlar orasidagi cast (int, float, double)

### int -> float/double
Odatda xavfsiz, chunki `float`/`double` ancha katta oraliqni qamrab oladi. Lekin **juda katta int qiymatlar** `float`da **aniqlikni yo'qotishi** mumkin, chunki `float` atigi ~7 xonali o'nlik aniqlikka ega:

```c
int katta = 123456789;
float f = katta;
printf("%.0f\n", f);  // 123456792 chiqishi mumkin — aniqlik yo'qoldi!
```

`double` esa ~15-16 xonali aniqlikka ega, shuning uchun bunday holatlar kamroq uchraydi, lekin printsip bir xil.

### float/double -> int
Har doim **kesib tashlash (truncation)** sodir bo'ladi, yaxlitlash emas:
```c
int a = (int)9.99;   // 9
int b = (int)-9.99;  // -9
```

Yaxlitlash kerak bo'lsa, `<math.h>` dagi `round()`, `floor()`, `ceil()` funksiyalaridan foydalaning:
```c
#include <math.h>
int a = (int)round(9.99);  // 10
```

**Xavfli holat — Overflow:** Agar `double` qiymati `int` sig'dira olmaydigan darajada katta bo'lsa, natija **aniqlanmagan xatti-harakat (undefined behavior)** hisoblanadi:
```c
int x = (int)1e20;  // Undefined behavior! Natija noaniq.
```

### double <-> float
`double` dan `float`ga o'tishda aniqlik yo'qolishi mumkin (double ~15-16 xonali, float ~7 xonali aniqlikka ega):
```c
double d = 3.141592653589793;
float f = (float)d;  // 3.1415927 (aniqlik yo'qoldi)
```

---

## 7. Kichik va katta turlar orasidagi cast (Truncation va Overflow)

### Katta turdan kichik turga (Narrowing / Toraytirish)

```c
int a = 300;
char c = (char)a;  // char odatda -128..127 yoki 0..255 oralig'ida
printf("%d\n", c);  // implementation-defined, ko'pincha 44 chiqadi (300 % 256 = 44)
```

Bu yerda nima bo'ladi: yuqori bitlar shunchaki **"kesib tashlanadi"** (truncate qilinadi), qolgan quyi bitlar saqlanib qoladi. Matematik jihatdan, `unsigned` turlar uchun bu **modulo (2^n)** operatsiyasiga teng, `n` — maqsad turining bit soni.

```c
unsigned char uc = (unsigned char)300;  // 300 % 256 = 44
```

`signed` turlar uchun esa bu **implementation-defined** (standart bo'yicha), garchi amalda deyarli barcha zamonaviy kompilyatorlar xuddi shu modulo/bit-pattern usulini qo'llaydi.

### Kichik turdan katta turga (Widening / Kengaytirish)

Bu har doim xavfsiz va aniq qiymatni saqlaydi:
```c
char c = 65;
int i = c;  // 65, xavfsiz
```

Lekin **signed** kichik tur kengaytirilganda, **sign extension (belgi kengaytirilishi)** sodir bo'ladi:
```c
signed char sc = -1;   // bit pattern: 11111111
int i = sc;             // -1 (barcha yuqori bitlar 1 bilan to'ldiriladi)
printf("%d\n", i);      // -1

unsigned char uc = 255;  // bit pattern: 11111111 (lekin unsigned!)
int j = uc;
printf("%d\n", j);       // 255 (yuqori bitlar 0 bilan to'ldiriladi, chunki unsigned)
```

Bu farq — bit patterni bir xil (`11111111`) bo'lsa ham, **turning signed yoki unsigned ekanligiga qarab natija butunlay boshqacha** bo'lishi mumkinligini ko'rsatadi.

---

## 8. Signed va Unsigned orasidagi cast

Bu — C dagi eng ko'p chalkashlik keltiradigan mavzulardan biri. Signed va unsigned orasidagi cast **qiymatni o'zgartirmaydi, faqat bitlarni qayta talqin qiladi (reinterpret)**.

```c
int a = -1;
unsigned int b = (unsigned int)a;
printf("%u\n", b);  // 4294967295 (32-bitli sistemada), ya'ni UINT_MAX
```

Buning sababi: `-1` ikkilik sanoq sistemasida **ikkilik to'ldiruvchi (two's complement)** ko'rinishida barcha bitlari `1` bo'lgan songa teng (`11111111...1`). Signed sifatida bu `-1`, lekin xuddi shu bit patterni unsigned sifatida o'qilsa — bu turning maksimal qiymati bo'ladi.

**Teskari yo'nalish:**
```c
unsigned int a = 4294967295;  // UINT_MAX
int b = (int)a;
printf("%d\n", b);  // -1
```

### Amaliy maslahat
- Hisoblashlarda manfiy son bo'lishi mumkin bo'lsa, `unsigned` ishlatmang.
- `size_t`, `strlen()` kabi unsigned qaytaruvchi funksiyalar bilan ishlaganda, ularni signed son bilan solishtirishdan saqlaning.
- Agar solishtirish kerak bo'lsa, ikkalasini ham bir xil (afzalan signed, agar oraliq ruxsat bersa) turga cast qiling.

---

## 9. Pointer Casting

Pointerlar orasidagi cast — C tilining eng kuchli, lekin ayni paytda eng xavfli qismlaridan biri, chunki bu bevosita **xotirani qanday talqin qilishni** belgilaydi.

### 9.1. Bir xil turdagi pointerlar orasida (odatiy holat)
```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr;  // cast shart emas, avtomatik moslashadi (massiv -> pointer)
```

### 9.2. Turli turdagi pointerlar orasida (explicit cast talab qilinadi)
```c
int x = 65;
int *pi = &x;
char *pc = (char*)pi;  // int* -> char*

printf("%d\n", *pc);  // faqat 'x'ning birinchi bayti (kichik-endian sistemada 65, ya'ni 'A')
```

Bu yerda **kichik-endian (little-endian)** sistemalarda `x = 65` xotirada `[65, 0, 0, 0]` tarzida saqlanadi (4 baytli int uchun), shuning uchun `*pc` faqat birinchi baytni, ya'ni `65`ni o'qiydi.

### 9.3. Strict Aliasing Rule (Qat'iy taxallus qoidasi)

C standarti bo'yicha, bir turdagi pointer orqali boshqa (mos kelmaydigan) turdagi obyektga murojaat qilish **undefined behavior** hisoblanadi (`char*` bundan mustasno — u har qanday turga murojaat qilishi mumkin):

```c
float f = 3.14f;
int *ip = (int*)&f;
int i = *ip;  // Texnik jihatdan UB, garchi ko'p kompilyatorlar buni "ishlatishi" mumkin
```

Bunday "type punning" (bir xil xotirani boshqa tur sifatida o'qish) kerak bo'lsa, **`memcpy()`** yoki **`union`** ishlatish tavsiya etiladi:

```c
float f = 3.14f;
int i;
memcpy(&i, &f, sizeof(float));  // xavfsiz usul
```

### 9.4. Pointer va Integer orasida cast

```c
int x = 10;
int *p = &x;
uintptr_t addr = (uintptr_t)p;  // pointer manzilini butun songa aylantirish
printf("Manzil: %p\n", (void*)addr);
```

`uintptr_t` (`<stdint.h>`da e'lon qilingan) — pointerni xavfsiz saqlay oladigan butun son turi. Oddiy `int` yoki `long`ga cast qilish platformaga bog'liq (masalan, 64-bitli sistemada pointer 64 bit, `int` esa 32 bit bo'lishi mumkin va ma'lumot yo'qoladi).

---

## 10. `void*` va uning ahamiyati

`void*` — **"turi noma'lum" yoki "umumiy" pointer**. U har qanday obyekt turiga ishora qilishi mumkin, va C tilida (C++dan farqli o'laroq) `void*` dan boshqa pointer turiga **avtomatik** (implicit) cast qilinadi:

```c
void *vp = malloc(sizeof(int) * 5);  // malloc void* qaytaradi
int *ip = vp;  // C da cast shart emas (lekin C++ da shart!)
```

C++ da esa buni albatta explicit cast qilish kerak:
```cpp
int *ip = (int*)vp;  // C++ da majburiy
```

`void*` odatda quyidagi holatlarda ishlatiladi:
- `malloc`, `calloc`, `realloc` funksiyalari
- Generic (umumiy) funksiyalar, masalan `qsort()`, `memcpy()`
- Ma'lumot turidan qat'iy nazar ishlaydigan strukturalar (masalan, generic linked list)

```c
void generic_print(void *data, char turi) {
    if (turi == 'i')
        printf("%d\n", *(int*)data);
    else if (turi == 'f')
        printf("%f\n", *(float*)data);
}
```

**Muhim:** `void*` bilan arifmetika (`vp + 1`) standart C da ruxsat etilmagan (GCC kengaytmasi sifatida ishlaydi, lekin portable emas).

---

## 11. Function Pointer Casting

Funksiya pointerlarini cast qilish alohida diqqatga loyiq, chunki ular **ma'lumot pointerlaridan** farqli xotira modeliga ega bo'lishi mumkin (ba'zi platformalarda).

```c
int qoshish(int a, int b) { return a + b; }

typedef int (*OperatsiyaPtr)(int, int);

OperatsiyaPtr op = qoshish;  // to'g'ri, cast shart emas

// Noto'g'ri turdagi cast - UB:
void (*notogri)() = (void(*)())qoshish;
```

Function pointer bilan `void*` orasida cast qilish **standart C da UB** hisoblanadi (POSIX kabi ba'zi platformalar buni ruxsat beradi, masalan `dlsym()` bilan ishlashda), shuning uchun bunga alohida ehtiyot bo'lish kerak.

---

## 12. Keng Tarqalgan Xatolar va Ulardan Qochish

### Xato 1: Butun sonli bo'linishni unutish
```c
int a = 5, b = 2;
double natija = a / b;  // XATO: natija 2.0 (5/2=2, keyin double'ga cast)
double togri = (double)a / b;  // TO'G'RI: 2.5
```

### Xato 2: Signed/Unsigned solishtirish
```c
int a = -1;
unsigned int b = 5;
if (a < b) { ... }  // XATO: a UINT_MAX'ga aylanadi, shart false bo'ladi
```

### Xato 3: Overflow'ni hisobga olmaslik
```c
char c = 200;  // signed char uchun overflow (agar diapazon -128..127 bo'lsa)
```

### Xato 4: float/double solishtirishda aniqlik xatoligi
```c
double x = 0.1 + 0.2;
if (x == 0.3) { ... }  // odatda FALSE! Floating-point aniqlik xatoligi

// TO'G'RI usul:
if (fabs(x - 0.3) < 1e-9) { ... }
```

### Xato 5: Pointer turini noto'g'ri cast qilish (strict aliasing buzilishi)
```c
double d = 3.14;
long *lp = (long*)&d;  // UB, memcpy ishlatish tavsiya etiladi
```

### Xato 6: Yaxlitlash o'rniga kesish deb o'ylash
```c
int yosh = (int)(25.9);  // 25 chiqadi, 26 emas!
```

---

## 13. Best Practices (Yaxshi amaliyotlar)

1. **Har doim explicit cast ishlating**, agar niyatingiz aniq bo'lsin desangiz — bu kodni o'qish osonlashtiradi va kelajakdagi xatolarni oldini oladi.
2. **Bo'lishdan oldin castni to'g'ri joyga qo'ying** — operandga, natijaga emas.
3. **Signed va unsigned turlarni aralashtirmang**, ayniqsa solishtirishlarda.
4. **`-Wall -Wextra -Wconversion`** kompilyator flaglaridan foydalaning (GCC/Clang) — bu yashirin, xavfli cast'larni ogohlantiradi:
   ```bash
   gcc -Wall -Wextra -Wconversion -o dastur dastur.c
   ```
5. **Type punning uchun `memcpy` yoki `union` ishlating**, raw pointer cast emas.
6. **Floating-point sonlarni `==` bilan solishtirmang**, tolerance (epsilon) ishlating.
7. **`<stdint.h>` turlaridan foydalaning** (`int32_t`, `uint64_t`, `uintptr_t`), platformalararo bir xillik uchun.
8. **Katta turdan kichikka cast qilishdan oldin, qiymat diapazonga sig'ishini tekshiring.**

---

## 14. Xulosa va Mashqlar

### Xulosa jadvali

| Holat | Xavfsizmi? | Izoh |
|---|---|---|
| int -> double | Ha (odatda) | Juda katta intlarda aniqlik yo'qolishi mumkin |
| double -> int | Yo'q | Kasr qismi kesiladi, overflow bo'lsa UB |
| kichik -> katta signed | Ha | Sign extension bilan |
| katta -> kichik | Yo'q | Truncation, ma'lumot yo'qolishi mumkin |
| signed -> unsigned | Ehtiyot bilan | Bit pattern qayta talqin qilinadi |
| pointer -> boshqa turdagi pointer | Ehtiyot bilan | Strict aliasing qoidasiga e'tibor bering |
| void* -> boshqa pointer | Ha (C da) | C++ da explicit cast talab qilinadi |

### Mashqlar

1. Quyidagi kod nima chiqaradi va nima uchun?
```c
unsigned char a = 250;
a = a + 10;
printf("%d\n", a);
```

2. `int` massividagi barcha elementlarning o'rtachasini `double` sifatida to'g'ri hisoblovchi funksiya yozing (int bo'linish xatosisiz).

3. Quyidagi sikl nima uchun cheksiz aylanadi va uni qanday tuzatish mumkin?
```c
for (unsigned int i = 5; i >= 0; i--) {
    printf("%u\n", i);
}
```

4. `float` va `int` pointerlari orasida `memcpy` yordamida xavfsiz "type punning" bajaruvchi kod yozing.

5. Nima uchun `(int)(0.1 + 0.2) * 10 == 3` ifodasi ba'zan noto'g'ri natija berishi mumkin? Tushuntiring va to'g'ri yechim taklif qiling.

---

**Keyingi mavzu tavsiyasi:** Agar xohlasangiz, keyingi darsda **Bitwise Operators (bitli operatorlar)** yoki **Union va Type Punning**ni chuqurroq o'rganishimiz mumkin — bular casting mavzusi bilan bevosita bog'liq.
