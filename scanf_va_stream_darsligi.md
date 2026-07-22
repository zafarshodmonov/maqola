# C tilida `scanf` va `<stdio.h>` Oqimlari (Streams): To'liq Darslik

## 1. Kirish: "Oqim" (Stream) nima?

C dasturlash tilida `<stdio.h>` kutubxonasi ma'lumotlarni kiritish/chiqarish (I/O) uchun **oqim (stream)** abstraksiyasidan foydalanadi. Oqim — bu dastur bilan tashqi dunyo (klaviatura, ekran, fayl) o'rtasidagi baytlar ketma-ketligining kanali, deb tasavvur qiling.

Har bir C dasturi ishga tushganda avtomatik ravishda 3 ta standart oqim ochiladi:

| Oqim | Ma'nosi | Odatiy manba/manzil |
|---|---|---|
| `stdin` | standard input | klaviatura |
| `stdout` | standard output | ekran (konsol) |
| `stderr` | standard error | ekran (xatoliklar uchun) |

`scanf` funksiyasi aynan **`stdin`** oqimidan ma'lumot o'qiydi. Aslida `scanf(...)` chaqiruvi ichki jihatdan `fscanf(stdin, ...)` ga ekvivalent.

Muhim tushuncha: oqim — bu **baytlar oqimi**, ya'ni C uchun klaviaturadan kelayotgan hamma narsa (raqamlar, harflar, bo'shliqlar, enter tugmasi) — bir xil turdagi belgilar ketma-ketligi. `scanf` bu ketma-ketlikni siz bergan format bo'yicha "parslaydi" (tahlil qiladi).

---

## 2. Bufer (Buffer) tushunchasi — eng muhim asos

`stdin` oqimi to'g'ridan-to'g'ri emas, balki **bufer** orqali ishlaydi.

Bufer — bu operatsion tizim tomonidan boshqariladigan vaqtinchalik xotira hududi. Siz klaviaturada nimadir yozganingizda, bu belgilar darhol dasturga uzatilmaydi — ular **Enter** tugmasi bosilgunga qadar shu buferda saqlanadi (bu — qatorli bufer, *line-buffered* rejim, terminal uchun standart).

```
Klaviatura → [Bufer: "1 2 3\n"] → Enter bosilganda → scanf shu buferni o'qiydi
```

Bu shuni anglatadiki:
- `scanf` chaqirilganda, agar buferda hali ma'lumot bo'lmasa, dastur **to'xtab turadi** (bloklanadi) va foydalanuvchi Enter bosishini kutadi.
- Agar buferda oldingi kiritishdan qolgan ma'lumot (masalan, `\n` belgisi) bo'lsa, `scanf` uni **darhol**, hech qanday kutishsiz o'qib oladi.

Aynan shu ikkinchi holat ko'plab boshlang'ich dasturchilarni chalg'itadigan mashhur xatolarga sabab bo'ladi (buni pastda §6 da batafsil ko'ramiz).

---

## 3. `scanf` funksiyasining sintaksisi va ishlash printsipi

```c
int scanf(const char *format, ...);
```

- **Birinchi argument** — format satri (masalan, `"%d"`, `"%s"`, `"%d %d"`).
- **Qolgan argumentlar** — o'zgaruvchilarning **manzillari** (`&` operatori bilan), chunki `scanf` qiymatlarni to'g'ridan-to'g'ri xotiraga yozadi (pointer orqali).

```c
int yosh;
scanf("%d", &yosh);   // &yosh — yosh o'zgaruvchisi joylashgan xotira manzili
```

Nima uchun `&` kerak? Chunki C da funksiyalar argumentlarni **qiymat bo'yicha** (by value) qabul qiladi. Agar biz `scanf("%d", yosh)` deb yozsak, `scanf` faqat `yosh` ning nusxasini oladi va asl o'zgaruvchini o'zgartira olmaydi. Manzilni (pointer) uzatish orqali `scanf` to'g'ridan-to'g'ri o'sha xotira katagiga borib yozadi.

### `scanf` qanday qaytariladigan qiymat beradi?

`scanf` — **muvaffaqiyatli o'qilgan va to'g'ri yozilgan elementlar sonini** (`int` turida) qaytaradi. Bu juda muhim, chunki u orqali kiritish xatolarini tekshirish mumkin:

```c
int a, b;
int natija = scanf("%d %d", &a, &b);

if (natija != 2) {
    printf("Xato: ikkita butun son kiritilishi kerak edi!\n");
}
```

Agar fayl oxiriga (EOF) yetib borilsa yoki o'qish boshlanishidanoq xato bo'lsa, `scanf` `EOF` (odatda `-1`) qaytaradi.

---

## 4. Format spetsifikatorlari (Format Specifiers)

`scanf` oqimni qanday "tahlil qilishi" kerakligini format satri orqali bilib oladi:

| Spetsifikator | Turi | Izoh |
|---|---|---|
| `%d` | `int` | Belgili o'nlik butun son |
| `%f` | `float` | Kasr son |
| `%lf` | `double` | Kasr son (double aniqlik) |
| `%c` | `char` | Bitta belgi (bo'shliqlarni ham o'qiydi!) |
| `%s` | `char[]` | Bo'shliqgacha bo'lgan satr |
| `%u` | `unsigned int` | Ishorasiz butun son |
| `%x` | `int` | O'n oltilik (hex) son |
| `%ld` | `long` | Uzun butun son |
| `%%` | — | Literal `%` belgisini kutadi |

### Misol: bir nechta qiymatni bir vaqtda o'qish

```c
int kun, oy, yil;
scanf("%d %d %d", &kun, &oy, &yil);
```

Bu yerda format satridagi bo'shliqlar muhim rol o'ynaydi — keyingi bo'limda buni ko'ramiz.

---

## 5. Format satridagi belgilarning ma'nosi

`scanf` format satrini uch xil "token" turi sifatida tushunadi:

### 5.1. Oddiy bo'shliq belgilari (space, `\t`, `\n`)

Format satrida **bitta bo'shliq** yozilishi — oqimdagi **noldan cheksizgacha** bo'lgan har qanday bo'shliq turkumidagi belgilarni (space, tab, newline) o'tkazib yuborishni bildiradi.

```c
scanf("%d %d", &a, &b);
```

Bu yerdagi `%d` orasidagi bo'shliq shuni bildiradiki: birinchi son o'qilgach, `scanf` ikkinchi sonni izlashdan oldin barcha bo'shliqlarni (shu jumladan enter tugmalarini ham) avtomatik o'tkazib yuboradi. Shuning uchun foydalanuvchi raqamlarni bir qatorga yoki alohida-alohida qatorlarga yozishi farqsiz ishlaydi.

### 5.2. Oddiy (bo'shliq bo'lmagan) belgilar

Agar format satrida `,` yoki `-` kabi oddiy belgi bo'lsa, `scanf` oqimda **aynan shu belgini** kutadi:

```c
int kun, oy, yil;
scanf("%d-%d-%d", &kun, &oy, &yil);
// Kiritish: 15-06-2026 ko'rinishida bo'lishi kerak
```

Agar oqimda kutilgan belgi topilmasa, `scanf` o'qishni o'sha yerda **to'xtatadi**.

### 5.3. Konversiya spetsifikatorlari (`%d`, `%s`, va h.k.)

Bular yuqorida ko'rib chiqilgan — muayyan turdagi ma'lumotni o'qib, mos o'zgaruvchiga yozadi.

**Muhim istisno: `%c`.** Boshqa spetsifikatorlardan farqli o'laroq, `%c` bo'shliqlarni **avtomatik o'tkazib yubormaydi** — u oqimdagi **keyingi istalgan belgini**, hatto u bo'shliq yoki `\n` bo'lsa ham, o'qib oladi. Bu §6 da ko'rsatiladigan muammoning asosiy manbai.

---

## 6. Eng mashhur xato: buferda qolib ketgan `\n`

Bu C o'rganuvchilarning aksariyati duch keladigan klassik muammo. Keling, sabab-oqibat zanjirini qadam-baqadam ko'ramiz.

```c
#include <stdio.h>

int main(void) {
    int son;
    char harf;

    printf("Sonni kiriting: ");
    scanf("%d", &son);          // Foydalanuvchi: 5[Enter]

    printf("Harfni kiriting: ");
    scanf("%c", &harf);         // Kutilmagan natija!

    printf("Son: %d, Harf: '%c'\n", son, harf);
    return 0;
}
```

**Nima sodir bo'ladi?**

1. Foydalanuvchi `5` yozadi va Enter bosadi. Buferda endi `"5\n"` saqlanadi.
2. `scanf("%d", &son)` — `5` raqamini o'qiydi, lekin **`\n` belgisini oqimda qoldiradi** (`%d` faqat raqamgacha bo'lgan qismni oladi, ortidagi bo'shliq/newline'ni "iste'mol qilmaydi", u faqat *oldidagi* bo'shliqlarni tashlab yuboradi).
3. Ikkinchi `scanf("%c", &harf)` chaqirilganda, bufer **bo'sh emas** — unda hali ham `\n` turibdi. `%c` esa bo'shliqlarni o'tkazib yubormasligini eslang — shuning uchun u foydalanuvchidan yangi kiritish so'ramay, o'sha eskirgan `\n` belgisini o'zlashtirib oladi!

Natijada dastur foydalanuvchidan harf kiritishni **kutmasdan** davom etadi, va `harf` o'zgaruvchisiga `\n` belgisi yoziladi.

### Yechim usullari

**1-usul: Format satrida oldindan bo'shliq qo'yish**

```c
scanf(" %c", &harf);   // %c dan oldingi bo'shlig'iga e'tibor bering
```

Bu bo'shliq oqimdagi barcha qoldiq bo'shliq belgilarini (jumladan `\n` ni) o'tkazib yuboradi, va `%c` haqiqiy, yangi kiritilgan belgini oladi.

**2-usul: Buferni qo'lda tozalash**

```c
scanf("%d", &son);
while (getchar() != '\n');   // Bufer '\n' gacha tozalanadi
scanf("%c", &harf);
```

**3-usul: `%s` ishlatish orqali muammoni chetlab o'tish**

`%s` — o'z-o'zidan bo'shliqlarni o'tkazib yuboradi, shu sababli satrlar bilan ishlashda bu muammo kamroq uchraydi, lekin printsip bir xil.

---

## 7. `%s` bilan ishlash: satr (string) o'qish xususiyatlari

```c
char ism[50];
scanf("%s", ism);   // E'tibor bering: & belgisi YO'Q!
```

Nima uchun `&` kerak emas? Chunki C da massiv nomi (`ism`) o'z-o'zidan massivning birinchi elementiga ko'rsatuvchi pointerga aylanadi (*array decay*). Demak, `ism` allaqachon manzil hisoblanadi.

**`%s` ning xususiyatlari:**
- U **birinchi bo'shliq belgisigacha** (space, tab yoki `\n`) bo'lgan hamma narsani o'qiydi.
- Oldindagi barcha bo'shliqlarni avtomatik o'tkazib yuboradi.
- O'qilgan satr oxiriga **avtomatik `'\0'` (null terminator)** qo'shadi.

```
Kiritish: "Ali Vali"
scanf("%s", ism);  →  ism = "Ali"   (faqat "Ali", bo'shliqdan keyingisi buferda qoladi!)
```

### ⚠️ Xavfsizlik ogohlantirishi: bufer to'lib ketishi (buffer overflow)

`scanf("%s", ...)` massiv hajmini **tekshirmaydi**! Agar foydalanuvchi 50 belgidan uzunroq satr kiritsa va massiv hajmi 50 bo'lsa, dastur xotiraning boshqa qismlariga yozib, **buffer overflow** xatosiga olib keladi — bu jiddiy xavfsizlik zaifligi hisoblanadi.

**Xavfsizroq yechim** — kenglikni cheklash:

```c
char ism[50];
scanf("%49s", ism);   // Maksimal 49 belgi + 1 ta '\0' uchun joy
```

Katta loyihalarda esa butun qatorni xavfsiz o'qish uchun `fgets(ism, sizeof(ism), stdin)` funksiyasidan foydalanish tavsiya etiladi, chunki u hajm chegarasini majburiy talab qiladi.

---

## 8. Nima uchun `scanf` ba'zan "abadiy" kutib qoladi (infinite loop)?

Quyidagi kodni ko'ramiz:

```c
int son;
while (scanf("%d", &son) != 1) {
    printf("Noto'g'ri kiritish, qaytadan urinib ko'ring: ");
}
```

Agar foydalanuvchi `abc` deb harflar kiritsa, muammo yuzaga keladi:

1. `scanf("%d", &son)` `a` belgisiga duch keladi — bu raqam emas, shuning uchun **o'qishni to'xtatadi** va `0` qaytaradi (muvaffaqiyatsiz).
2. Muhimi: `scanf` **oqimdagi `"abc\n"` ni o'chirmaydi** — u hamon buferda turaveradi!
3. Sikl yana `scanf` ni chaqiradi — u yana o'sha `a` belgisiga duch keladi — yana muvaffaqiyatsiz bo'ladi — va bu **cheksiz davom etadi**.

**To'g'ri yechim:** muvaffaqiyatsiz urinishdan so'ng buferni tozalash kerak:

```c
int son;
while (scanf("%d", &son) != 1) {
    printf("Noto'g'ri kiritish, qaytadan urinib ko'ring: ");
    while (getchar() != '\n');   // Noto'g'ri kiritilgan qatorni butunlay tozalash
}
```

---

## 9. `scanf` bilan bog'liq oqim holatlari: EOF va xato

`scanf` ikki turdagi "muammoli" holatga duch kelishi mumkin:

| Holat | Sabab | `scanf` xatti-harakati |
|---|---|---|
| **Matching failure** | Kutilgan formatga mos kelmaydigan belgi (masalan, `%d` kutilganda harf kelishi) | O'sha yerda to'xtaydi, mos kelgan elementlar sonini qaytaradi, muammoli belgi buferda qoladi |
| **Input failure (EOF)** | Oqim tugagan (fayl oxiri yoki `Ctrl+D` / `Ctrl+Z`) | `EOF` qaytaradi |

```c
int son;
while (scanf("%d", &son) == 1) {
    printf("Siz kiritdingiz: %d\n", son);
}
// Sikl EOF kelganda yoki noto'g'ri format kiritilganda tugaydi
```

Bu naqsh (*pattern*) — masalan, fayldan yoki `Ctrl+D` orqali "kiritishni tugatish" signali kelguncha, noma'lum sondagi elementlarni o'qishda juda foydali (masalan, musobaqabop dasturlashda — *competitive programming* — ko'p sonli kiritishlarni o'qishda keng qo'llaniladi).

---

## 10. Umumiy oqim modeli: vizual sxema

```
┌─────────────┐     Enter bosilganda     ┌──────────────────┐
│  Klaviatura │ ───────────────────────▶ │   stdin buferi    │
└─────────────┘                          │  "1 2 abc\n"       │
                                          └──────────────────┘
                                                   │
                                                   │ scanf("%d", &a)
                                                   ▼
                                          buferdan "1" olinadi,
                                          qolgani: " 2 abc\n"
                                                   │
                                                   │ scanf("%d", &b)
                                                   ▼
                                          bo'shliq o'tkaziladi,
                                          "2" olinadi,
                                          qolgani: " abc\n"
                                                   │
                                                   │ scanf("%d", &c)
                                                   ▼
                                          "a" raqam emas — TO'XTAYDI
                                          qolgani hamon: " abc\n"
```

Bu sxema shuni ko'rsatadiki: **bufer — bu umumiy, "sizib chiquvchi" resurs**. Bir `scanf` chaqiruvi butun buferni tozalab yubormaydi — u faqat o'ziga kerakli qismini "kesib oladi", qolganini keyingi chaqiruvlar uchun qoldiradi.

---

## 11. Xulosa: yodda tutish kerak bo'lgan asosiy qoidalar

1. `scanf` — `stdin` oqimidan, ya'ni bufer orqali o'qiydi; bufer Enter bosilgunga qadar to'ldiriladi.
2. `scanf` faqat o'ziga kerakli qismini oqimdan "iste'mol qiladi" — qolган belgilar (ayniqsa `\n`) buferda **qoladi** va keyingi chaqiruvga ta'sir qiladi.
3. Format satridagi bo'shliq — noldan cheksizgacha bo'lgan bo'shliq belgilarini o'tkazib yuborishni bildiradi; bundan `%c` mustasno — u hech narsani o'tkazib yubormaydi.
4. `scanf` doim o'qilgan elementlar sonini qaytaradi — bu qiymatni tekshirish orqali xatolarni aniqlash mumkin.
5. `%s` bilan ishlaganda massiv hajmini har doim cheklang (`%49s` kabi) — aks holda buffer overflow xavfi bor.
6. Noto'g'ri kiritish yuz berganda, buferni `while (getchar() != '\n');` yordamida tozalamasangiz, dastur cheksiz siklga tushib qolishi mumkin.

Ushbu tamoyillarni chuqur tushunish nafaqat `scanf` bilan bog'liq g'alati xatoliklarni bartaraf etishga, balki umuman oqimlar (`fgets`, `fread`, fayllar bilan ishlash) mantig'ini tushunishga ham asos bo'ladi.
