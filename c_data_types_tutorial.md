# C Dasturlash Tilida Data Types (Ma'lumotlar Turlari)

## Mundarija
1. Kirish: Data Type nima va nima uchun kerak
2. Data Types klassifikatsiyasi
3. Asosiy (Primary/Basic) tiplar
4. Type Modifiers (signed, unsigned, short, long)
5. `sizeof` operatori
6. Butun sonlarning ikkilik (binary) tasviri
7. Signed va Unsigned: Two's Complement
8. Floating Point sonlar (IEEE 754)
9. `limits.h` va `float.h`: tip chegaralari
10. Literallar (Constants) va suffikslar
11. `void` tipi
12. Derived Types: array, pointer, function (qisqacha)
13. `enum` — Enumeration Types
14. `typedef` bilan nomlash
15. `stdint.h` — Platformadan mustaqil tiplar
16. Amaliy misollar
17. Xulosa va best practices

---

## 1. Kirish: Data Type nima va nima uchun kerak

Kompyuter xotirasi — bu millionlab kichik "katakchalar" (bytelar) yig'indisi bo'lib, har biri faqat 0 va 1 lardan iborat bitlarni saqlaydi. Xotiraning o'zi hech qanday ma'noga ega emas — u shunchaki bitlar ketma-ketligi. Masalan, xotirada saqlangan `01000001` degan sakkiz bit nimani anglatadi? Bu:

- Butun son sifatida: **65**
- ASCII belgi sifatida: **'A'**
- Ba'zi kontekstda: rangning bir qismi yoki boshqa narsa

**Data type (ma'lumot turi)** — bu kompilyatorga ushbu bitlar ketma-ketligini **qanday talqin qilish**, xotirada **qancha joy ajratish** va u ustida **qanday amallar** bajarish mumkinligini aytadigan qoidalar to'plami.

C tili — **statik va kuchli tiplashtirilgan (statically, strongly typed)** til. Bu shuni anglatadiki:
- Har bir o'zgaruvchi e'lon qilinganda tipi aniq ko'rsatilishi kerak (statik)
- Kompilyator turli tiplar orasidagi mos kelmaydigan amallarni (masalan, pointerni to'g'ridan-to'g'ri structga qo'shish) xatolik deb hisoblaydi (kuchli)

Data type uchta asosiy narsani belgilaydi:
1. **Xotira hajmi** (necha bayt ajratiladi)
2. **Talqin qilish usuli** (bitlar nimani anglatadi — butun son, kasr son, belgi)
3. **Ruxsat etilgan amallar** (masalan, `float` ustida modul olish `%` mumkin emas)

---

## 2. Data Types klassifikatsiyasi

C tilidagi tiplarni quyidagicha guruhlash mumkin:

```
Data Types
│
├── Primary (Basic) Types
│   ├── int
│   ├── char
│   ├── float
│   ├── double
│   └── void
│
├── Derived Types
│   ├── Array
│   ├── Pointer
│   ├── Function
│   └── Structure/Union (texnik jihatdan derived hisoblanadi)
│
├── User-defined Types
│   ├── struct
│   ├── union
│   ├── enum
│   └── typedef (yangi nom, yangi tip emas)
│
└── Qo'shimcha (Type Qualifiers/Modifiers)
    ├── signed / unsigned
    ├── short / long / long long
    └── const / volatile (qualifierlar)
```

Ushbu darslikda asosan **primary types** va ularning **modifierlari** ustida to'xtalamiz, derived va user-defined tiplarni esa qisqacha ko'rib chiqamiz (chunki `struct`, array va pointerlar alohida chuqur mavzular).

---

## 3. Asosiy (Primary/Basic) Tiplar

C tilida beshta asosiy tip mavjud:

| Tip | Nima saqlaydi | Odatiy hajm (64-bit tizim) |
|---|---|---|
| `char` | Bitta belgi (yoki kichik butun son) | 1 bayt |
| `int` | Butun son | 4 bayt |
| `float` | Kasr son (single precision) | 4 bayt |
| `double` | Kasr son (double precision, aniqroq) | 8 bayt |
| `void` | "Tip yo'q" — bo'sh qiymat | — |

### Misol kod

```c
#include <stdio.h>

int main(void) {
    char letter = 'A';
    int age = 25;
    float pi_approx = 3.14f;
    double pi_precise = 3.14159265358979;

    printf("letter = %c\n", letter);
    printf("age = %d\n", age);
    printf("pi_approx = %f\n", pi_approx);
    printf("pi_precise = %lf\n", pi_precise);

    return 0;
}
```

Diqqat qiling: C standarti tiplarning **aniq bайt hajmini kafolatlamaydi** — u faqat **minimal diapazonni** kafolatlaydi. Haqiqiy hajm platforma va kompilyatorga bog'liq (buni 9-bo'limda batafsil ko'ramiz).

---

## 4. Type Modifiers (signed, unsigned, short, long)

`int` va `char` tiplariga **modifier**lar qo'shib, ularning diapazoni va xotira hajmini o'zgartirish mumkin:

- `signed` — musbat va manfiy qiymatlarni saqlaydi (default holat, `int` uchun)
- `unsigned` — faqat nolь va musbat qiymatlarni saqlaydi, lekin diapazon kattaroq
- `short` — kamroq xotira, kichikroq diapazon
- `long` — ko'proq xotira, kattaroq diapazon
- `long long` — yanada kattaroq diapazon (C99 dan beri)

Bu modifierlarni birlashtirish mumkin:

```c
short int a;              // yoki shunchaki: short a;
unsigned short int b;     // yoki: unsigned short b;
long int c;               // yoki: long c;
unsigned long long int d; // yoki: unsigned long long d;
signed char e;
unsigned char f;
```

**Muhim qoida**: modifier qo'llanilmagan `int` — bu har doim `signed int` bilan bir xil. Lekin oddiy `char` tipi **signed yoki unsigned ekanligi kompilyatorga bog'liq** (bu tez-tez xatoliklarga sabab bo'ladi, shuning uchun belgi/son sifatida ishlatilganda `signed char` yoki `unsigned char` deb aniq yozish tavsiya etiladi).

### Tiplarning to'liq jadvali (odatiy 64-bit Linux/GCC tizimida)

| Tip | Hajm (bayt) | Diapazon |
|---|---|---|
| `char` | 1 | -128 dan 127 gacha (yoki 0–255, platformaga bog'liq) |
| `signed char` | 1 | -128 dan 127 gacha |
| `unsigned char` | 1 | 0 dan 255 gacha |
| `short int` | 2 | -32,768 dan 32,767 gacha |
| `unsigned short int` | 2 | 0 dan 65,535 gacha |
| `int` | 4 | -2,147,483,648 dan 2,147,483,647 gacha |
| `unsigned int` | 4 | 0 dan 4,294,967,295 gacha |
| `long int` | 8 (Linux) / 4 (Windows) | juda katta diapazon |
| `unsigned long int` | 8 (Linux) / 4 (Windows) | 0 dan juda katta songacha |
| `long long int` | 8 | ~-9.2×10¹⁸ dan ~9.2×10¹⁸ gacha |
| `unsigned long long int` | 8 | 0 dan ~1.8×10¹⁹ gacha |
| `float` | 4 | ~±3.4×10³⁸ (≈7 xona aniqlik) |
| `double` | 8 | ~±1.7×10³⁰⁸ (≈15-16 xona aniqlik) |
| `long double` | 8, 12 yoki 16 (platformaga bog'liq) | `double`dan kengroq |

---

## 5. `sizeof` Operatori

`sizeof` — bu **operator** (funksiya emas!), u istalgan tip yoki o'zgaruvchining bayt hajmini `size_t` (unsigned butun son) sifatida qaytaradi. Bu **compile-time**da hisoblanadi, ya'ni dastur ishga tushishidan oldin, kompilyatsiya vaqtida aniqlanadi.

```c
#include <stdio.h>

int main(void) {
    printf("char: %zu bayt\n", sizeof(char));
    printf("int: %zu bayt\n", sizeof(int));
    printf("long: %zu bayt\n", sizeof(long));
    printf("float: %zu bayt\n", sizeof(float));
    printf("double: %zu bayt\n", sizeof(double));

    int x = 10;
    printf("x o'zgaruvchisi: %zu bayt\n", sizeof(x));   // qavs bilan yoki qavssiz ishlaydi
    printf("x o'zgaruvchisi: %zu bayt\n", sizeof x);

    return 0;
}
```

> **Eslatma**: `%zu` — bu `size_t` tipini chop etish uchun `printf` format spesifikatori (`size_t` odatda `unsigned long` bilan bir xil).

`sizeof` natijasi **platformaga bog'liq** bo'lgani uchun, dastur portativ (turli platformalarda ishlaydigan) bo'lishi kerak bo'lsa, hech qachon `int` ning 4 bayt ekanligiga qattiq ishonmang — buning o'rniga har doim `sizeof` dan foydalaning yoki `stdint.h` dagi aniq tiplarni ishlating (16-bo'limga qarang).

---

## 6. Butun Sonlarning Ikkilik (Binary) Tasviri

Har qanday butun son xotirada **ikkilik sanoq sistemasida (binary)** saqlanadi. Masalan, 1 baytlik (`unsigned char`) xotira katakchasida 25 soni qanday saqlanadi:

```
Decimal:  25
Binary:   0 0 0 1 1 0 0 1
Bit:      7 6 5 4 3 2 1 0   (chapdan o'ngga, eng katta bit — MSB, eng kichik — LSB)
Qiymat:   0×128 + 0×64 + 0×32 + 1×16 + 1×8 + 0×4 + 0×2 + 1×1 = 25
```

**MSB** (Most Significant Bit) — eng chap, eng "og'ir" bit. **LSB** (Least Significant Bit) — eng o'ng bit.

`n` bitlik **unsigned** son uchun diapazon: `0` dan `2ⁿ - 1` gacha.
Masalan, 8 bit uchun: `0` dan `255` gacha (`2⁸ - 1 = 255`).

### Bitlarni ko'rish uchun dastur

```c
#include <stdio.h>

void print_binary(unsigned char num) {
    for (int i = 7; i >= 0; i--) {
        printf("%d", (num >> i) & 1);
    }
    printf("\n");
}

int main(void) {
    unsigned char x = 25;
    print_binary(x);   // 00011001
    return 0;
}
```

Bu yerda `>>` — o'ngga siljitish (right shift), `&` — bitwise AND operatori. Har bir bitni navbat bilan eng chapdan (bit 7) tekshirib chiqamiz.

---

## 7. Signed va Unsigned: Two's Complement

`unsigned` tiplarda manfiy sonlar yo'q — barcha bitlar musbat qiymatni ifodalash uchun ishlatiladi. Lekin `signed` tiplarda manfiy sonlarni ham saqlash kerak. Buning uchun zamonaviy kompyuterlarning deyarli barchasi **two's complement (ikkilik to'ldiruvchi)** usulini ishlatadi.

### Nega Two's Complement?

Muqobil usullar (masalan, "sign-magnitude" — birinchi bitni ishora uchun ajratish) muammoli: ular ikkita nol hosil qiladi (`+0` va `-0`) va arifmetik amallar murakkablashadi. Two's complement bu muammolarni hal qiladi va qo'shish/ayirish amallarini **bir xil sxema** bilan bajarish imkonini beradi.

### Qanday ishlaydi

`signed char` (8 bit) uchun eng chap bit **ishora biti (sign bit)** hisoblanadi:
- Eng chap bit `0` bo'lsa → son musbat (yoki nol)
- Eng chap bit `1` bo'lsa → son manfiy

Manfiy sonni hosil qilish uchun (masalan, `+5` dan `-5` ni topish):
1. Barcha bitlarni teskarisiga o'giring (invert / NOT): `0000 0101` → `1111 1010`
2. Natijaga 1 qo'shing: `1111 1010 + 1 = 1111 1011`

```
+5  = 0000 0101
-5  = 1111 1011   (two's complement)
```

Tekshirib ko'ramiz: `0000 0101 + 1111 1011 = 1 0000 0000`. Ortiqcha 9-bit chiqib ketadi (overflow sifatida e'tiborga olinmaydi), natija `0000 0000` = 0. To'g'ri, chunki `5 + (-5) = 0`.

### 8-bitlik signed char diapazoni nega -128 dan 127 gacha?

8 bit bilan jami `2⁸ = 256` xil kombinatsiya hosil qilish mumkin. Two's complementda bu kombinatsiyalar quyidagicha taqsimlanadi:
- `0000 0000` dan `0111 1111` gacha → `0` dan `127` gacha (128 ta musbat/nol qiymat)
- `1000 0000` dan `1111 1111` gacha → `-128` dan `-1` gacha (128 ta manfiy qiymat)

Diqqat qiling — `-128` bor, lekin `+128` yo'q (asimmetriya). Buning sababi: `0` faqat bitta kombinatsiyani (`0000 0000`) egallaydi, ikkinchisini esa (`1000 0000`) `-128` uchun "qarz" oladi.

### Signed/Unsigned aralashtirishning xavfi

```c
#include <stdio.h>

int main(void) {
    int a = -1;
    unsigned int b = 1;

    if (a < b) {
        printf("a kichik\n");
    } else {
        printf("a katta yoki teng\n");   // aslida shu chop etiladi!
    }
    return 0;
}
```

Bu yerda kutilmagan natija chiqadi, chunki taqqoslashda C **implicit conversion (yashirin konversiya)** qoidasiga ko'ra `signed int`ni `unsigned int`ga aylantiradi. `-1` ni `unsigned int`ga aylantirsak, u eng katta mumkin bo'lgan `unsigned int` qiymatiga (`4294967295`) aylanadi — chunki bitlar tasviri o'zgarmaydi, faqat talqin o'zgaradi! Natijada `4294967295 < 1` — noto'g'ri, shuning uchun else blok ishlaydi. Bu C tilidagi eng ko'p uchraydigan buglardan biri, shuning uchun signed va unsigned sonlarni birga taqqoslashdan saqlaning.

---

## 8. Floating Point Sonlar (IEEE 754)

`float` va `double` tiplari kasr sonlarni saqlash uchun **IEEE 754** standartiga asoslangan. Butun sonlardan farqli o'laroq, kasr sonlar uchta qismga bo'linadi:

```
[ Sign (1 bit) | Exponent (daraja) | Mantissa/Fraction (mantissa) ]
```

**32-bit `float`** uchun:
- 1 bit — ishora (sign)
- 8 bit — daraja (exponent)
- 23 bit — mantissa (aniqlik)

**64-bit `double`** uchun:
- 1 bit — ishora
- 11 bit — daraja
- 52 bit — mantissa

Bu deyarli xuddi ilmiy notatsiyaga o'xshaydi: `qiymat = (-1)^sign × 1.mantissa × 2^(exponent - bias)`.

### Nega `0.1 + 0.2 != 0.3`?

```c
#include <stdio.h>

int main(void) {
    double a = 0.1 + 0.2;
    printf("%.17f\n", a);   // 0.30000000000000004
    if (a == 0.3) {
        printf("Teng\n");
    } else {
        printf("Teng emas!\n");   // shu chop etiladi
    }
    return 0;
}
```

Sababi: `0.1` va `0.2` kabi o'nlik kasrlarni **ikkilik sanoq sistemasida aniq ifodalab bo'lmaydi** (xuddi 1/3 ni o'nlik sistemada aniq yozib bo'lmaganidek — `0.3333...` cheksiz davom etadi). Shuning uchun `float`/`double` bilan **hech qachon** `==` orqali to'g'ridan-to'g'ri tenglikni tekshirmang. Buning o'rniga:

```c
#include <math.h>

double epsilon = 1e-9;
if (fabs(a - 0.3) < epsilon) {
    printf("Amalda teng\n");
}
```

### `float` va `double` orasidagi tanlov

- `float` — kamroq xotira, tezroq (ba'zi GPU/embedded tizimlarda), lekin ~7 xonagacha aniqlik
- `double` — ko'proq xotira, ~15-16 xonagacha aniqlik, C da odatiy tanlov hisoblanadi (matematik kutubxonalar, `printf` ichida ham `float` avtomatik `double`ga aylantiriladi)

---

## 9. `limits.h` va `float.h`: Tip Chegaralari

C standarti aniq bayt hajmini emas, **minimal talablarni** belgilaydi. Haqiqiy chegaralarni bilish uchun standart kutubxona headerlaridan foydalaning — bu **portativ (ko'chma) kod** yozishning to'g'ri yo'li.

```c
#include <stdio.h>
#include <limits.h>
#include <float.h>

int main(void) {
    printf("INT_MIN = %d\n", INT_MIN);
    printf("INT_MAX = %d\n", INT_MAX);
    printf("CHAR_MIN = %d\n", CHAR_MIN);
    printf("CHAR_MAX = %d\n", CHAR_MAX);
    printf("LONG_MAX = %ld\n", LONG_MAX);
    printf("UINT_MAX = %u\n", UINT_MAX);

    printf("FLT_MAX = %e\n", FLT_MAX);
    printf("DBL_MAX = %e\n", DBL_MAX);
    printf("FLT_EPSILON = %e\n", FLT_EPSILON);   // eng kichik farqlanadigan qadam

    return 0;
}
```

Bu qiymatlarni **hech qachon qo'lda yozib qo'ymang** (masalan `2147483647` deb kod ichiga yozish) — buning o'rniga doim `INT_MAX` kabi makrolardan foydalaning, chunki bu kodni turli platformalarga moslashtiradi.

---

## 10. Literallar (Constants) va Suffikslar

Kodda to'g'ridan-to'g'ri yozilgan qiymatlar **literal (constant)** deb ataladi. Har bir literalning ham o'z tipi bor.

### Butun son literallari

```c
int a = 42;            // odatiy int
long b = 42L;           // long
unsigned int c = 42U;   // unsigned int
unsigned long d = 42UL; // unsigned long
long long e = 42LL;     // long long

int hex = 0x2A;   // 16-lik (hexadecimal) — 42 ga teng
int oct = 052;    // 8-lik (octal) — 42 ga teng
int bin = 0b101010; // 2-lik (binary, C23/GNU kengaytmasi) — 42 ga teng
```

### Kasr son literallari

```c
float f = 3.14f;    // 'f' suffiksisiz bu double bo'lardi!
double d = 3.14;     // default — double
long double ld = 3.14L;
```

**Muhim**: `f` suffiksisiz yozilgan har qanday kasr literal (`3.14`) avtomatik ravishda `double` hisoblanadi. Agar uni `float` o'zgaruvchiga bersangiz, kompilyator uni "narrowing conversion" orqali kichraytiradi (ba'zan ogohlantirish/warning bilan).

### Belgi (char) va satr (string) literallari

```c
char letter = 'A';       // bitta belgi, apostrof bilan
char newline = '\n';     // maxsus (escape) belgi
char *text = "Salom";    // satr, qo'shtirnoq bilan — bu aslida char massivi (array)
```

---

## 11. `void` Tipi

`void` — "hech narsa yo'q" degan ma'noni bildiradi va uch xil holatda ishlatiladi:

**1. Funksiya qiymat qaytarmasligini bildirish uchun:**
```c
void greet(void) {
    printf("Salom!\n");
    // return qiymat qaytarmaydi
}
```

**2. Funksiya parametr qabul qilmasligini bildirish uchun:**
```c
int get_value(void) {   // parametr yo'q
    return 42;
}
```

**3. "Generic pointer" (istalgan tipga ishora qiluvchi pointer) sifatida:**
```c
void *ptr;   // istalgan tipdagi manzilni saqlashi mumkin, lekin
             // to'g'ridan-to'g'ri dereferencing qilib bo'lmaydi
             // (avval kerakli tipga cast qilish kerak)

int x = 10;
void *generic = &x;
int *specific = (int *)generic;
printf("%d\n", *specific);   // 10
```

`malloc()` funksiyasi aynan `void *` qaytaradi — chunki u qaysi tipdagi ma'lumot uchun xotira ajratayotganini bilmaydi, buni chaqiruvchi kod hal qiladi.

---

## 12. Derived Types: Array, Pointer, Function (qisqacha)

Bu uchtasi asosiy tiplardan **hosila (derived)** hisoblanadi — ular asosiy tiplar ustiga qurilgan:

```c
int arr[5];        // array — bir xil tipdagi elementlar ketma-ketligi
int *ptr;           // pointer — xotira manzilini saqlaydi
int add(int, int);  // function — kiruvchi/chiquvchi tiplarga ega

char *name = "Ali";      // char massiviga pointer
int matrix[3][3];         // ikki o'lchamli array
int (*func_ptr)(int, int); // funksiyaga pointer
```

Bu mavzular (ayniqsa pointer va memory management) chuqur va alohida darslik talab qiladi — bu haqda batafsil to'xtalmoqchi bo'lsangiz, alohida so'rov qilishingiz mumkin.

---

## 13. `enum` — Enumeration Types

`enum` nomlangan butun son konstantalarini guruhlash uchun ishlatiladi — kodni o'qish osonroq bo'lishi uchun:

```c
#include <stdio.h>

enum Weekday { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY };

int main(void) {
    enum Weekday today = WEDNESDAY;
    printf("today = %d\n", today);   // 2 (default: MONDAY=0 dan boshlab ketma-ket)

    enum Color { RED = 1, GREEN = 5, BLUE };   // BLUE avtomatik 6 bo'ladi
    printf("BLUE = %d\n", BLUE);

    return 0;
}
```

Ichki holatda `enum` — bu shunchaki `int` (yoki kompilyatorga bog'liq boshqa butun tip), lekin u kodga semantik ma'no va o'qilishni yaxshilaydi (masalan, `2` yozish o'rniga `WEDNESDAY` yozish).

---

## 14. `typedef` Bilan Nomlash

`typedef` **yangi tip yaratmaydi** — u mavjud tipga **yangi nom (alias)** beradi. Bu ko'proq o'qilishi oson kod yozish uchun ishlatiladi:

```c
typedef unsigned long ulong;
typedef struct {
    int x;
    int y;
} Point;

ulong distance = 1000;
Point p1 = {3, 4};
```

`stdint.h` kabi standart headerlar ham aslida `typedef` yordamida yaratilgan (keyingi bo'limga qarang).

---

## 15. `stdint.h` — Platformadan Mustaqil Tiplar

`int` ning hajmi platforma va kompilyatorga qarab **o'zgarishi mumkinligini** ko'rdik. Agar sizga aniq bayt hajmiga ega tip kerak bo'lsa (masalan, tarmoq protokoli yoki fayl formatini yozayotgan bo'lsangiz), C99 standarti bilan kelgan `<stdint.h>` headerini ishlating:

```c
#include <stdio.h>
#include <stdint.h>

int main(void) {
    int8_t   a = -128;         // aniq 1 bayt, signed
    uint8_t  b = 255;          // aniq 1 bayt, unsigned
    int16_t  c = -32000;       // aniq 2 bayt
    uint32_t d = 4000000000U;  // aniq 4 bayt
    int64_t  e = -9000000000000000000LL; // aniq 8 bayt

    printf("%zu %zu %zu %zu %zu\n",
        sizeof(a), sizeof(b), sizeof(c), sizeof(d), sizeof(e));
    return 0;
}
```

| Tip | Kafolatlangan hajm |
|---|---|
| `int8_t` / `uint8_t` | 1 bayt |
| `int16_t` / `uint16_t` | 2 bayt |
| `int32_t` / `uint32_t` | 4 bayt |
| `int64_t` / `uint64_t` | 8 bayt |

Zamonaviy C kodlarida, ayniqsa **portativlik** muhim bo'lgan loyihalarda (masalan, embedded systems, network protocols, fayl formatlari), oddiy `int`/`long` o'rniga aynan shu tiplardan foydalanish tavsiya etiladi.

---

## 16. Amaliy Misollar

### 16.1. Integer Overflow (to'lib toshish)

```c
#include <stdio.h>
#include <limits.h>

int main(void) {
    int max = INT_MAX;
    printf("INT_MAX = %d\n", max);
    printf("INT_MAX + 1 = %d\n", max + 1);   // Undefined Behavior!
                                               // Amalda ko'pincha INT_MIN chiqadi
    unsigned int umax = UINT_MAX;
    printf("UINT_MAX + 1 = %u\n", umax + 1); // 0 — unsigned overflow
                                               // aniq belgilangan (wraps around)
    return 0;
}
```

**Muhim farq**: `signed` overflow — bu **Undefined Behavior (UB)**, ya'ni C standarti natijani kafolatlamaydi (kompilyator istalgan narsa qilishi mumkin). `unsigned` overflow esa **aniq belgilangan** — u har doim modul arifmetikasi bo'yicha "aylanib" (wrap around) qaytadi (`2^n` bo'yicha modul).

### 16.2. Turli tiplar hajmini chiqarish dasturi

```c
#include <stdio.h>

int main(void) {
    printf("%-20s %s\n", "Tip", "Hajm (bayt)");
    printf("%-20s %zu\n", "char", sizeof(char));
    printf("%-20s %zu\n", "short", sizeof(short));
    printf("%-20s %zu\n", "int", sizeof(int));
    printf("%-20s %zu\n", "long", sizeof(long));
    printf("%-20s %zu\n", "long long", sizeof(long long));
    printf("%-20s %zu\n", "float", sizeof(float));
    printf("%-20s %zu\n", "double", sizeof(double));
    printf("%-20s %zu\n", "long double", sizeof(long double));
    printf("%-20s %zu\n", "pointer (void*)", sizeof(void *));
    return 0;
}
```

### 16.3. Implicit conversion (yashirin konversiya) namunasi

```c
#include <stdio.h>

int main(void) {
    int a = 5;
    int b = 2;
    printf("%d\n", a / b);        // 2 — butun son bo'lish (integer division)

    float c = a / b;              // hisoblash int/int sifatida bajariladi (=2),
    printf("%f\n", c);            // keyin float'ga aylantiriladi => 2.000000

    float d = (float)a / b;       // avval a float'ga aylantiriladi => 2.5
    printf("%f\n", d);            // 2.500000

    return 0;
}
```

Bu misol C dagi **"usual arithmetic conversions"** qoidasini ko'rsatadi: ikkita butun son bo'linayotganda, natija ham butun son bo'ladi (kasr qismi tashlab yuboriladi) — kasr natija olish uchun kamida bitta operandni aniq `float`/`double`ga cast qilish kerak.

---

## 17. Xulosa va Best Practices

1. **Har doim `sizeof` yoki `stdint.h` dan foydalaning** — `int` ning 4 bayt ekanligiga qattiq ishonmang.
2. **Signed va unsigned sonlarni aralashtirmang** — taqqoslashda kutilmagan natijalarga olib kelishi mumkin.
3. **Kasr sonlarni `==` bilan taqqoslamang** — `fabs(a - b) < epsilon` dan foydalaning.
4. **Signed integer overflow — Undefined Behavior** — bunga tayanmang, chegaralarni tekshiring.
5. **Portativ kod uchun `int8_t`, `uint32_t` kabi aniq tiplarni** ishlating, ayniqsa fayl formatlari yoki tarmoq protokollarida.
6. **Kasr literallar uchun suffikslardan** foydalaning (`3.14f` — float, `3.14` — double, `42UL` — unsigned long) — bu implicit conversion xatolarining oldini oladi.
7. **`char` ning signed yoki unsigned ekanligi platformaga bog'liq** — belgi kodlari bilan ishlaganda buni yodda tuting.

Agar davom ettirishni istasangiz, tabiiy keyingi mavzular: **pointers va memory management**, **arrays va stringlar (chuqurroq)**, yoki **struct/union**. Qaysi birini xohlaysiz?
