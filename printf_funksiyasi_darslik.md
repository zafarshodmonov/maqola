# `printf()` funksiyasi — to'liq va batafsil darslik

## Mundarija

1. `printf()` nima va u qanday ishlaydi
2. Prototip va header fayl
3. Format specifikatorlari (format specifiers) — to'liq jadval
4. Format specifikator anatomiyasi: flag, width, precision, length modifier
5. `printf()` ichida nima sodir bo'ladi (variadic funksiyalar mexanizmi)
6. Qaytish qiymati (return value)
7. `printf()` oilasi: `fprintf`, `sprintf`, `snprintf`, `vprintf` va h.k.
8. Keng tarqalgan xatolar va xavfsizlik muammolari (format string vulnerability)
9. Amaliy misollar
10. Xulosa va mashqlar

---

## 1. `printf()` nima va u qanday ishlaydi

`printf()` — C dasturlash tilidagi standart kutubxona (`stdio.h`) funksiyasi bo'lib, ma'lumotlarni **formatlangan holda** standart chiqishga (`stdout`, odatda terminal ekrani) chiqarish uchun ishlatiladi.

Nomi **"print formatted"** iborasidan qisqartirilgan.

Oddiy misol:

```c
#include <stdio.h>

int main(void) {
    printf("Salom, dunyo!\n");
    return 0;
}
```

Bu yerda `printf()` ikkita narsani bajaradi:
- Berilgan matnni ekranga chiqaradi
- Matn ichidagi maxsus belgilarni (masalan, `\n` — yangi qator) izohlaydi

Lekin `printf()`ning haqiqiy kuchi — u **formatlash** imkonini beradi, ya'ni turli xil ma'lumot turlarini (butun son, kasr son, satr, belgi va h.k.) oldindan belgilangan shablon (format string) asosida chiqarishi mumkin:

```c
int yosh = 25;
float bo_y = 1.75f;
printf("Yoshim: %d, bo'yim: %.2f metr\n", yosh, bo_y);
```

Natija:
```
Yoshim: 25, bo'yim: 1.75 metr
```

---

## 2. Prototip va header fayl

`printf()` funksiyasidan foydalanish uchun `<stdio.h>` header faylini qo'shish shart:

```c
#include <stdio.h>
```

Funksiyaning rasmiy prototipi (C standarti bo'yicha):

```c
int printf(const char *restrict format, ...);
```

Bu yerda:
- `const char *restrict format` — format string (shablon), qaysi ko'rinishda chiqarish kerakligini bildiradi. `restrict` kalit so'zi kompilyatorga bu ko'rsatkich boshqa hech qanday ko'rsatkich bilan bir xil xotira maydonini ko'rsatmasligini aytadi (optimallashtirish uchun).
- `...` — bu **variadic parametrlar** belgisi. Ya'ni `printf()` istalgan sondagi va istalgan turdagi qo'shimcha argumentlarni qabul qila oladi.
- Qaytish turi `int` — funksiya muvaffaqiyatli chiqarilgan belgilar sonini qaytaradi (bu haqda 6-bo'limda batafsil).

---

## 3. Format specifikatorlari — to'liq jadval

Format string ichida `%` belgisidan boshlanuvchi maxsus ketma-ketliklar **format specifikatorlari** (format specifiers) deb ataladi. Ular `printf()`ga qo'shimcha argumentni qanday turda va qanday ko'rinishda chiqarish kerakligini aytadi.

### 3.1. Asosiy specifikatorlar

| Specifikator | Ma'nosi | Argument turi |
|---|---|---|
| `%d` yoki `%i` | Ishorali o'nlik butun son (signed decimal integer) | `int` |
| `%u` | Ishorasiz o'nlik butun son (unsigned decimal integer) | `unsigned int` |
| `%f` | Kasr son, oddiy notatsiyada (fixed-point) | `double` |
| `%e` / `%E` | Kasr son, eksponensial (ilmiy) notatsiyada | `double` |
| `%g` / `%G` | `%f` yoki `%e` dan qisqarog'ini avtomatik tanlaydi | `double` |
| `%c` | Bitta belgi (character) | `int` (belgi sifatida talqin qilinadi) |
| `%s` | Satr (string), `\0` bilan tugaydigan belgilar massivi | `char *` |
| `%p` | Ko'rsatkich manzili (pointer address) | `void *` |
| `%x` / `%X` | O'n oltilik son (hexadecimal), kichik/katta harflar bilan | `unsigned int` |
| `%o` | Sakkizlik son (octal) | `unsigned int` |
| `%%` | Literal `%` belgisini chiqaradi | argument talab qilmaydi |
| `%n` | Hozirgacha chiqarilgan belgilar sonini ko'rsatilgan ko'rsatkichga yozadi | `int *` (**xavfli, ishlatish tavsiya etilmaydi**) |

### 3.2. Misollar

```c
#include <stdio.h>

int main(void) {
    int son = 42;
    unsigned int u_son = 4000000000u;
    double kasr = 3.14159265;
    char belgi = 'A';
    char *matn = "C dasturlash";
    void *ptr = &son;

    printf("Butun son: %d\n", son);
    printf("Ishorasiz son: %u\n", u_son);
    printf("Kasr son: %f\n", kasr);
    printf("Eksponensial: %e\n", kasr);
    printf("Belgi: %c\n", belgi);
    printf("Satr: %s\n", matn);
    printf("O'n oltilik: %x\n", son);
    printf("Sakkizlik: %o\n", son);
    printf("Ko'rsatkich manzili: %p\n", ptr);
    printf("Foiz belgisi: 100%%\n");

    return 0;
}
```

Natija (manzil qiymati har safar boshqacha bo'ladi):
```
Butun son: 42
Ishorasiz son: 4000000000
Kasr son: 3.141593
Eksponensial: 3.141593e+00
Belgi: A
Satr: C dasturlash
O'n oltilik: 2a
Sakkizlik: 52
Ko'rsatkich manzili: 0x7ffeeb1a2c4c
Foiz belgisi: 100%
```

**Muhim izoh:** `%f` standart holatda **6 ta raqamni** verguldan keyin ko'rsatadi, hatto son shuncha aniq bo'lmasa ham. Buni aniqlik (precision) yordamida boshqarish mumkin (4-bo'limda).

---

## 4. Format specifikator anatomiyasi

To'liq format specifikatori quyidagi umumiy ko'rinishga ega:

```
%[flags][width][.precision][length]specifier
```

Har bir qism ixtiyoriy (specifier'dan tashqari), lekin ular birgalikda chiqishni juda moslashuvchan qiladi.

### 4.1. Flag (bayroqlar)

| Flag | Ma'nosi |
|---|---|
| `-` | Chapga tekislash (default — o'ngga tekislanadi) |
| `+` | Musbat sonlar oldiga ham `+` belgisini qo'yadi |
| ` ` (bo'sh joy) | Musbat son oldiga bo'sh joy qo'yadi (ishora o'rniga) |
| `0` | Bo'sh joylarni `0` bilan to'ldiradi (padding) |
| `#` | Muqobil formatlash: `%#x` → `0x` prefiksi, `%#o` → `0` prefiksi qo'shadi |

```c
printf("[%5d]\n", 42);     // [   42]  — o'ngga tekislangan, kenglik 5
printf("[%-5d]\n", 42);    // [42   ]  — chapga tekislangan
printf("[%+d]\n", 42);     // [+42]    — ishora ko'rsatilgan
printf("[%05d]\n", 42);    // [00042]  — nol bilan to'ldirilgan
printf("[%#x]\n", 255);    // [0xff]   — prefiks bilan
```

### 4.2. Width (kenglik)

Minimal chiqish kengligini belgilaydi (belgilar sonida). Agar qiymat kenglikdan kichik bo'lsa, bo'sh joy (yoki `0` flag bilan nol) bilan to'ldiriladi.

```c
printf("[%10d]\n", 7);   // [         7]
printf("[%-10d]|\n", 7); // [7         ]|
```

Kenglikni dinamik tarzda ham berish mumkin — `*` belgisi orqali, qiymat argumentlar ro'yxatidan olinadi:

```c
int kenglik = 8;
printf("[%*d]\n", kenglik, 42);  // [      42]
```

### 4.3. Precision (aniqlik)

Nuqta (`.`) bilan boshlanadi va turga qarab har xil ma'noga ega:

- **Kasr sonlar uchun (`%f`, `%e`)**: verguldan keyingi raqamlar soni
- **Satrlar uchun (`%s`)**: chiqariladigan maksimal belgilar soni
- **Butun sonlar uchun (`%d`)**: minimal raqamlar soni (kerak bo'lsa oldiga nollar qo'shiladi)

```c
printf("%.2f\n", 3.14159);     // 3.14
printf("%.10f\n", 3.14159);    // 3.1415900000
printf("%.3s\n", "Salom");     // Sal   (faqat 3 ta belgi)
printf("%.5d\n", 42);          // 00042
```

Precision ham `*` orqali dinamik berilishi mumkin:

```c
printf("%.*f\n", 2, 3.14159);  // 3.14
```

### 4.4. Length modifier (uzunlik modifikatori)

Argumentning haqiqiy xotira o'lchamini bildiradi:

| Modifikator | `%d` bilan | `%u`/`%x` bilan | `%f` bilan |
|---|---|---|---|
| `h` | `short int` | `unsigned short` | — |
| `hh` | `signed char` | `unsigned char` | — |
| `l` | `long int` | `unsigned long` | — |
| `ll` | `long long int` | `unsigned long long` | — |
| `L` | — | — | `long double` |
| `z` | `size_t` (`%zu` bilan ko'p ishlatiladi) | | |

```c
long uzun_son = 123456789012L;
long long juda_uzun = 123456789012345LL;
size_t hajm = sizeof(int);

printf("%ld\n", uzun_son);
printf("%lld\n", juda_uzun);
printf("%zu\n", hajm);
```

**Diqqat:** `%f` uchun `double` argument avtomatik ravishda variadic funksiyalarda `float`dan `double`ga ko'tariladi (default argument promotion), shuning uchun `%f` ham `float`, ham `double` uchun ishlatiladi — `l` modifikatori shart emas.

---

## 5. `printf()` ichida nima sodir bo'ladi (variadic mexanizmi)

`printf()` qanday qilib istalgan sondagi argumentni qabul qilishini tushunish uchun **variadic funksiyalar** mexanizmini bilish kerak.

C tilida bunday funksiyalar `<stdarg.h>` header fayli orqali yaratiladi. `printf()`ning ichki (soddalashtirilgan) mantig'i taxminan quyidagicha ko'rinadi:

```c
#include <stdarg.h>
#include <stdio.h>

void mini_printf(const char *format, ...) {
    va_list args;
    va_start(args, format);   // 'format'dan keyingi argumentlarga o'tish

    for (const char *p = format; *p != '\0'; p++) {
        if (*p == '%' && *(p + 1) == 'd') {
            int qiymat = va_arg(args, int);  // navbatdagi argumentni int sifatida olish
            printf("%d", qiymat);
            p++;
        } else if (*p == '%' && *(p + 1) == 's') {
            char *qiymat = va_arg(args, char *);
            printf("%s", qiymat);
            p++;
        } else {
            putchar(*p);
        }
    }

    va_end(args);  // tozalash
}
```

### Muhim tushuncha: `printf()` argument sonini bilmaydi

C tilida funksiyalar o'ziga necha ta argument kelganini avtomatik bilmaydi. `printf()` **faqat format string ichidagi `%` belgilarini sanab**, xotiradan shuncha argumentni "o'qib chiqadi" (stack yoki register orqali, kompilyator va arxitekturaga qarab).

Bu degani — agar siz format string bilan haqiqiy argumentlar sonini **mos kelmasa**, dastur **noaniq xatti-harakatga (undefined behavior)** duch keladi:

```c
printf("%d %d\n", 5);  // XATO! Ikkinchi %d uchun argument yo'q
```

Bu holatda `printf()` xotiraning tasodifiy joyidan qiymat o'qib, uni son sifatida chiqaradi — bu **xavfsizlik zaifligi** hisoblanadi (9-bo'limda batafsil).

---

## 6. Qaytish qiymati (return value)

`printf()` — `int` turini qaytaradi:

- **Muvaffaqiyatli bo'lsa**: chiqarilgan belgilar sonini qaytaradi (yangi qator belgisi `\n` ham hisobga olinadi)
- **Xato yuz bersa**: manfiy qiymat (odatda `-1`) qaytaradi

```c
int soni = printf("Salom\n");
printf("Chiqarilgan belgilar soni: %d\n", soni);
```

Natija:
```
Salom
Chiqarilgan belgilar soni: 6
```

(`"Salom\n"` — 5 ta harf + 1 ta `\n` = 6 belgi)

Bu qaytish qiymati kamdan-kam ishlatiladi, lekin ba'zan xatoni tekshirish yoki chiqarilgan belgilar sonini hisoblash uchun foydali bo'ladi.

---

## 7. `printf()` oilasi

`printf()`dan tashqari standart kutubxonada bir nechta o'xshash funksiyalar mavjud:

| Funksiya | Vazifasi |
|---|---|
| `fprintf(FILE *stream, ...)` | Fayl yoki boshqa oqimga (masalan, `stderr`) yozadi |
| `sprintf(char *str, ...)` | Natijani `stdout`ga emas, balki xotiradagi satrga yozadi |
| `snprintf(char *str, size_t n, ...)` | `sprintf`ning xavfsiz varianti — maksimal `n` belgigacha yozadi (buffer overflow'dan himoyalaydi) |
| `vprintf`, `vfprintf`, `vsprintf`, `vsnprintf` | Yuqoridagilarning `va_list` argument qabul qiluvchi versiyalari |

### Misollar

```c
#include <stdio.h>

int main(void) {
    // fprintf — xato xabarlarini stderr'ga yozish odatiy amaliyot
    fprintf(stderr, "Xatolik: fayl topilmadi\n");

    // sprintf — natijani satrga yozish
    char buffer[50];
    sprintf(buffer, "Yig'indi: %d", 15);
    printf("%s\n", buffer);

    // snprintf — xavfsiz, hajm chegaralangan
    char kichik_buffer[10];
    int yozildi = snprintf(kichik_buffer, sizeof(kichik_buffer), "%d + %d = %d", 100, 200, 300);
    printf("Buferga yozilgan: %s\n", kichik_buffer);
    printf("Agar bufer cheksiz bo'lganda yozilishi kerak bo'lgan uzunlik: %d\n", yozildi);

    return 0;
}
```

**Nima uchun `sprintf()` o'rniga `snprintf()` ishlatish tavsiya etiladi?**

`sprintf()` bufer hajmini bilmaydi — agar natija buferdan katta bo'lsa, u xotiraning boshqa qismlariga "toshib ketadi" (**buffer overflow**), bu esa dastur ishdan chiqishiga yoki xavfsizlik zaifligiga olib kelishi mumkin. `snprintf()` esa ikkinchi argument (`n`) orqali maksimal yozish hajmini cheklaydi.

---

## 8. Keng tarqalgan xatolar va xavfsizlik muammolari

### 8.1. Format specifikator va argument turi mos kelmasligi

```c
int son = 5;
printf("%s\n", son);  // XATO! %s char* kutadi, int emas
```

Bu **undefined behavior** — dastur ishdan chiqishi yoki noto'g'ri ma'lumot chiqarishi mumkin. Zamonaviy kompilyatorlar (`gcc -Wall`, `-Wformat`) bunday xatolarni ogohlantiradi.

### 8.2. Format string vulnerability (foydalanuvchi kiritgan matnni to'g'ridan-to'g'ri format sifatida ishlatish)

```c
char foydalanuvchi_matni[100];
fgets(foydalanuvchi_matni, sizeof(foydalanuvchi_matni), stdin);

printf(foydalanuvchi_matni);  // JUDA XAVFLI!
```

Agar foydalanuvchi `"%x %x %x %n"` kabi matn kiritsa, dastur xotiradan o'qiydi yoki hatto **`%n` orqali xotiraga yozadi** — bu klassik xavfsizlik zaifligi (**format string attack**) bo'lib, ko'plab haqiqiy hujumlarda ishlatilgan.

**To'g'ri yechim:**
```c
printf("%s", foydalanuvchi_matni);  // Har doim format sifatida %s ishlating
```

### 8.3. `%f` bilan `float` va `double`ni chalkashtirish

```c
float f = 3.14f;
printf("%f\n", f);  // Ishlaydi — variadic funksiyalarda float avtomatik double'ga ko'tariladi
```

Bu haqiqatda xato emas, lekin dasturchilar buni tushunmasdan chalkashib ketishadi.

### 8.4. Yangi qatorni unutish yoki bufer tozalanmasligi

```c
printf("Natija: ");  // \n yo'q, ba'zi sharoitlarda bufer darhol chiqarilmasligi mumkin
```

Standart chiqish (`stdout`) odatda **satr bo'yicha buferlanadi** (line-buffered) terminalga chiqishda. `\n` bo'lmasa, chiqish kechikishi mumkin. Kerak bo'lsa `fflush(stdout);` bilan majburiy tozalash mumkin.

---

## 9. Amaliy misollar

### 9.1. Jadval ko'rinishida chiqarish

```c
#include <stdio.h>

int main(void) {
    printf("%-15s%10s\n", "Ism", "Yosh");
    printf("%-15s%10d\n", "Aziz", 23);
    printf("%-15s%10d\n", "Malika", 21);
    return 0;
}
```

Natija:
```
Ism                  Yosh
Aziz                   23
Malika                 21
```

### 9.2. Turli sanoq sistemalarida chiqarish

```c
#include <stdio.h>

int main(void) {
    int son = 255;
    printf("O'nlik: %d\n", son);
    printf("O'n oltilik: %#x\n", son);
    printf("Sakkizlik: %#o\n", son);
    return 0;
}
```

Natija:
```
O'nlik: 255
O'n oltilik: 0xff
Sakkizlik: 0377
```

### 9.3. Pul miqdorini formatlash

```c
#include <stdio.h>

int main(void) {
    double narx = 1234567.891;
    printf("Narx: $%.2f\n", narx);
    return 0;
}
```

Natija:
```
Narx: $1234567.89
```

---

## 10. Xulosa va mashqlar

### Asosiy tushunchalar xulosasi

- `printf()` — formatlangan chiqish uchun `<stdio.h>`dagi variadic funksiya
- Format string ichidagi `%` belgisi orqali specifikatorlar orqali qiymatlar joylashtiriladi
- To'liq format: `%[flags][width][.precision][length]specifier`
- Argument turi va format specifikatori **aniq mos kelishi shart** — aks holda undefined behavior
- Foydalanuvchi kiritgan matnni **hech qachon** to'g'ridan-to'g'ri format string sifatida ishlatmang
- `snprintf()` — xavfsizroq muqobil, bufer hajmini nazorat qiladi

### Mashqlar

1. Foydalanuvchidan ism va yoshni o'qib, ularni jadval ko'rinishida (`%-15s%5d`) chiqaruvchi dastur yozing.
2. Berilgan `double` sonni 3 xil aniqlikda (`.0f`, `.2f`, `.5f`) chiqarib, natijalarni solishtiring.
3. `snprintf()` yordamida 10 belgidan katta bo'lmagan buferga xavfsiz yozuv qiluvchi funksiya yozing va agar matn kesilgan bo'lsa, buni aniqlang (`snprintf` qaytargan qiymatni tekshirish orqali).
4. `%x`, `%o`, `%d` specifikatorlaridan foydalanib, 1 dan 20 gacha bo'lgan sonlarni uch xil sanoq sistemasida chiqaruvchi jadval yarating.
