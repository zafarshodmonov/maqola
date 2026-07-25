# C va C++ da Ma'lumot Turlari (Data Types): To'liq Qo'llanma

## Mundarija

1. Nega bu mavzu muhim?
2. Bit va bayt asoslari
3. Sonlarning binary tizimda ifodalanishi
4. Fundamental (asosiy) data types
5. `sizeof` operatori
6. Signed va Unsigned turlari
7. Two's Complement — signed sonlarning ichki tuzilishi
8. Overflow: signed vs unsigned xatti-harakati
9. Type conversion va integer promotion
10. Platformaga bog'liqlik muammosi
11. Fixed-width types (`stdint.h`)
12. Amaliy misollar (C va C++)
13. Best practices va xulosa

---

## 1. Nega bu mavzu muhim?

C va C++ — bu "yaqin metall" (close to metal) tillar. Ya'ni, siz yozgan har bir o'zgaruvchi xotirada aniq nechta bayt (byte) egallashini, u qanday bitlar ketma-ketligi bilan ifodalanishini bilishingiz kerak. Python yoki JavaScript kabi tillarda integer turi "cheksiz" kattalikda bo'lishi mumkin va til buni siz uchun avtomatik boshqaradi. C/C++ da esa bu boshqacha: har bir tur qat'iy o'lchamga (fixed size) ega, va agar siz bu chegaralardan chiqib ketsangiz, natija kutilmagan (undefined yoki implementation-defined) bo'lishi mumkin.

Bu mavzuni chuqur tushunish quyidagilar uchun zarur:
- Xotirani samarali ishlatish (memory-constrained tizimlar, embedded systems)
- Integer overflow kabi xatolarning oldini olish (bu xavfsizlik zaifliklariga olib kelishi mumkin)
- Turli platformalar orasida portable kod yozish
- Bit manipulation, low-level optimizatsiya va network protocol dasturlash

---

## 2. Bit va bayt asoslari

Kompyuter xotirasining eng kichik birligi — **bit** (binary digit). Bitning qiymati faqat 2 ta bo'lishi mumkin: `0` yoki `1`.

**Bayt (byte)** — odatda 8 ta bitdan tashkil topgan guruh. Deyarli barcha zamonaviy tizimlarda `1 bayt = 8 bit` deb qabul qilingan (bu C standartida `CHAR_BIT` konstantasi orqali ifodalanadi, va u har doim 8 bo'lishi shart emas, lekin amalda 99% hollarda 8 ga teng).

8 bitli bayt necha xil qiymatni ifodalay oladi?

```
2^8 = 256 ta qiymat (0 dan 255 gacha, agar unsigned bo'lsa)
```

Umumiy formula: agar bir necha bayt (yoki bit)ni birlashtirsak, ular ifodalay oladigan qiymatlar soni:

```
2^(bit soni)
```

| Bit soni | Bayt soni | Qiymatlar soni | Nomi |
|---|---|---|---|
| 8 bit | 1 bayt | 256 | byte |
| 16 bit | 2 bayt | 65,536 | halfword |
| 32 bit | 4 bayt | 4,294,967,296 | word |
| 64 bit | 8 bayt | ~1.8 × 10^19 | doubleword |

---

## 3. Sonlarning binary tizimda ifodalanishi

Kundalik hayotda biz **decimal** (o'nlik, 10 asosli) tizimdan foydalanamiz. Kompyuter esa ichki jarayonlarda **binary** (ikkilik, 2 asosli) tizimdan foydalanadi, chunki elektron sxemalar faqat ikkita holatni ("kuchlanish bor" / "kuchlanish yo'q") ishonchli ravishda farqlay oladi.

Har qanday pozitsion sanoq tizimida son quyidagi formula bilan hisoblanadi:

```
qiymat = Σ (raqam_i × asos^pozitsiya_i)
```

Masalan, binary sondagi `1011`:

```
1011 (binary) = 1×2^3 + 0×2^2 + 1×2^1 + 1×2^0
             = 8 + 0 + 2 + 1
             = 11 (decimal)
```

1 baytlik (8 bitli) unsigned son misolida, `00000000` dan `11111111` gacha (0 dan 255 gacha) qiymatlarni ifodalay oladi:

```
11111111 (binary) = 128+64+32+16+8+4+2+1 = 255 (decimal)
```

Bundan tashqari, kompyuter dasturlashda **hexadecimal** (16 asosli) tizim ham keng qo'llaniladi, chunki u binary bilan qulay moslashadi (har bir hex raqam aynan 4 bitga to'g'ri keladi):

```
0xFF (hex) = 1111 1111 (binary) = 255 (decimal)
```

C/C++ da literal yozish shakllari:
```c
int a = 42;        // decimal
int b = 0x2A;       // hexadecimal (0x prefiksi)
int c = 052;        // octal (0 prefiksi) — 42 emas, 8-asosli!
int d = 0b101010;   // binary (C++14 dan, GCC/Clang extension sifatida ham C da bor)
```

---

## 4. Fundamental (asosiy) data types

C/C++ standarti *aniq* bayt o'lchamlarini belgilamaydi — u faqat **minimal kafolatlangan diapazonlarni** belgilaydi. Ammo amalda ko'pchilik zamonaviy 64-bitli tizimlarda (Linux/Windows/macOS, GCC/Clang/MSVC) quyidagi o'lchamlar odatiy hisoblanadi:

### Butun sonlar (integer types)

| Tur | Odatiy o'lcham | Standart minimal diapazon | Izoh |
|---|---|---|---|
| `char` | 1 bayt (8 bit) | kamida -127...127 | signed yoki unsigned — bu compiler-ga bog'liq! |
| `signed char` | 1 bayt | -128...127 | aniq signed |
| `unsigned char` | 1 bayt | 0...255 | aniq unsigned |
| `short` (`short int`) | 2 bayt | kamida -32767...32767 | |
| `unsigned short` | 2 bayt | 0...65535 | |
| `int` | 4 bayt | kamida -32767...32767 | zamonaviy tizimlarda deyarli har doim 4 bayt |
| `unsigned int` | 4 bayt | 0...4294967295 | |
| `long` | 4 yoki 8 bayt | kamida -2147483647...2147483647 | Windows-da 4, Linux/macOS-da 8 bayt (LLP64 vs LP64) |
| `unsigned long` | 4 yoki 8 bayt | platformaga bog'liq | |
| `long long` | 8 bayt | kamida -9223372036854775807...9223372036854775807 | C99/C++11 dan standart, deyarli har doim 8 bayt |
| `unsigned long long` | 8 bayt | 0...18446744073709551615 | |

### Haqiqiy sonlar (floating-point types)

| Tur | Odatiy o'lcham | Standart | Aniqlik |
|---|---|---|---|
| `float` | 4 bayt (32 bit) | IEEE 754 single precision | ~6-7 decimal digit aniqlik |
| `double` | 8 bayt (64 bit) | IEEE 754 double precision | ~15-16 decimal digit aniqlik |
| `long double` | 8, 12 yoki 16 bayt | platformaga bog'liq | x86 da ko'pincha 80-bit extended precision, 16 baytga align qilingan |

### Boshqa asosiy turlar

| Tur | O'lcham | Izoh |
|---|---|---|
| `bool` (C++) / `_Bool` (C99) | 1 bayt | faqat `true`(1)/`false`(0) qiymat oladi, lekin xotirada kamida 1 bayt egallaydi |
| `void` | 0 bayt | "tur yo'qligi"ni bildiradi, o'zgaruvchi yaratib bo'lmaydi |
| `wchar_t` | 2 yoki 4 bayt | keng belgi (wide character), platformaga bog'liq |

**Muhim eslatma:** Standart faqat quyidagi tartibni kafolatlaydi:
```
sizeof(char) ≤ sizeof(short) ≤ sizeof(int) ≤ sizeof(long) ≤ sizeof(long long)
```
Aniq sonlarni emas! Shu sababli portable kod yozishda `sizeof()` yoki fixed-width types (bo'lim 11) ishlatish tavsiya etiladi.

---

## 5. `sizeof` operatori

`sizeof` — bu compile-time operator (funksiya emas!), u berilgan tur yoki o'zgaruvchi nechta bayt egallashini qaytaradi.

```c
#include <stdio.h>

int main(void) {
    printf("sizeof(char)      = %zu bayt\n", sizeof(char));
    printf("sizeof(short)     = %zu bayt\n", sizeof(short));
    printf("sizeof(int)       = %zu bayt\n", sizeof(int));
    printf("sizeof(long)      = %zu bayt\n", sizeof(long));
    printf("sizeof(long long) = %zu bayt\n", sizeof(long long));
    printf("sizeof(float)     = %zu bayt\n", sizeof(float));
    printf("sizeof(double)    = %zu bayt\n", sizeof(double));
    printf("sizeof(void*)     = %zu bayt\n", sizeof(void*));
    return 0;
}
```

`%zu` format spetsifikatori `size_t` turi uchun ishlatiladi (`sizeof` natijasi aynan shu turda qaytadi — bu unsigned integer turi).

**Diqqat:** `sizeof` natijasi doim `size_t` (unsigned) bo'lgani uchun, uni signed son bilan solishtirishda ehtiyot bo'ling:

```c
int n = -1;
if (n < sizeof(int)) {  // XATO! n avtomatik unsigned-ga o'giriladi
    // bu shart HECH QACHON true bo'lmaydi, chunki -1 -> juda katta unsigned son
}
```

---

## 6. Signed va Unsigned turlari

Har bir integer turi ikki xil "rejim"da bo'lishi mumkin:

- **signed** — manfiy va musbat sonlarni ifodalaydi (masalan, `-128` dan `127` gacha)
- **unsigned** — faqat manfiy bo'lmagan sonlarni ifodalaydi, ammo shu tufayli musbat diapazon 2 barobar kattaroq (masalan, `0` dan `255` gacha)

Buning sababi oddiy: bitlar soni bir xil (masalan, 8 bit = 256 ta kombinatsiya), lekin signed turida bitlarning yarmi manfiy sonlar uchun "band" qilinadi.

### 1 baytlik (8-bit) misol:

| Tur | Diapazon | Formula |
|---|---|---|
| `unsigned char` | 0 dan 255 gacha | 0 dan (2^8 - 1) gacha |
| `signed char` | -128 dan 127 gacha | -(2^7) dan (2^7 - 1) gacha |

Umumiy formula, `n` bitlik tur uchun:

```
unsigned: 0 dan (2^n - 1) gacha
signed:   -(2^(n-1)) dan (2^(n-1) - 1) gacha
```

Diqqat qiling — signed turida manfiy tomon bitta ko'proq qiymat oladi (`-128` bor, lekin `+128` yo'q, faqat `127` gacha). Buning sababi bo'lim 7 da tushuntiriladi (Two's Complement).

### Misol: 4 baytlik (32-bit) `int`

```
signed int:   -2,147,483,648 dan 2,147,483,647 gacha
unsigned int:  0 dan 4,294,967,295 gacha
```

### `char` ning maxsus holati

C/C++ standartida `char`, `signed char` va `unsigned char` — bular **3 ta har xil tur** hisoblanadi (garchi hammasi 1 bayt bo'lsa ham). `char` ning o'zi signed yoki unsigned ekanligi **compiler va platformaga bog'liq** (masalan, x86 da GCC odatda `char`ni signed qiladi, ARM da esa ko'pincha unsigned). Shu sababli, agar `char`ni son sifatida ishlatmoqchi bo'lsangiz va aniq xatti-harakat kerak bo'lsa, har doim `signed char` yoki `unsigned char`ni aniq ko'rsating.

---

## 7. Two's Complement — signed sonlarning ichki tuzilishi

Zamonaviy deyarli barcha protsessorlar (va C++20 standarti rasman) signed integerlarni **two's complement** usulida saqlaydi.

### Qanday ishlaydi?

8-bitli sonda eng yuqori (chap) bit — **sign bit** deb ataladi:
- `0` — musbat son
- `1` — manfiy son

Manfiy sonni olish algoritmi ("negation"):
1. Barcha bitlarni teskarisiga o'girish (invert / NOT)
2. Natijaga 1 qo'shish

**Misol:** `5` ning manfiysi (`-5`) ni topamiz (8 bit):

```
   5  = 00000101

Qadam 1 — barcha bitlarni invert qilish (one's complement):
        11111010

Qadam 2 — 1 qo'shish:
        11111010
      +        1
      ----------
        11111011  =  -5 (two's complement ko'rinishida)
```

Tekshirib ko'ramiz: `00000101 (5) + 11111011 (-5) = 100000000`. Bu 8 bitga sig'maydi (9-bitli natija), ortiqcha bit "carry out" qilib tashlab yuboriladi, va qolgan 8 bit — `00000000` = 0. To'g'ri! `5 + (-5) = 0`.

### Nega Two's Complement qulay?

1. **Arifmetika soddalashadi** — protsessor qo'shish (addition) uchun bitta sxemadan foydalanadi, signed va unsigned uchun alohida "ayirish" mantiqi kerak emas.
2. **Faqat bitta nol bor** (`+0` va `-0` alohida emas — one's complement usulida bo'lgani kabi emas).
3. Shu sababli signed diapazon assimetrik: `-128` bor, lekin `+128` yo'q — chunki `10000000` bittagina "eng katta manfiy son" uchun ishlatiladi, ikkita nolga sarflanmaydi.

### Sign bit orqali diapazonni tushunish

```
8-bit signed:
0xxxxxxx  ->  0 dan 127 gacha (musbat)
1xxxxxxx  ->  -128 dan -1 gacha (manfiy)
```

Eng yuqori bit — sign bit — nafaqat belgi, balki qiymatning bir qismi hisoblanadi: `-2^(n-1)` og'irligiga ega.

---

## 8. Overflow: signed vs unsigned xatti-harakati

Bu C/C++ dasturlashdagi eng ko'p uchraydigan va eng xavfli xatolardan biri.

### Unsigned overflow — **wraparound** (belgilangan xatti-harakat)

Unsigned turlar uchun standart aniq belgilaydi: agar natija diapazondan chiqsa, u **modulo 2^n** qoidasi bo'yicha "aylanib" o'tadi (wraparound). Bu **undefined behavior emas** — bu to'liq predictable (bashorat qilinadigan) xatti-harakat.

```c
unsigned char x = 255;
x = x + 1;
// natija: 0 (chunki 256 mod 256 = 0)

unsigned char y = 0;
y = y - 1;
// natija: 255 (0 dan "orqaga" aylanib, eng katta qiymatga o'tadi)
```

Bu xususiyat ba'zan **atayin** ishlatiladi (masalan, hash funksiyalarda, circular buffer indekslarida), lekin ko'pincha kutilmagan xatolarga sabab bo'ladi.

### Signed overflow — **Undefined Behavior (UB)**

Bu juda muhim farq: signed integer overflow C/C++ standartida **undefined behavior** hisoblanadi. Ya'ni, dastur *nazariy jihatdan istalgan narsa* qilishi mumkin — noto'g'ri natija berishi, crash bo'lishi, yoki hatto compiler optimizatsiya paytida butun kod bo'lagini "yo'q qilib tashlashi" mumkin.

```c
int x = 2147483647;  // INT_MAX
x = x + 1;
// UNDEFINED BEHAVIOR! Amalda ko'pincha -2147483648 ga aylanadi,
// lekin standart buni KAFOLATLAMAYDI, va optimizatsiya rejimida
// compiler bu qatorni "hech qachon sodir bo'lmaydigan holat" deb hisoblab,
// atrofidagi kodni kutilmagan tarzda o'zgartirishi mumkin.
```

**Nega bunday?** Tarixiy sabab: barcha protsessorlar signed overflow uchun bir xil xatti-harakatga ega emas edi (ba'zilari two's complement emas, one's complement yoki sign-magnitude ishlatgan). Standart bu noaniqlikni "UB" deb belgilab, compiler'larga optimizatsiya erkinligini berdi — masalan, `x + 1 > x` degan tekshiruvni compiler har doim "true" deb qabul qilishi mumkin, chunki overflow "sodir bo'lmasligi kerak" deb faraz qilinadi.

### Amaliy xulosa

```c
// XATO-GA MOYIL:
int a = INT_MAX;
if (a + 1 < a) { /* overflow ni aniqlashga urinish — bu ham UB! */ }

// TO'G'RI YECHIM (overflow oldindan tekshirish):
#include <limits.h>
if (a > INT_MAX - 1) {
    printf("Overflow bo'lardi!\n");
} else {
    a = a + 1;
}

// YOKI C23/GCC/Clang built-in funksiyalardan foydalanish:
int result;
if (__builtin_add_overflow(a, 1, &result)) {
    printf("Overflow!\n");
}
```

---

## 9. Type conversion va integer promotion

C/C++ da turli turdagi qiymatlar bir amalda ishtirok etganda, avtomatik konvertatsiya (**implicit conversion**) sodir bo'ladi.

### Integer promotion

`char`, `short` kabi "kichik" turlar arifmetik amallarda avtomatik ravishda `int` (yoki `unsigned int`) ga ko'tariladi:

```c
char a = 100, b = 100;
int result = a + b;  // a va b avval int-ga promote qilinadi, keyin qo'shiladi
// natija: 200 (char overflow bo'lmaydi, chunki amal int darajasida bajariladi)
```

### Signed/unsigned aralashganda — xavfli holat

Agar bir amalda signed va unsigned son ishtirok etsa, **signed son avtomatik unsigned-ga o'giriladi** (agar ikkala tur bir xil "rank"da bo'lsa):

```c
int a = -1;
unsigned int b = 1;

if (a < b) {
    printf("a kichik\n");
} else {
    printf("a katta yoki teng\n");  // AYNAN SHU chop etiladi!
}
// Sabab: -1, unsigned-ga o'girilganda 4294967295 ga aylanadi,
// va 4294967295 > 1 bo'ladi.
```

Bu — kompyuter xavfsizligida (security) eng ko'p uchraydigan bug turlaridan biri (masalan, buffer overflow zaifliklarida array indexini unsigned bilan solishtirishda).

### Explicit conversion (cast)

```c
double d = 3.99;
int i = (int)d;      // C-style cast: 3 (kesib tashlanadi, yaxlitlanmaydi!)
int j = static_cast<int>(d);  // C++ style, xuddi shu natija: 3

float f = (float)10 / 3;  // 3.333333...
```

**Muhim:** `double` dan `int`ga o'tishda qiymat **kesiladi** (truncation), yaxlitlanmaydi: `3.99 -> 3`, `-3.99 -> -3` (nolga tomon kesiladi).

---

## 10. Platformaga bog'liqlik muammosi

C/C++ standartlari atayin "qattiq" o'lchamlarni belgilamagan, chunki til turli protsessor arxitekturalarida (8-bitli mikrokontrollerlardan tortib 64-bitli superkompyuterlargacha) samarali ishlashi kerak edi.

Ikkita mashhur "data model" mavjud:

| Model | `int` | `long` | pointer | Qayerda ishlatiladi |
|---|---|---|---|---|
| **LP64** | 4 bayt | 8 bayt | 8 bayt | Linux, macOS (64-bit) |
| **LLP64** | 4 bayt | 4 bayt | 8 bayt | Windows (64-bit) |

Shu sababli, `long` turi Linux-da 8 bayt, Windows-da esa 4 bayt bo'lishi mumkin — bu ko'plab "portability bug"larining manbai.

**Xulosa:** Agar sizga aniq o'lcham muhim bo'lsa (masalan, fayl formatlari, network protokollar, yoki cross-platform kod uchun), hech qachon oddiy `int`/`long`ga tayanmang — bo'lim 11 dagi fixed-width turlardan foydalaning.

---

## 11. Fixed-width types (`<stdint.h>` / `<cstdint>`)

C99 va C++11 dan boshlab, standart kutubxona aniq o'lchamli turlarni taqdim etadi:

```c
#include <stdint.h>   // C uchun
// #include <cstdint>  // C++ uchun

int8_t   a;   // aniq 1 bayt, signed  (-128 ... 127)
uint8_t  b;   // aniq 1 bayt, unsigned (0 ... 255)
int16_t  c;   // aniq 2 bayt, signed
uint16_t d;   // aniq 2 bayt, unsigned
int32_t  e;   // aniq 4 bayt, signed
uint32_t f;   // aniq 4 bayt, unsigned
int64_t  g;   // aniq 8 bayt, signed
uint64_t h;   // aniq 8 bayt, unsigned
```

Bulardan tashqari foydali turlar:

```c
size_t     // sizeof natijasi qaytaradigan tur, har doim unsigned,
           // xotira o'lchamlarini ifodalash uchun ideal
ptrdiff_t  // ikki pointer orasidagi farqni ifodalaydi, signed
intptr_t   // pointer qiymatini butun songa aylantirish uchun (signed)
uintptr_t  // xuddi shu, lekin unsigned

int_fast32_t   // kamida 32 bit, lekin platformada "eng tez" ishlaydigan o'lcham
int_least32_t  // kamida 32 bit, lekin "eng kichik mumkin bo'lgan" o'lcham
```

Diapazon konstantalari `<limits.h>` (C) yoki `<climits>` (C++) da:

```c
#include <limits.h>
#include <stdio.h>

printf("INT_MIN = %d\n", INT_MIN);
printf("INT_MAX = %d\n", INT_MAX);
printf("UINT_MAX = %u\n", UINT_MAX);
printf("CHAR_BIT = %d\n", CHAR_BIT);  // odatda 8
```

Yoki C++ da `<limits>` orqali template-based yechim:

```cpp
#include <limits>
#include <iostream>

std::cout << std::numeric_limits<int>::min() << "\n";
std::cout << std::numeric_limits<int>::max() << "\n";
std::cout << std::numeric_limits<unsigned int>::max() << "\n";
```

---

## 12. Amaliy misollar

### 12.1. Har bir turning bayt o'lchami va diapazonini chiqarish (C)

```c
#include <stdio.h>
#include <limits.h>
#include <float.h>

int main(void) {
    printf("=== BUTUN SONLAR ===\n");
    printf("char:      %2zu bayt, [%d, %d]\n", sizeof(char), CHAR_MIN, CHAR_MAX);
    printf("short:     %2zu bayt, [%d, %d]\n", sizeof(short), SHRT_MIN, SHRT_MAX);
    printf("int:       %2zu bayt, [%d, %d]\n", sizeof(int), INT_MIN, INT_MAX);
    printf("long:      %2zu bayt, [%ld, %ld]\n", sizeof(long), LONG_MIN, LONG_MAX);
    printf("long long: %2zu bayt, [%lld, %lld]\n", sizeof(long long), LLONG_MIN, LLONG_MAX);

    printf("\n=== HAQIQIY SONLAR ===\n");
    printf("float:     %2zu bayt, aniqlik ~%d digit\n", sizeof(float), FLT_DIG);
    printf("double:    %2zu bayt, aniqlik ~%d digit\n", sizeof(double), DBL_DIG);

    return 0;
}
```

### 12.2. Signed overflow ni xavfsiz tekshirish (C++)

```cpp
#include <iostream>
#include <limits>
#include <optional>

std::optional<int> safe_add(int a, int b) {
    if (b > 0 && a > std::numeric_limits<int>::max() - b) {
        return std::nullopt;  // overflow bo'lardi
    }
    if (b < 0 && a < std::numeric_limits<int>::min() - b) {
        return std::nullopt;
    }
    return a + b;
}

int main() {
    auto r1 = safe_add(2000000000, 2000000000);
    if (!r1) std::cout << "Overflow aniqlandi!\n";

    auto r2 = safe_add(5, 10);
    if (r2) std::cout << "Natija: " << *r2 << "\n";

    return 0;
}
```

### 12.3. Unsigned wraparound-dan foydalanish (circular buffer indeksi)

```c
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint8_t index = 0;
    // 250 marta orqaga o'tkazamiz, wraparound tufayli
    // index avtomatik 0-255 oralig'ida "aylanadi"
    for (int i = 0; i < 260; i++) {
        index++;  // 255 dan keyin avtomatik 0 ga qaytadi
    }
    printf("Yakuniy index: %u\n", index);  // 260 mod 256 = 4
    return 0;
}
```

### 12.4. Signed/unsigned taqqoslash xatosi va uni tuzatish

```c
#include <stdio.h>

int main(void) {
    int a = -1;
    unsigned int b = 1;

    // XATO YO'L:
    if (a < b) {
        printf("Kutilmagan: a < b (chunki -1 unsigned-ga aylanadi)\n");
    }

    // TO'G'RI YO'L: ikkalasini bir xil signed turga keltirib solishtirish
    if ((long long)a < (long long)b) {
        printf("To'g'ri: a haqiqatan ham kichik\n");
    }

    return 0;
}
```

---

## 13. Best practices va xulosa

1. **Aniq o'lcham kerak bo'lsa** — har doim `int32_t`, `uint64_t` kabi fixed-width turlardan foydalaning, oddiy `int`/`long`ga tayanmang.
2. **Signed va unsigned turlarni aralashtirmang** — ayniqsa taqqoslash (`<`, `>`) amallarida. Compiler warning-larni (`-Wsign-compare`) yoqib qo'ying va e'tiborsiz qoldirmang.
3. **Massiv indekslash uchun** `size_t` dan foydalaning (u unsigned va xotira o'lchamlariga mos).
4. **Signed overflow'dan qo'rqing** — bu UB, shunchaki "noto'g'ri natija" emas, balki compiler kodni kutilmagan tarzda optimallashtirishi mumkin bo'lgan xavfli holat.
5. **`char`ni son sifatida ishlatmang** — agar sizga son kerak bo'lsa, `signed char` yoki `unsigned char`ni aniq belgilang, chunki oddiy `char` ning signedligi platformaga bog'liq.
6. **`sizeof` natijasini unsigned deb yodda tuting** — uni signed son bilan to'g'ridan-to'g'ri solishtirmang.
7. **Portable kod uchun** — hech qachon `sizeof(int) == 4` deb faraz qilmang, buning o'rniga `<stdint.h>` dagi turlarni ishlating yoki runtime'da `sizeof` orqali tekshiring.
8. **Overflow tekshirish kerak bo'lsa**, GCC/Clang'dagi `__builtin_add_overflow` va shunga o'xshash built-in funksiyalardan, yoki C++23 dagi `std::add_sat` kabi vositalardan foydalaning.

Ushbu darslikda ko'rilgan tushunchalar — bit/bayt asoslari, two's complement, signed/unsigned farqi, overflow xatti-harakati va fixed-width turlar — C/C++ da xavfsiz va samarali dastur yozishning poydevori hisoblanadi. Bu bilimlar ayniqsa competitive programming'da (overflow xatolaridan qochish), systems programming'da (xotira boshqaruvi) va xavfsizlik-muhim kodda (security-critical code) juda muhim rol o'ynaydi.
