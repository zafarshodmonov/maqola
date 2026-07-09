# C va C++ da Xotira Boshqaruvi va Pointerlar — To'liq Darslik

---

## Mundarija

1. [Kirish: Xotira boshqaruvi nima uchun kerak](#1-kirish)
2. [Kompyuter xotirasi asoslari: Stack va Heap](#2-xotira-asoslari)
3. [C tilida Pointerlar](#3-c-pointerlar)
4. [Pointer arifmetikasi](#4-pointer-arifmetikasi)
5. [Pointerlar va massivlar](#5-pointer-massiv)
6. [Pointer to Pointer (ko'rsatkichga ko'rsatkich)](#6-pointer-to-pointer)
7. [Function Pointer](#7-function-pointer)
8. [void*, NULL va xavfsizlik](#8-void-null)
9. [C da dinamik xotira: malloc, calloc, realloc, free](#9-dinamik-xotira-c)
10. [Tipik xatolar: memory leak, dangling pointer, double free](#10-tipik-xatolar)
11. [C++ da References (murojaatlar)](#11-cpp-references)
12. [C++ da new va delete](#12-cpp-new-delete)
13. [RAII tamoyili](#13-raii)
14. [Smart Pointerlar: unique_ptr, shared_ptr, weak_ptr](#14-smart-pointers)
15. [Debug qilish vositalari (Valgrind, Sanitizers)](#15-debug)
16. [Eng yaxshi amaliyotlar (Best Practices)](#16-best-practices)
17. [Mashqlar](#17-mashqlar)

---

<a name="1-kirish"></a>
## 1. Kirish: Xotira boshqaruvi nima uchun kerak

C va C++ tillarining eng katta kuchi — dasturchiga kompyuter xotirasini **to'g'ridan-to'g'ri boshqarish** imkonini berishidir. Python yoki Java kabi tillarda xotira avtomatik boshqariladi (garbage collector orqali), lekin C/C++ da bu ishni siz o'zingiz bajarasiz.

Bu nima degani?

- Siz xotirani **qachon** ajratishni (allocate) o'zingiz belgilaysiz.
- Siz xotirani **qachon** bo'shatishni (free/delete) o'zingiz belgilaysiz.
- Agar xato qilsangiz — dastur ishlamay qolishi, sekinlashishi yoki xavfsizlik teshigiga aylanishi mumkin.

**Nega bu muhim?**

- Operatsion tizimlar, drayverlar, o'yin dvigatellari, ma'lumotlar bazalari, real-time tizimlar — barchasi C/C++ da yoziladi, chunki bu tillar xotira ustidan to'liq nazorat beradi.
- Pointerlarni tushunmasdan turib, C/C++ da professional darajada dastur yozib bo'lmaydi.
- Zamonaviy C++ (C++11 va undan keyingi versiyalar) xotirani xavfsizroq boshqarish uchun **smart pointer**larni taqdim etadi — ammo ularning ichida nima sodir bo'layotganini tushunish uchun ham xom pointerlarni bilish shart.

Ushbu darslikda biz eng tubidan boshlab, xotira qanday tuzilganidan tortib, zamonaviy C++ smart pointerlarigacha bo'lgan yo'lni bosib o'tamiz.

---

<a name="2-xotira-asoslari"></a>
## 2. Kompyuter xotirasi asoslari: Stack va Heap

Dastur ishga tushganda, operatsion tizim unga xotira maydonini beradi. Bu xotira bir necha qismlarga bo'linadi:

```
Yuqori manzillar
+---------------------------+
|   Kernel space (OS uchun) |
+---------------------------+
|          STACK            |  <- pastga qarab o'sadi
|            |               |
|            v               |
|                            |
|            ^               |
|            |               |
|          HEAP             |  <- yuqoriga qarab o'sadi
+---------------------------+
|   BSS (initsializatsiya   |
|   qilinmagan globallar)   |
+---------------------------+
|   Data (initsializatsiya  |
|   qilingan globallar)     |
+---------------------------+
|   Text/Code (dastur kodi) |
+---------------------------+
Quyi manzillar
```

### 2.1. Stack (Steк)

- Funksiyalar chaqirilganda avtomatik yaratiladigan xotira.
- Lokal o'zgaruvchilar shu yerda saqlanadi.
- **Juda tez** ishlaydi (shunchaki bitta ko'rsatkichni siljitish orqali).
- Funksiya tugagach, o'zgaruvchilar **avtomatik** tozalanadi.
- Hajmi **cheklangan** (odatda bir necha megabayt). Shu sababli katta massivlarni stackda yaratish `stack overflow` xatosiga olib kelishi mumkin.

```c
void funksiya() {
    int x = 10;        // stack'da yaratiladi
    int massiv[100];   // stack'da yaratiladi
}   // funksiya tugagach x va massiv avtomatik yo'qoladi
```

### 2.2. Heap (Xip)

- Dasturchi **qo'lda** so'raydigan va **qo'lda** bo'shatadigan xotira.
- Stack'ga qaraganda **sekinroq**, lekin hajmi ancha **katta**.
- Xotira dastur tugagunga qadar yoki siz uni bo'shatmagunga qadar mavjud bo'lib turadi.
- C da: `malloc`, `calloc`, `realloc`, `free`.
- C++ da: `new`, `delete`.

```c
int *ptr = malloc(sizeof(int) * 100); // heap'da 100 ta int uchun joy
free(ptr); // ishlatib bo'lgach, majburiy bo'shatish kerak!
```

**Muhim farq:**

| Xususiyat | Stack | Heap |
|---|---|---|
| Tezlik | Juda tez | Nisbatan sekin |
| Hajmi | Kichik (MB) | Katta (GB gacha) |
| Boshqaruv | Avtomatik | Qo'lda (dasturchi) |
| Umr davomiyligi | Funksiya doirasida | O'zingiz belgilaysiz |
| Xavf | Stack overflow | Memory leak, dangling pointer |

---

<a name="3-c-pointerlar"></a>
## 3. C tilida Pointerlar

### 3.1. Pointer nima?

**Pointer (ko'rsatkich)** — bu boshqa o'zgaruvchining xotiradagi **manzilini** saqlaydigan o'zgaruvchidir.

Har bir o'zgaruvchi xotirada muayyan manzilga ega. Pointer o'sha manzilni "eslab qoladi".

```c
#include <stdio.h>

int main() {
    int son = 42;
    int *ptr = &son;  // ptr — son o'zgaruvchisining manzilini saqlaydi

    printf("son ning qiymati: %d\n", son);
    printf("son ning manzili: %p\n", (void*)&son);
    printf("ptr ning qiymati (manzil): %p\n", (void*)ptr);
    printf("ptr orqali qiymatni olish: %d\n", *ptr);

    return 0;
}
```

### 3.2. Ikkita asosiy operator

- **`&` (address-of)** — o'zgaruvchining manzilini oladi.
- **`*` (dereference)** — pointer ko'rsatib turgan manzildagi qiymatni oladi (yoki o'zgartiradi).

```c
int son = 10;
int *ptr = &son;

*ptr = 20;  // son ning qiymati endi 20 bo'ladi, chunki ptr son'ga ishora qiladi
printf("%d\n", son); // 20
```

**Diqqat:** `*` belgisi ikki xil ma'noda ishlatiladi:
1. Deklaratsiyada: `int *ptr;` — bu "ptr — int turiga pointer" degani.
2. Ishlatishda: `*ptr` — bu "ptr ko'rsatib turgan manzildagi qiymat" degani.

### 3.3. Pointer turlari mos kelishi kerak

```c
int son = 5;
double kasr = 3.14;

int *p1 = &son;     // to'g'ri
double *p2 = &kasr;  // to'g'ri
int *p3 = &kasr;     // XATO! (kompilyator ogohlantiradi yoki xato beradi)
```

Buning sababi — pointer nafaqat manzilni, balki **shu manzildan qancha bayt o'qish kerakligini** ham bilishi kerak. `int` odatda 4 bayt, `double` esa 8 bayt.

### 3.4. NULL pointer

Pointer hech narsaga ishora qilmasa, uni `NULL` bilan initsializatsiya qilish kerak:

```c
int *ptr = NULL; // hech qanday manzilga ishora qilmaydi

if (ptr != NULL) {
    printf("%d\n", *ptr);
} else {
    printf("ptr bo'sh (NULL)\n");
}
```

**Muhim qoida:** Hech qachon initsializatsiya qilinmagan pointer'ni ishlatmang! Bu — "wild pointer" (yovvoyi pointer) deb ataladi va segmentation fault'ga olib kelishi mumkin.

```c
int *ptr; // XAVFLI — qayerga ishora qilishi noma'lum
*ptr = 5; // KATTA XATO! Dastur qulab tushishi mumkin
```

---

<a name="4-pointer-arifmetikasi"></a>
## 4. Pointer arifmetikasi

Pointerlar ustida arifmetik amallar bajarish mumkin, lekin bu odatdagi son arifmetikasi emas — bu **tur o'lchamiga** bog'liq.

```c
int massiv[5] = {10, 20, 30, 40, 50};
int *ptr = massiv; // massiv nomi — birinchi elementga pointer

printf("%d\n", *ptr);       // 10
printf("%d\n", *(ptr + 1)); // 20 (ptr manzili + 1*sizeof(int))
printf("%d\n", *(ptr + 2)); // 30
```

`ptr + 1` — bu `ptr` manziliga 1 bayt emas, balki `sizeof(int)` (odatda 4 bayt) qo'shadi. Shu tufayli `ptr + 1` — massivning **keyingi elementiga** ishora qiladi.

### 4.1. Ruxsat etilgan amallar

| Amal | Misol | Izoh |
|---|---|---|
| Pointer + son | `ptr + 3` | Yangi manzilni beradi |
| Pointer - son | `ptr - 2` | Orqaga siljish |
| Pointer - Pointer | `ptr2 - ptr1` | Ikki manzil orasidagi elementlar soni |
| Pointer taqqoslash | `ptr1 < ptr2` | Manzillarni solishtirish |

**Ruxsat etilmagan:** ikkita pointer'ni qo'shish (`ptr1 + ptr2`) — bu mantiqsiz amal.

### 4.2. Increment va decrement

```c
int massiv[3] = {1, 2, 3};
int *p = massiv;

printf("%d\n", *p);   // 1
p++;                   // keyingi elementga o'tadi
printf("%d\n", *p);   // 2
p++;
printf("%d\n", *p);   // 3
```

---

<a name="5-pointer-massiv"></a>
## 5. Pointerlar va massivlar

C tilida massiv nomi — bu, aslida, massivning birinchi elementiga pointer sifatida "degradatsiya" qiladi (array decay).

```c
int massiv[4] = {1, 2, 3, 4};

printf("%p\n", (void*)massiv);     // birinchi element manzili
printf("%p\n", (void*)&massiv[0]); // xuddi shu manzil
```

### 5.1. Massiv elementlariga ikki xil murojaat

```c
int massiv[4] = {10, 20, 30, 40};
int *ptr = massiv;

// Bu ikkisi bir xil natija beradi:
printf("%d\n", massiv[2]);
printf("%d\n", *(ptr + 2));
printf("%d\n", *(massiv + 2));
printf("%d\n", ptr[2]);
```

### 5.2. Muhim farq: massiv va pointer bir xil emas

```c
int massiv[5];
int *ptr = massiv;

printf("%zu\n", sizeof(massiv)); // 5 * sizeof(int) = 20 (butun massiv hajmi)
printf("%zu\n", sizeof(ptr));    // 8 (pointer hajmi, 64-bitli tizimda)
```

Bu — ko'p dasturchilar tushunmaydigan nozik farq. Massiv o'zi haqiqiy xotira blokini bildiradi, pointer esa faqat manzilni saqlaydi.

### 5.3. Funksiyaga massiv uzatish

Funksiyaga massiv uzatilganda, u har doim pointer'ga aylanadi:

```c
void chopEt(int *massiv, int uzunlik) {
    for (int i = 0; i < uzunlik; i++) {
        printf("%d ", massiv[i]);
    }
}

int main() {
    int m[5] = {1, 2, 3, 4, 5};
    chopEt(m, 5); // massiv nomi pointer'ga aylanadi
    return 0;
}
```

Shu sababli funksiya ichida `sizeof(massiv)` massiv hajmini emas, pointer hajmini beradi — shuning uchun uzunlikni alohida parametr sifatida uzatish kerak.

### 5.4. String va pointerlar (C uslubi)

C da string — bu `'\0'` (null-terminator) bilan tugaydigan `char` massivi:

```c
char soz[] = "Salom";
char *ptr = soz;

while (*ptr != '\0') {
    printf("%c", *ptr);
    ptr++;
}
```

---

<a name="6-pointer-to-pointer"></a>
## 6. Pointer to Pointer (ko'rsatkichga ko'rsatkich)

Pointer ham xotirada joylashgan, demak uning ham manzili bor. Pointer manzilini saqlaydigan o'zgaruvchi — bu **pointer to pointer** (`**`).

```c
int son = 10;
int *ptr = &son;
int **ptrPtr = &ptr;

printf("%d\n", son);       // 10
printf("%d\n", *ptr);      // 10
printf("%d\n", **ptrPtr);  // 10
```

### 6.1. Qachon kerak bo'ladi?

Eng ko'p ishlatiladigan holat — funksiya ichida pointer'ning o'zini o'zgartirish kerak bo'lganda:

```c
void xotiraAjrat(int **ptr) {
    *ptr = malloc(sizeof(int));
    **ptr = 100;
}

int main() {
    int *p = NULL;
    xotiraAjrat(&p); // p ning manzilini uzatamiz
    printf("%d\n", *p); // 100
    free(p);
    return 0;
}
```

Agar biz `xotiraAjrat(int *ptr)` deb yozganimizda, funksiya ichida `ptr`ga yangi manzil bersak ham, bu o'zgarish faqat funksiya ichida qolib, `main` dagi `p` o'zgarmas edi — chunki C da hamma narsa **qiymat bo'yicha** (by value) uzatiladi.

---

<a name="7-function-pointer"></a>
## 7. Function Pointer (funksiyaga ko'rsatkich)

C/C++ da funksiyalarning ham xotirada manzili bor, va shu manzilni pointer orqali saqlash mumkin.

```c
#include <stdio.h>

int qoshish(int a, int b) { return a + b; }
int ayirish(int a, int b) { return a - b; }

int main() {
    int (*amal)(int, int); // int qabul qiluvchi, int qaytaruvchi funksiyaga pointer

    amal = qoshish;
    printf("%d\n", amal(5, 3)); // 8

    amal = ayirish;
    printf("%d\n", amal(5, 3)); // 2

    return 0;
}
```

### 7.1. Qayerda ishlatiladi?

- **Callback funksiyalar** (masalan, `qsort` funksiyasidagi taqqoslash funksiyasi).
- **State machine** (holatlar mashinasi) yaratishda.
- Plugin arxitekturalarida.

```c
#include <stdlib.h>

int solishtir(const void *a, const void *b) {
    return (*(int*)a - *(int*)b);
}

int main() {
    int massiv[] = {5, 2, 8, 1, 9};
    qsort(massiv, 5, sizeof(int), solishtir); // solishtir funksiyasi pointer sifatida uzatiladi
    return 0;
}
```

---

<a name="8-void-null"></a>
## 8. void*, NULL va xavfsizlik

### 8.1. void* — universal pointer

`void*` — bu tur haqida ma'lumot bermaydigan pointer. Har qanday turdagi manzilni saqlashi mumkin, lekin uni ishlatishdan oldin kerakli turga o'zgartirish (cast) kerak.

```c
void malumotChopEt(void *data, char tur) {
    if (tur == 'i') {
        printf("%d\n", *(int*)data);
    } else if (tur == 'f') {
        printf("%f\n", *(float*)data);
    }
}

int main() {
    int son = 10;
    float kasr = 3.14f;

    malumotChopEt(&son, 'i');
    malumotChopEt(&kasr, 'f');
    return 0;
}
```

`malloc` ham `void*` qaytaradi, shuning uchun uni istalgan turdagi pointer'ga tayinlash mumkin.

### 8.2. NULL va xavfsiz tekshiruv

```c
int *ptr = malloc(sizeof(int));
if (ptr == NULL) {
    printf("Xotira ajratilmadi!\n");
    return 1; // dasturni to'xtatish
}
*ptr = 5;
free(ptr);
```

**Qoida:** `malloc` (yoki `calloc`, `realloc`) natijasini har doim `NULL`ga tekshiring — ayniqsa katta xotira so'ralganda, tizimda joy yetishmasligi mumkin.

---

<a name="9-dinamik-xotira-c"></a>
## 9. C da dinamik xotira: malloc, calloc, realloc, free

`<stdlib.h>` kutubxonasi dinamik xotira bilan ishlash uchun 4 ta asosiy funksiyani beradi.

### 9.1. malloc — xotira ajratish

```c
int *ptr = malloc(5 * sizeof(int)); // 5 ta int uchun xotira
if (ptr == NULL) {
    // xatoni qayta ishlash
}
```

`malloc` xotirani ajratadi, lekin uni **initsializatsiya qilmaydi** — ichida "axlat" qiymatlar (garbage values) bo'lishi mumkin.

### 9.2. calloc — nol bilan to'ldirilgan xotira

```c
int *ptr = calloc(5, sizeof(int)); // 5 ta int, hammasi 0 bilan to'ldirilgan
```

`calloc(n, size)` — `malloc`dan farqli o'laroq, ajratilgan xotirani avtomatik ravishda 0 bilan to'ldiradi.

### 9.3. realloc — xotira hajmini o'zgartirish

```c
int *ptr = malloc(5 * sizeof(int));
// ... ishlatildi ...
int *yangiPtr = realloc(ptr, 10 * sizeof(int)); // endi 10 ta int uchun joy
if (yangiPtr == NULL) {
    // realloc muvaffaqiyatsiz bo'lsa, eski ptr hali ham amal qiladi!
    free(ptr);
} else {
    ptr = yangiPtr;
}
```

**Muhim:** `realloc` muvaffaqiyatsiz bo'lsa, `NULL` qaytaradi, lekin **eski pointer hali ham amal qiladi**. Shuning uchun to'g'ridan-to'g'ri `ptr = realloc(ptr, ...)` yozish xavfli — agar `realloc` `NULL` qaytarsa, eski manzilni yo'qotib qo'yasiz (memory leak).

### 9.4. free — xotirani bo'shatish

```c
int *ptr = malloc(sizeof(int) * 10);
// ... ishlatish ...
free(ptr);
ptr = NULL; // yaxshi amaliyot: dangling pointer'ning oldini olish
```

**Oltin qoida:** Har bir `malloc`/`calloc`/`realloc` uchun aynan bitta `free` bo'lishi kerak. Ko'p emas, kam ham emas.

### 9.5. To'liq misol: dinamik massiv

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int n;
    printf("Nechta son kiritasiz: ");
    scanf("%d", &n);

    int *massiv = malloc(n * sizeof(int));
    if (massiv == NULL) {
        printf("Xotira ajratib bo'lmadi!\n");
        return 1;
    }

    for (int i = 0; i < n; i++) {
        massiv[i] = i * i;
    }

    for (int i = 0; i < n; i++) {
        printf("%d ", massiv[i]);
    }
    printf("\n");

    free(massiv);
    massiv = NULL;

    return 0;
}
```

---

<a name="10-tipik-xatolar"></a>
## 10. Tipik xatolar: memory leak, dangling pointer, double free

### 10.1. Memory Leak (xotira sizib chiqishi)

Ajratilgan xotirani bo'shatishni unutib qo'yish:

```c
void funksiya() {
    int *ptr = malloc(sizeof(int) * 100);
    // free(ptr) yozilmagan!
} // funksiya tugadi, ptr yo'qoldi, lekin xotira hali ham band!
```

Bu xotira dastur ishlab turgan davomida "yo'qolgan" bo'lib qoladi — uni ishlatib ham, bo'shatib ham bo'lmaydi. Uzoq ishlaydigan dasturlarda (masalan, serverlar) bu asta-sekin butun tizim xotirasini yeb qo'yishi mumkin.

### 10.2. Dangling Pointer (osilib qolgan pointer)

Bo'shatilgan xotiraga hali ham ishora qilib turgan pointer:

```c
int *ptr = malloc(sizeof(int));
*ptr = 10;
free(ptr);

printf("%d\n", *ptr); // XAVFLI! ptr endi bo'shatilgan xotiraga ishora qiladi
```

**Yechim:** `free`dan keyin darhol `ptr = NULL` qo'ying.

### 10.3. Double Free (ikki marta bo'shatish)

```c
int *ptr = malloc(sizeof(int));
free(ptr);
free(ptr); // XATO! Xotira ikki marta bo'shatildi — dastur qulashi mumkin
```

### 10.4. Buffer Overflow (chegaradan chiqib ketish)

```c
int *massiv = malloc(5 * sizeof(int));
massiv[10] = 100; // XAVFLI! ajratilgan chegaradan tashqariga yozish
```

Bu — boshqa dasturlarning xotirasini buzishi va xavfsizlik zaifligiga (masalan, buffer overflow attack) sabab bo'lishi mumkin.

### 10.5. Xulosa jadvali

| Xato turi | Sabab | Oqibat |
|---|---|---|
| Memory leak | `free` chaqirilmagan | Xotira asta-sekin tugaydi |
| Dangling pointer | `free`dan keyin ishlatish | Aniqlanmagan xatti-harakat (UB) |
| Double free | Bir xotirani 2 marta `free` qilish | Dastur qulashi, xavfsizlik zaifligi |
| Buffer overflow | Chegaradan tashqariga yozish/o'qish | Ma'lumotlar buzilishi, xavfsizlik zaifligi |
| Wild pointer | Initsializatsiya qilinmagan pointer | Bashorat qilib bo'lmaydigan xatolar |

---

<a name="11-cpp-references"></a>
## 11. C++ da References (murojaatlar)

C++ pointerlardan tashqari yana bir tushunchani kiritadi — **reference (murojaat)**. Bu — allaqachon mavjud o'zgaruvchiga "taxallus" (alias) beradi.

```cpp
#include <iostream>
using namespace std;

int main() {
    int son = 10;
    int &ref = son; // ref — son ning boshqa nomi, xolos

    ref = 20;
    cout << son << endl; // 20

    return 0;
}
```

### 11.1. Reference va Pointer farqi

| Xususiyat | Pointer | Reference |
|---|---|---|
| NULL bo'lishi mumkinmi | Ha | Yo'q — har doim biror narsaga bog'langan bo'lishi kerak |
| Qayta bog'lash | Mumkin (boshqa manzilga ishora qilishi mumkin) | Mumkin emas — bir marta bog'langach o'zgarmaydi |
| Sintaksis | `*` va `&` kerak | Oddiy o'zgaruvchidek ishlatiladi |
| Arifmetika | Mumkin (`ptr++`) | Mumkin emas |
| Initsializatsiya | Keyinroq ham mumkin | Deklaratsiyada shart |

### 11.2. Funksiyalarda reference ishlatish

```cpp
void ikkilantirish(int &son) {
    son = son * 2;
}

int main() {
    int x = 5;
    ikkilantirish(x); // & belgisisiz, lekin x haqiqatda o'zgaradi
    cout << x << endl; // 10
    return 0;
}
```

Bu — C dagi pointer orqali "by reference" uzatishning tozaroq alternativasi:

```c
// C uslubi:
void ikkilantirish(int *son) {
    *son = *son * 2;
}
ikkilantirish(&x);
```

```cpp
// C++ uslubi:
void ikkilantirish(int &son) {
    son = son * 2;
}
ikkilantirish(x);
```

### 11.3. const reference — samaradorlik uchun

Katta obyektlarni funksiyaga nusxalamasdan uzatish uchun:

```cpp
void chopEt(const std::string &matn) {
    cout << matn << endl;
    // matn = "yangi"; // XATO — const bo'lgani uchun o'zgartirib bo'lmaydi
}
```

Bu — obyektni nusxalashdan (copy) qochib, ishlash tezligini oshiradi, shu bilan birga funksiya ichida uni tasodifan o'zgartirib qo'yishdan himoyalaydi.

---

<a name="12-cpp-new-delete"></a>
## 12. C++ da new va delete

C++ `malloc`/`free` o'rniga `new`/`delete` operatorlarini taklif etadi. Ular C funksiyalaridan farqli — **konstruktor/destruktorni avtomatik chaqiradi**.

### 12.1. Oddiy new/delete

```cpp
int *ptr = new int(42); // xotira ajratiladi VA 42 bilan initsializatsiya qilinadi
cout << *ptr << endl;
delete ptr;
ptr = nullptr; // C++11 dan beri NULL o'rniga nullptr tavsiya etiladi
```

### 12.2. Massiv uchun new[]/delete[]

```cpp
int *massiv = new int[10]; // 10 ta int uchun xotira
for (int i = 0; i < 10; i++) {
    massiv[i] = i;
}
delete[] massiv; // MUHIM: massiv uchun delete[] ishlatilishi shart!
```

**Diqqat:** Agar `new[]` bilan ajratilgan xotirani `delete` (qavssiz) bilan bo'shatsangiz — bu **aniqlanmagan xatti-harakat** (undefined behavior)ga olib keladi, chunki massivning har bir elementi uchun destruktor to'g'ri chaqirilmasligi mumkin.

### 12.3. Obyektlar bilan ishlash

```cpp
class Inson {
public:
    string ism;
    Inson(string i) : ism(i) {
        cout << ism << " yaratildi\n";
    }
    ~Inson() {
        cout << ism << " yo'q qilindi\n";
    }
};

int main() {
    Inson *p = new Inson("Zafar"); // konstruktor avtomatik chaqiriladi
    delete p; // destruktor avtomatik chaqiriladi
    return 0;
}
```

`malloc`/`free`da bunday narsa yo'q — ular faqat xom xotira bilan ishlaydi, konstruktor/destruktorni bilmaydi. Shuning uchun C++ da obyektlar uchun har doim `new`/`delete` ishlatish kerak, `malloc`/`free` emas.

### 12.4. new va malloc taqqoslashi

| Xususiyat | malloc/free (C) | new/delete (C++) |
|---|---|---|
| Konstruktor/destruktor | Chaqirmaydi | Avtomatik chaqiradi |
| Qaytariladigan tur | `void*` (cast kerak) | To'g'ri tur (cast shart emas) |
| Xotira yetmasa | `NULL` qaytaradi | `bad_alloc` exception tashlaydi |
| Operator/funksiya | Funksiya | Operator (overload qilish mumkin) |

---

<a name="13-raii"></a>
## 13. RAII tamoyili (Resource Acquisition Is Initialization)

RAII — zamonaviy C++ dasturlashning eng muhim tamoyillaridan biri. G'oyasi oddiy:

> **Resurs (xotira, fayl, tarmoq ulanishi) obyekt yaratilganda olinadi, va obyekt yo'q qilinganda (destruktor chaqirilganda) avtomatik qaytariladi.**

Bu — resursni qo'lda bo'shatishni unutib qo'yish xavfini butunlay yo'q qiladi, chunki C++ da lokal obyektning destruktori **doim** avtomatik chaqiriladi (hatto exception tashlansa ham).

### 13.1. RAII siz muammo

```cpp
void funksiya() {
    int *ptr = new int(5);
    
    if (/* qandaydir shart */ true) {
        throw std::runtime_error("Xato!");
        // delete ptr hech qachon chaqirilmaydi — MEMORY LEAK!
    }
    
    delete ptr;
}
```

### 13.2. RAII bilan yechim

```cpp
class ResursBoshqaruvchi {
    int *data;
public:
    ResursBoshqaruvchi() : data(new int(5)) {}
    ~ResursBoshqaruvchi() { delete data; } // avtomatik chaqiriladi!
};

void funksiya() {
    ResursBoshqaruvchi r; // konstruktor: xotira ajratiladi
    
    if (/* qandaydir shart */ true) {
        throw std::runtime_error("Xato!");
        // r ning destruktori baribir chaqiriladi, xotira bo'shatiladi!
    }
} // r doirasi tugaydi -> destruktor -> delete data avtomatik
```

Aynan shu tamoyil asosida C++ standart kutubxonasidagi `std::vector`, `std::string`, va — eng muhimi — **smart pointer**lar ishlaydi.

---

<a name="14-smart-pointers"></a>
## 14. Smart Pointerlar: unique_ptr, shared_ptr, weak_ptr

C++11 dan boshlab, `<memory>` kutubxonasi RAII tamoyiliga asoslangan **smart pointer**larni taqdim etadi. Ular xom pointerlarni o'rab oladi va xotirani avtomatik boshqaradi — `delete`ni qo'lda chaqirish shart emas!

### 14.1. unique_ptr — yagona egalik

`unique_ptr` — resursga faqat **bitta** egasi bo'lishini kafolatlaydi. Nusxalab bo'lmaydi, faqat "ko'chirish" (move) mumkin.

```cpp
#include <memory>
#include <iostream>
using namespace std;

int main() {
    unique_ptr<int> ptr = make_unique<int>(42);
    cout << *ptr << endl; // 42

    // unique_ptr<int> ptr2 = ptr; // XATO! Nusxalab bo'lmaydi
    unique_ptr<int> ptr2 = move(ptr); // OK — egalik ptr2 ga o'tadi
    
    if (!ptr) {
        cout << "ptr endi bo'sh\n"; // chunki egalik ptr2 ga o'tdi
    }

    return 0;
} // ptr2 doirasi tugaganda, xotira AVTOMATIK bo'shatiladi
```

**Obyektlar bilan:**

```cpp
class Mashina {
public:
    Mashina() { cout << "Mashina yaratildi\n"; }
    ~Mashina() { cout << "Mashina yo'q qilindi\n"; }
    void yur() { cout << "Mashina yuryapti\n"; }
};

int main() {
    unique_ptr<Mashina> m = make_unique<Mashina>();
    m->yur();
    // delete yozish shart emas! Doira tugaganda avtomatik yo'q qilinadi
}
```

### 14.2. shared_ptr — umumiy egalik

`shared_ptr` — resursga **bir nechta** egasi bo'lishiga ruxsat beradi. Ichida "reference counter" (havolalar sanog'i) saqlaydi — sanoq nolga tushgandagina xotira bo'shatiladi.

```cpp
#include <memory>
using namespace std;

int main() {
    shared_ptr<int> p1 = make_shared<int>(100);
    cout << p1.use_count() << endl; // 1

    {
        shared_ptr<int> p2 = p1; // egalik BO'LISHILADI, nusxalanmaydi
        cout << p1.use_count() << endl; // 2
    } // p2 doirasi tugadi, sanoq kamayadi

    cout << p1.use_count() << endl; // 1

    return 0;
} // p1 doirasi tugaganda, sanoq 0 ga tushadi, xotira bo'shatiladi
```

### 14.3. weak_ptr — kuzatuvchi pointer

`weak_ptr` — `shared_ptr` ga "egalik qilmasdan" kuzatib turadi. Bu — **circular reference** (aylanma bog'liqlik) muammosini hal qilish uchun ishlatiladi.

**Muammo — circular reference:**

```cpp
class Node {
public:
    shared_ptr<Node> next;
    ~Node() { cout << "Node yo'q qilindi\n"; }
};

int main() {
    shared_ptr<Node> a = make_shared<Node>();
    shared_ptr<Node> b = make_shared<Node>();
    a->next = b;
    b->next = a; // aylanma bog'liqlik! Ikkalasi ham hech qachon yo'q qilinmaydi (LEAK!)
}
```

**Yechim — weak_ptr:**

```cpp
class Node {
public:
    weak_ptr<Node> next; // endi egalik qilmaydi, faqat kuzatadi
    ~Node() { cout << "Node yo'q qilindi\n"; }
};

int main() {
    shared_ptr<Node> a = make_shared<Node>();
    shared_ptr<Node> b = make_shared<Node>();
    a->next = b;
    b->next = a; // endi muammo yo'q, chunki weak_ptr sanoqqa ta'sir qilmaydi
} // ikkalasi ham to'g'ri yo'q qilinadi
```

`weak_ptr`dan qiymatni olish uchun `lock()` metodidan foydalaniladi:

```cpp
weak_ptr<int> weak = make_shared<int>(10);
if (shared_ptr<int> sp = weak.lock()) {
    cout << *sp << endl; // hali mavjud bo'lsa
} else {
    cout << "Obyekt allaqachon yo'q qilingan\n";
}
```

### 14.4. Qaysi birini qachon ishlatish kerak?

| Smart Pointer | Qachon ishlatiladi |
|---|---|
| `unique_ptr` | Resursga faqat bitta egasi bo'lganda (95% holatlarda tavsiya etiladi) |
| `shared_ptr` | Resursga bir nechta joy bir vaqtda egalik qilishi kerak bo'lganda |
| `weak_ptr` | Aylanma bog'liqlikni oldini olish yoki "kuzatib turish, egalik qilmasdan" kerak bo'lganda |

**Zamonaviy C++ qoidasi:** Iloji boricha xom pointer (`new`/`delete`) o'rniga smart pointerlardan foydalaning. Xom pointerlarni faqat "egalik qilmaydigan, vaqtinchalik murojaat" sifatida ishlating (masalan, funksiya parametrida).

---

<a name="15-debug"></a>
## 15. Debug qilish vositalari

Xotira xatolarini qo'l bilan topish qiyin — shuning uchun maxsus vositalardan foydalanish tavsiya etiladi.

### 15.1. Valgrind (Linux/Mac)

Memory leak va noto'g'ri xotira ishlatilishini aniqlaydi:

```bash
gcc -g dastur.c -o dastur
valgrind --leak-check=full ./dastur
```

Natija sizga aynan qaysi qatorda xotira ajratilgan-yu, bo'shatilmagan ekanini ko'rsatadi.

### 15.2. AddressSanitizer (ASan)

GCC/Clang'ga o'rnatilgan, tezkor va qulay vosita:

```bash
gcc -fsanitize=address -g dastur.c -o dastur
./dastur
```

Bu — buffer overflow, use-after-free, double free kabi xatolarni darhol, tushunarli xabar bilan ko'rsatadi.

### 15.3. Statik analiz

- `cppcheck` — kodni ishga tushirmasdan tahlil qiladi.
- Zamonaviy IDElar (VS Code + C/C++ extension, CLion) ko'plab xatolarni yozish jarayonidayoq ko'rsatadi.

---

<a name="16-best-practices"></a>
## 16. Eng yaxshi amaliyotlar (Best Practices)

### C tilida:

1. Har doim `malloc` natijasini `NULL`ga tekshiring.
2. Har bir `malloc`/`calloc`/`realloc` uchun mos `free` yozing.
3. `free`dan keyin pointer'ni `NULL` qiling (dangling pointer'dan himoya).
4. `realloc`ning natijasini alohida o'zgaruvchiga yozing (`ptr = realloc(ptr, ...)` emas).
5. Massiv chegarasidan tashqariga chiqmang.
6. Initsializatsiya qilinmagan pointer'ni hech qachon ishlatmang.

### C++ da:

1. Xom `new`/`delete` o'rniga **smart pointerlar**dan foydalaning (`unique_ptr`, `shared_ptr`).
2. Katta obyektlarni funksiyaga uzatishda `const &` ishlating (nusxalashdan qoching).
3. RAII tamoyiliga rioya qiling — resurslarni har doim konstruktor/destruktor orqali boshqaring.
4. `NULL` o'rniga `nullptr` ishlating (C++11+).
5. Aylanma bog'liqlik (`shared_ptr` ↔ `shared_ptr`) bo'lsa, `weak_ptr` ishlating.
6. Iloji bo'lsa, dinamik xotiradan umuman qoching — `std::vector`, `std::string` kabi konteynerlar buni siz uchun xavfsiz bajaradi.

### Umumiy qoida (oltin qoida):

> **"Kim xotirani ajratsa, o'sha uni bo'shatishga javobgar bo'lsin — yoki bu javobgarlikni smart pointer/RAII obyektiga topshiring."**

---

<a name="17-mashqlar"></a>
## 17. Mashqlar

Bilimingizni mustahkamlash uchun quyidagi mashqlarni bajarib ko'ring:

1. **Swap funksiyasi:** Ikkita `int` o'zgaruvchining qiymatlarini pointer orqali almashtiruvchi `void swap(int *a, int *b)` funksiyasini yozing.

2. **Dinamik massiv:** Foydalanuvchidan `n` ta son kiritishni so'rab, `malloc` orqali dinamik massiv yaratuvchi va ularning yig'indisini hisoblovchi dastur yozing.

3. **String uzunligini hisoblash:** `strlen` funksiyasini ishlatmasdan, faqat pointer arifmetikasi orqali o'zingizning `mystrlen` funksiyangizni yozing.

4. **Bog'langan ro'yxat (Linked List):** C tilida oddiy bir yo'nalishli bog'langan ro'yxat (singly linked list) yarating: `qo'shish`, `o'chirish`, `chop etish` funksiyalari bilan. Bu — pointer to pointer'ni chuqur tushunish uchun ajoyib mashq.

5. **Matritsa (2D dinamik massiv):** `malloc` yordamida `n x m` o'lchamli dinamik ikki o'lchamli massiv yarating va uni to'g'ri bo'shating.

6. **unique_ptr bilan Stack:** C++ da `unique_ptr` yordamida oddiy `Stack` klassini (push, pop, top metodlari bilan) yozing — xotira avtomatik boshqarilishini ta'minlang.

7. **shared_ptr sanoq:** Ikkita `shared_ptr` bitta obyektga ishora qilganda, `use_count()` qanday o'zgarishini kuzatib, natijalarni chop eting.

8. **Xatoni toping:** Quyidagi kodda nechta xotira xatosi bor? Ularni toping va tuzating:
```c
int *funksiya() {
    int *ptr = malloc(sizeof(int) * 5);
    for (int i = 0; i < 10; i++) {
        ptr[i] = i;
    }
    return ptr;
}

int main() {
    int *p = funksiya();
    printf("%d\n", p[0]);
    // free yo'q...
    p = funksiya(); // eski p ga nima bo'ladi?
    free(p);
    free(p); // ...
    return 0;
}
```
*(Javob: 1) buffer overflow — 5 ta joy ajratilib 10 ta yozilmoqda; 2) birinchi `funksiya()` natijasi hech qachon bo'shatilmagan — memory leak; 3) `p = funksiya()` qayta chaqirilganda eski manzil yo'qoladi — yana leak; 4) `free(p)` ikki marta chaqirilgan — double free.)*

---

## Xulosa

Pointerlar va xotira boshqaruvi — C/C++ dasturlashning yuragi hisoblanadi. Ularni chuqur tushunish sizga:

- Dastur qanday ishlashini "past darajada" (low-level) tushunish imkonini beradi;
- Yuqori samaradorlikka ega dasturlar yozish qobiliyatini beradi;
- Boshqa tillarda (Python, Java, Go) "sahna ortida" nima sodir bo'layotganini tushunishga yordam beradi;
- Zamonaviy C++ da esa, smart pointerlar orqali bu murakkablikni xavfsiz va qulay tarzda boshqarish imkonini beradi.

**Eslab qoling:** xom pointerlar bilan ishlash katta kuch, ammo katta mas'uliyat ham talab qiladi. Har doim xotirani qanday ajratganingizni va qanday bo'shatishingiz kerakligini aniq bilib turing.

Omad tilaymiz! 🚀
