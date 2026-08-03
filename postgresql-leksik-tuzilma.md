# 4.1. Leksik tuzilma (Lexical Structure)

SQL kirish ma'lumotlari buyruqlar (commands) ketma-ketligidan iborat. Har bir buyruq bir qator token (token)lardan tashkil topgan bo'lib, oxirida nuqta-vergul (;) qo'yiladi. Kirish oqimining tugashi ham buyruqni tugatadi. Qaysi tokenlar to'g'ri hisoblanishi muayyan buyruqning sintaksisiga bog'liq.

Token — bu kalit so'z (key word), identifikator (identifier), qo'shtirnoqqa olingan identifikator (quoted identifier), literal (yoki konstanta) yoki maxsus belgi bo'lishi mumkin. Tokenlar odatda bo'sh joy (probel, tab, yangi qator) bilan ajratiladi, lekin noaniqlik bo'lmasa, ajratish shart emas (bu odatda maxsus belgi boshqa token turiga tutash bo'lgandagina yuz beradi).

Masalan, quyidagi SQL kiritmasi (sintaktik jihatdan) to'g'ri hisoblanadi:

```sql
SELECT * FROM MY_TABLE;
UPDATE MY_TABLE SET A = 5;
INSERT INTO MY_TABLE VALUES (3, 'hi there');
```

Bu — har biri bir qatordan iborat uchta buyruqning ketma-ketligi (garchi bu shart emas; bir qatorda bir nechta buyruq bo'lishi mumkin, buyruqlar esa bir necha qatorga bo'linishi ham mumkin).

Bundan tashqari, SQL kiritmasida izohlar (comments) ham uchrashi mumkin. Ular token hisoblanmaydi va aslida bo'sh joyga teng deb qaraladi.

SQL sintaksisi qaysi tokenlar buyruqni, qaysilari operand yoki parametrlarni bildirishi borasida unchalik izchil emas. Odatda dastlabki bir necha token buyruq nomini bildiradi, shu sababli yuqoridagi misolda biz "SELECT", "UPDATE" va "INSERT" buyruqlari haqida gapiramiz. Ammo, masalan, UPDATE buyrug'ida SET tokeni har doim ma'lum bir o'rinda kelishi shart, va INSERT buyrug'ining ushbu ko'rinishida to'liq bo'lishi uchun VALUES so'zi ham talab qilinadi. Har bir buyruq uchun aniq sintaksis qoidalari VI qismda tavsiflangan.

## 4.1.1. Identifikatorlar va kalit so'zlar (Identifiers and Key Words)

Yuqoridagi misoldagi SELECT, UPDATE yoki VALUES kabi tokenlar kalit so'zlarga (key words) misol bo'lib, ular SQL tilida qat'iy ma'noga ega so'zlardir. MY_TABLE va A tokenlari esa identifikatorlarga (identifiers) misoldir. Ular ishlatilayotgan buyruqqa qarab jadval, ustun yoki boshqa ma'lumotlar bazasi obyektlarining nomlarini bildiradi. Shu sababli ularni ba'zan shunchaki "nomlar" deb ham atashadi. Kalit so'zlar va identifikatorlar bir xil leksik tuzilishga ega, ya'ni tilni bilmasdan turib biror tokenning identifikator yoki kalit so'z ekanini aniqlab bo'lmaydi. Kalit so'zlarning to'liq ro'yxatini C ilovasidan (Appendix C) topish mumkin.

SQL identifikatorlari va kalit so'zlari harf (a-z, shuningdek diakritik belgili va lotin bo'lmagan harflar) yoki pastki chiziq (_) bilan boshlanishi kerak. Identifikator yoki kalit so'zdagi keyingi belgilar harflar, pastki chiziqlar, raqamlar (0-9) yoki dollar belgilari ($) bo'lishi mumkin. E'tibor bering, SQL standarti bo'yicha dollar belgisiga identifikatorlarda ruxsat berilmaydi, shuning uchun ulardan foydalanish ilovalarning portativligini pasaytirishi mumkin. SQL standarti raqam bilan boshlanadigan yoki tugaydigan yoxud pastki chiziq bilan boshlanadigan/tugaydigan kalit so'zni belgilamaydi, shu sababli shunday shakldagi identifikatorlar standartning kelajakdagi kengaytmalari bilan mumkin bo'lgan ziddiyatdan xoli hisoblanadi.

Tizim identifikator uchun ko'pi bilan NAMEDATALEN-1 baytdan foydalanadi; uzunroq nomlarni buyruqlarda yozish mumkin, lekin ular qisqartiriladi. Standart bo'yicha NAMEDATALEN 64 ga teng, shuning uchun identifikatorning maksimal uzunligi 63 bayt. Agar bu chegara muammo tug'dirsa, uni src/include/pg_config_manual.h faylidagi NAMEDATALEN konstantasini o'zgartirish orqali oshirish mumkin.

Kalit so'zlar va qo'shtirnoqsiz identifikatorlar registrga (case) sezgir emas. Shu sababli:

```sql
UPDATE MY_TABLE SET A = 5;
```

quyidagicha ham yozilishi mumkin:

```sql
uPDaTE my_TabLE SeT a = 5;
```

Ko'pincha qo'llaniladigan konventsiya — kalit so'zlarni katta harf bilan, nomlarni esa kichik harf bilan yozish, masalan:

```sql
UPDATE my_table SET a = 5;
```

Identifikatorning ikkinchi turi mavjud: chegaralangan identifikator (delimited identifier) yoki qo'shtirnoqqa olingan identifikator (quoted identifier). U ixtiyoriy belgilar ketma-ketligini qo'sh qo'shtirnoq (") ichiga olish orqali hosil qilinadi. Chegaralangan identifikator har doim identifikator hisoblanadi, hech qachon kalit so'z emas. Shuning uchun "select" so'zini "select" nomli ustun yoki jadvalga murojaat qilish uchun ishlatish mumkin, qo'shtirnoqsiz select esa kalit so'z sifatida qabul qilinadi va jadval yoki ustun nomi talab qilingan joyda ishlatilsa, sintaksis xatosini keltirib chiqaradi. Misolni qo'shtirnoqqa olingan identifikatorlar bilan quyidagicha yozish mumkin:

```sql
UPDATE "my_table" SET "a" = 5;
```

Qo'shtirnoqqa olingan identifikatorlar kod raqami noldan boshqa istalgan belgini o'z ichiga olishi mumkin. (Qo'sh qo'shtirnoqni kiritish uchun ikkita qo'sh qo'shtirnoqni yozing.) Bu bo'shliq yoki ampersand kabi belgilarni o'z ichiga olgan, boshqa yo'l bilan yaratib bo'lmaydigan jadval yoki ustun nomlarini yaratishga imkon beradi. Uzunlik cheklovi baribir amal qiladi.

Identifikatorni qo'shtirnoqqa olish uni registrga sezgir qiladi, qo'shtirnoqsiz nomlar esa doim kichik harfga aylantiriladi. Masalan, FOO, foo va "foo" identifikatorlari PostgreSQL tomonidan bir xil deb hisoblanadi, ammo "Foo" va "FOO" bu uchtasidan va bir-biridan farq qiladi. (PostgreSQL'da qo'shtirnoqsiz nomlarni kichik harfga aylantirish SQL standartiga mos kelmaydi, chunki standartga ko'ra qo'shtirnoqsiz nomlar katta harfga aylantirilishi kerak. Ya'ni standart bo'yicha foo — "foo" emas, "FOO" ga teng bo'lishi kerak. Portativ ilovalar yozmoqchi bo'lsangiz, ma'lum bir nomni har doim qo'shtirnoqqa olish yoki hech qachon olmaslik tavsiya etiladi.)

Qo'shtirnoqqa olingan identifikatorlarning bir varianti kod nuqtalari (code points) orqali belgilangan qochirilgan Unicode belgilarini kiritishga imkon beradi. Bu variant U& (katta yoki kichik U harfi, undan keyin ampersand) bilan boshlanadi va u ochiluvchi qo'sh qo'shtirnoqdan darhol oldin, orasida bo'shliqsiz yoziladi, masalan U&"foo". (Bu & operatori bilan noaniqlik yaratishini unutmang. Bu muammoning oldini olish uchun operator atrofida bo'shliq qoldiring.) Qo'shtirnoqlar ichida Unicode belgilarini teskari chiziq (backslash) va to'rt xonali o'n oltilik (hexadecimal) kod raqami yoki teskari chiziq, undan keyin plyus belgisi va olti xonali o'n oltilik kod raqami orqali qochirilgan shaklda ko'rsatish mumkin. Masalan, "data" identifikatorini quyidagicha yozish mumkin:

```
U&"d\0061t\+000061"
```

Quyidagi ancha murakkabroq misolda rus tilidagi "slon" (fil) so'zi kiril harflarida yozilgan:

```
U&"\0441\043B\043E\043D"
```

Agar teskari chiziqdan boshqa qochirish belgisi (escape character) kerak bo'lsa, uni satrdan keyin UESCAPE bandi (clause) yordamida belgilash mumkin, masalan:

```
U&"d!0061t!+000061" UESCAPE '!'
```

Qochirish belgisi sifatida o'n oltilik raqam, plyus belgisi, bitta qo'shtirnoq, qo'sh qo'shtirnoq yoki bo'shliq belgisidan boshqa istalgan bitta belgini ishlatish mumkin. E'tibor bering, qochirish belgisi UESCAPE'dan keyin qo'sh qo'shtirnoqda emas, bitta qo'shtirnoqda yoziladi.

Qochirish belgisini identifikator ichida so'zma-so'z kiritish uchun uni ikki marta yozing.

UTF-16 surrogat juftliklarini (surrogate pairs) belgilash uchun 4 xonali yoki 6 xonali qochirish shaklidan foydalanish mumkin, bu esa U+FFFF dan katta kod nuqtali belgilarni hosil qiladi, garchi 6 xonali shaklning mavjudligi texnik jihatdan buni keraksiz qilsa ham. (Surrogat juftliklar to'g'ridan-to'g'ri saqlanmaydi, balki bitta kod nuqtasiga birlashtiriladi.)

Agar server kodlashi (server encoding) UTF-8 bo'lmasa, ushbu qochirish ketma-ketliklaridan biri bilan belgilangan Unicode kod nuqtasi haqiqiy server kodlashiga aylantiriladi; agar bu imkonsiz bo'lsa, xato haqida xabar beriladi.

## 4.1.2. Konstantalar (Constants)

PostgreSQL'da uchta oshkora tipdagi (implicitly-typed) konstanta turi mavjud: satrlar (strings), bit satrlar (bit strings) va sonlar (numbers). Konstantalarni aniq tiplar bilan ham belgilash mumkin, bu esa tizim tomonidan ko'proq aniq ifodalash va samaraliroq ishlov berishga imkon beradi. Bu alternativalar quyidagi bo'limlarda ko'rib chiqiladi.

### 4.1.2.1. Satr konstantalari (String Constants)

SQL'da satr konstantasi bitta qo'shtirnoq (') bilan chegaralangan ixtiyoriy belgilar ketma-ketligidir, masalan 'This is a string'. Satr konstantasi ichida bitta qo'shtirnoq belgisini kiritish uchun ikkita ketma-ket bitta qo'shtirnoq yoziladi, masalan, 'Dianne''s horse'. Bu qo'sh qo'shtirnoq belgisi (") bilan bir xil emasligiga e'tibor bering.

Kamida bitta yangi qator bilan ajratilgan, faqat bo'shliq bilan ajratilgan ikkita satr konstantasi birlashtiriladi va xuddi bitta konstanta sifatida yozilgandek qaraladi. Masalan:

```sql
SELECT 'foo'
'bar';
```

quyidagiga teng:

```sql
SELECT 'foobar';
```

lekin:

```sql
SELECT 'foo'      'bar';
```

to'g'ri sintaksis emas. (Bu biroz g'alati xatti-harakat SQL tomonidan belgilangan; PostgreSQL standartga amal qiladi.)

### 4.1.2.2. C uslubidagi qochirishlarga ega satr konstantalari (String Constants with C-Style Escapes)

PostgreSQL, shuningdek, SQL standartiga kengaytma bo'lgan "qochirilgan" (escape) satr konstantalarini ham qabul qiladi. Qochirilgan satr konstantasi ochiluvchi bitta qo'shtirnoqdan oldin E (katta yoki kichik) harfini yozish orqali belgilanadi, masalan, E'foo'. (Qochirilgan satr konstantasi bir necha qatorga davom etganda, E faqat birinchi ochiluvchi qo'shtirnoqdan oldin yoziladi.) Qochirilgan satr ichida teskari chiziq belgisi (\) C uslubidagi teskari chiziqli qochirish ketma-ketligini (backslash escape sequence) boshlaydi, unda teskari chiziq va undan keyingi belgi(lar) birikmasi 4.1-jadvalda ko'rsatilgandek maxsus bayt qiymatini bildiradi.

**4.1-jadval. Teskari chiziqli qochirish ketma-ketliklari (Backslash Escape Sequences)**

| Teskari chiziqli ketma-ketlik | Ma'nosi |
|---|---|
| \b | orqaga siljish (backspace) |
| \f | forma uzatish (form feed) |
| \n | yangi qator (newline) |
| \r | qatorni boshiga qaytarish (carriage return) |
| \t | tab |
| \o, \oo, \ooo (o = 0–7) | sakkizlik (octal) bayt qiymati |
| \xh, \xhh (h = 0–9, A–F) | o'n oltilik (hexadecimal) bayt qiymati |
| \uxxxx, \Uxxxxxxxx (x = 0–9, A–F) | 16 yoki 32-bitli o'n oltilik Unicode belgi qiymati |

Teskari chiziqdan keyin keladigan boshqa har qanday belgi so'zma-so'z qabul qilinadi. Shu sababli, teskari chiziq belgisini kiritish uchun ikkita teskari chiziq yozing (\\). Shuningdek, qochirilgan satrda bitta qo'shtirnoqni oddiy '' usulidan tashqari \' orqali ham kiritish mumkin.

Siz yaratayotgan bayt ketma-ketliklari, ayniqsa sakkizlik yoki o'n oltilik qochirishlardan foydalanganda, server belgilar kodlashida (character set encoding) to'g'ri belgilarni hosil qilishi — bu sizning javobgarligingiz. Foydali alternativa — Unicode qochirishlaridan yoki 4.1.2.3-bo'limda tushuntirilgan muqobil Unicode qochirish sintaksisidan foydalanish; bunda server belgi konvertatsiyasi mumkinligini tekshiradi.

**Ehtiyot bo'ling:** Agar standard_conforming_strings konfiguratsiya parametri o'chirilgan bo'lsa, PostgreSQL teskari chiziqli qochirishlarni ham oddiy, ham qochirilgan satr konstantalarida taniydi. Biroq, PostgreSQL 9.1 dan boshlab, standart holat — yoqilgan (on), ya'ni teskari chiziqli qochirishlar faqat qochirilgan satr konstantalarida taniladi. Bu xatti-harakat standartlarga ko'proq mos keladi, ammo teskari chiziq har doim tanilishi kerak bo'lgan eski xatti-harakatga tayanadigan ilovalarni buzishi mumkin. Vaqtincha yechim sifatida bu parametrni o'chirib qo'yish mumkin, ammo teskari chiziqli qochirishlardan foydalanishdan voz kechish yaxshiroqdir. Agar maxsus belgini ifodalash uchun teskari chiziqli qochirish kerak bo'lsa, satr konstantasini E bilan yozing.

standard_conforming_strings bilan bir qatorda, escape_string_warning va backslash_quote konfiguratsiya parametrlari ham satr konstantalaridagi teskari chiziqlarga munosabatni belgilaydi.

Kod raqami nol bo'lgan belgi satr konstantasida bo'lishi mumkin emas.

### 4.1.2.3. Unicode qochirishlariga ega satr konstantalari (String Constants with Unicode Escapes)

PostgreSQL, shuningdek, kod nuqtasi (code point) orqali ixtiyoriy Unicode belgilarini belgilashga imkon beruvchi yana bir qochirish sintaksisini qo'llab-quvvatlaydi. Unicode qochirilgan satr konstantasi ochiluvchi qo'shtirnoqdan darhol oldin, orasida bo'shliqsiz yoziladigan U& (katta yoki kichik U harfi, undan keyin ampersand) bilan boshlanadi, masalan, U&'foo'. (Bu & operatori bilan noaniqlik yaratishini unutmang. Bu muammoning oldini olish uchun operator atrofida bo'shliq qoldiring.) Qo'shtirnoqlar ichida Unicode belgilarini teskari chiziq va to'rt xonali o'n oltilik kod raqami yoki teskari chiziq, plyus belgisi va olti xonali o'n oltilik kod raqami orqali qochirilgan shaklda ko'rsatish mumkin. Masalan, 'data' satrini quyidagicha yozish mumkin:

```
U&'d\0061t\+000061'
```

Quyidagi ancha murakkabroq misolda rus tilidagi "slon" (fil) so'zi kiril harflarida yozilgan:

```
U&'\0441\043B\043E\043D'
```

Agar teskari chiziqdan boshqa qochirish belgisi kerak bo'lsa, uni satrdan keyin UESCAPE bandi yordamida belgilash mumkin, masalan:

```
U&'d!0061t!+000061' UESCAPE '!'
```

Qochirish belgisi sifatida o'n oltilik raqam, plyus belgisi, bitta qo'shtirnoq, qo'sh qo'shtirnoq yoki bo'shliq belgisidan boshqa istalgan bitta belgini ishlatish mumkin.

Qochirish belgisini satr ichida so'zma-so'z kiritish uchun uni ikki marta yozing.

UTF-16 surrogat juftliklarini belgilash uchun 4 xonali yoki 6 xonali qochirish shaklidan foydalanish mumkin, bu esa U+FFFF dan katta kod nuqtali belgilarni hosil qiladi, garchi 6 xonali shaklning mavjudligi texnik jihatdan buni keraksiz qilsa ham. (Surrogat juftliklar to'g'ridan-to'g'ri saqlanmaydi, balki bitta kod nuqtasiga birlashtiriladi.)

Agar server kodlashi UTF-8 bo'lmasa, ushbu qochirish ketma-ketliklaridan biri bilan belgilangan Unicode kod nuqtasi haqiqiy server kodlashiga aylantiriladi; agar bu imkonsiz bo'lsa, xato haqida xabar beriladi.

Shuningdek, satr konstantalari uchun Unicode qochirish sintaksisi faqat standard_conforming_strings konfiguratsiya parametri yoqilgan bo'lsagina ishlaydi. Buning sababi, aks holda bu sintaksis SQL bayonotlarini tahlil qiluvchi klientlarni chalkashtirib yuborishi mumkin, bu esa SQL in'ektsiyalari (SQL injections) va shunga o'xshash xavfsizlik muammolariga olib kelishi mumkin. Agar parametr o'chirilgan bo'lsa, bu sintaksis xato xabari bilan rad etiladi.

### 4.1.2.4. Dollar belgili satr konstantalari (Dollar-Quoted String Constants)

Satr konstantalarini belgilashning standart sintaksisi odatda qulay bo'lsa-da, kerakli satr ko'p bitta qo'shtirnoqlarni o'z ichiga olganda uni tushunish qiyinlashadi, chunki ularning har biri ikkilantirilishi kerak. Bunday holatlarda o'qish osonroq so'rovlar yozish uchun PostgreSQL "dollar qo'shtirnoqlash" (dollar quoting) deb ataladigan boshqa usulni taqdim etadi. Dollar belgili satr konstantasi dollar belgisi ($), ixtiyoriy "yorliq" (tag, nol yoki undan ko'p belgidan iborat), yana bitta dollar belgisi, satr mazmunini tashkil etuvchi ixtiyoriy belgilar ketma-ketligi, dollar belgisi, shu dollar qo'shtirnoqni boshlagan xuddi shu yorliq va dollar belgisidan iborat. Masalan, quyida "Dianne's horse" satrini dollar qo'shtirnoq yordamida belgilashning ikki xil usuli keltirilgan:

```
$$Dianne's horse$$
$SomeTag$Dianne's horse$SomeTag$
```

E'tibor bering, dollar belgili satr ichida bitta qo'shtirnoqlarni qochirmasdan ishlatish mumkin. Aslida, dollar belgili satr ichidagi hech qanday belgi hech qachon qochirilmaydi: satr mazmuni har doim so'zma-so'z yoziladi. Teskari chiziqlar maxsus emas, ochish yorlig'iga mos keladigan ketma-ketlikning bir qismi bo'lmasa, dollar belgilari ham maxsus emas.

Har bir ichma-ich (nesting) darajasida turli yorliqlarni tanlash orqali dollar belgili satr konstantalarini bir-biriga ichma-ich joylashtirish (nest) mumkin. Bu ko'pincha funksiya ta'riflarini yozishda qo'llaniladi. Masalan:

```sql
$function$
BEGIN
    RETURN ($1 ~ $q$[\t\r\n\v\\]$q$);
END;
$function$
```

Bu yerda $q$[\t\r\n\v\\]$q$ ketma-ketligi [\t\r\n\v\\] dollar belgili literal satrni ifodalaydi, u funksiya tanasi PostgreSQL tomonidan bajarilganda tanilinadi. Ammo bu ketma-ketlik tashqi dollar qo'shtirnoq chegaralovchisi ($function$) bilan mos kelmagani sababli, tashqi satr nuqtai nazaridan u shunchaki konstanta ichidagi qo'shimcha belgilar hisoblanadi.

Dollar belgili satrning yorlig'i, agar mavjud bo'lsa, dollar belgisini o'z ichiga ololmasligidan tashqari, qo'shtirnoqsiz identifikator bilan bir xil qoidalarga amal qiladi. Yorliqlar registrga sezgir, shuning uchun $tag$String content$tag$ to'g'ri, lekin $TAG$String content$tag$ noto'g'ri.

Kalit so'z yoki identifikatordan keyin keladigan dollar belgili satr undan bo'shliq bilan ajratilishi kerak; aks holda dollar qo'shtirnoq chegaralovchisi oldingi identifikatorning bir qismi sifatida qabul qilinadi.

Dollar qo'shtirnoqlash SQL standartining bir qismi emas, ammo murakkab satr literallarini yozishda standartga mos bitta qo'shtirnoq sintaksisiga qaraganda ko'pincha qulayroq usul hisoblanadi. Bu, ayniqsa, protsedura funksiyalarining ta'riflarida tez-tez kerak bo'ladigan, boshqa konstantalar ichida satr konstantalarini ifodalashda foydali. Bitta qo'shtirnoq sintaksisi bilan yuqoridagi misoldagi har bir teskari chiziq to'rtta teskari chiziq sifatida yozilishi kerak bo'lar edi, bu esa asl satr konstantasini tahlil qilishda ikkitaga, so'ngra ichki satr konstantasi funksiya bajarilishida qayta tahlil qilinganda bittaga qisqargan bo'lar edi.

### 4.1.2.5. Bit-satr konstantalari (Bit-String Constants)

Bit-satr konstantalari ochiluvchi qo'shtirnoqdan darhol oldin (orada bo'shliqsiz) B (katta yoki kichik) harfi qo'yilgan oddiy satr konstantalariga o'xshaydi, masalan, B'1001'. Bit-satr konstantalari ichida faqat 0 va 1 belgilariga ruxsat beriladi.

Muqobil ravishda, bit-satr konstantalarini o'n oltilik yozuvda, oldida X (katta yoki kichik) harfi bilan belgilash mumkin, masalan, X'1FF'. Bu yozuv har bir o'n oltilik raqam uchun to'rtta ikkilik raqamga ega bit-satr konstantasiga teng.

Bit-satr konstantasining ikkala shakli ham oddiy satr konstantalari kabi bir necha qatorga davom ettirilishi mumkin. Bit-satr konstantasida dollar qo'shtirnoqlashdan foydalanib bo'lmaydi.

### 4.1.2.6. Sonli konstantalar (Numeric Constants)

Sonli konstantalar quyidagi umumiy shakllarda qabul qilinadi:

```
digits
digits.[digits][e[+-]digits]
[digits].digits[e[+-]digits]
digitse[+-]digits
```

bu yerda digits bir yoki undan ko'p o'nlik raqam (0 dan 9 gacha). Agar o'nlik nuqta ishlatilgan bo'lsa, undan oldin yoki keyin kamida bitta raqam bo'lishi shart. Agar eksponent belgisi (e) mavjud bo'lsa, undan keyin kamida bitta raqam kelishi shart. Konstanta ichida bo'shliq yoki boshqa belgilarga, quyida tavsiflangan vizual guruhlash uchun ishlatiladigan pastki chiziqlardan tashqari, ruxsat berilmaydi. E'tibor bering, oldindagi plyus yoki minus belgisi konstantaning bir qismi hisoblanmaydi; u konstantaga qo'llaniladigan operatordir.

To'g'ri sonli konstantalarga ba'zi misollar:

```
42
3.5
4.
.001
5e2
1.925e-3
```

Bundan tashqari, o'nlik bo'lmagan butun son konstantalari quyidagi shakllarda qabul qilinadi:

```
0xhexdigits
0ooctdigits
0bbindigits
```

bu yerda hexdigits — bir yoki undan ko'p o'n oltilik raqam (0-9, A-F), octdigits — bir yoki undan ko'p sakkizlik raqam (0-7), bindigits esa bir yoki undan ko'p ikkilik raqam (0 yoki 1). O'n oltilik raqamlar va radiks prefikslari katta yoki kichik harfda bo'lishi mumkin. E'tibor bering, faqat butun sonlar o'nlik bo'lmagan shaklga ega bo'lishi mumkin, kasr qismli sonlar emas.

To'g'ri o'nlik bo'lmagan butun son konstantalariga ba'zi misollar:

```
0b100101
0B10011001
0o273
0O755
0x42f
0XFFFF
```

Vizual guruhlash uchun raqamlar orasiga pastki chiziqlar qo'yilishi mumkin. Ular konstantaning qiymatiga hech qanday qo'shimcha ta'sir ko'rsatmaydi. Masalan:

```
1_500_000_000
0b10001000_00000000
0o_1_755
0xFFFF_FFFF
1.618_034
```

Pastki chiziqlarga sonli konstanta yoki raqamlar guruhining boshida yoki oxirida (ya'ni o'nlik nuqta yoki eksponent belgisidan darhol oldin yoki keyin) ruxsat berilmaydi, va ketma-ket bir nechta pastki chiziqqa ham ruxsat berilmaydi.

O'nlik nuqta ham, eksponent ham o'z ichiga olmagan sonli konstanta, agar uning qiymati integer (butun son, 32 bit) tipiga sig'sa, dastlab shu tipda deb hisoblanadi; aks holda, agar qiymati bigint (64 bit) tipiga sig'sa, bigint deb hisoblanadi; aks holda numeric tipida deb qabul qilinadi. O'nlik nuqta va/yoki eksponentga ega konstantalar har doim dastlab numeric tipida deb hisoblanadi.

Sonli konstantaning dastlab belgilangan ma'lumot tipi shunchaki tip aniqlash algoritmlari uchun boshlang'ich nuqta xolos. Ko'p hollarda konstanta kontekstga qarab avtomatik ravishda eng mos tipga aylantiriladi (coerced). Zarur bo'lganda, sonli qiymatni aniq ma'lumot tipiga o'tkazish (casting) orqali uni shu tipda talqin qilinishini majburlash mumkin. Masalan, sonli qiymatni real (float4) tipida qaralishini quyidagicha majburlash mumkin:

```sql
REAL '1.23'  -- satr uslubi
1.23::REAL   -- PostgreSQL (tarixiy) uslubi
```

Bular aslida keyin muhokama qilinadigan umumiy tip o'tkazish (casting) yozuvlarining alohida holatlaridir.

### 4.1.2.7. Boshqa tiplardagi konstantalar (Constants of Other Types)

Ixtiyoriy tipdagi konstantani quyidagi yozuvlardan biri orqali kiritish mumkin:

```
type 'string'
'string'::type
CAST ( 'string' AS type )
```

Satr konstantasining matni type deb nomlangan tip uchun kirish konvertatsiya protsedurasiga (input conversion routine) uzatiladi. Natija ko'rsatilgan tipdagi konstanta bo'ladi. Agar konstantaning qanday tip bo'lishi kerakligida noaniqlik bo'lmasa (masalan, u to'g'ridan-to'g'ri jadval ustuniga tayinlanganda), aniq tip o'tkazishni tushirib qoldirish mumkin, bunday holda u avtomatik tarzda aylantiriladi.

Satr konstantasini oddiy SQL yozuvi yoki dollar qo'shtirnoqlash orqali yozish mumkin.

Funksiyaga o'xshash sintaksis yordamida ham tip aylantirishni belgilash mumkin:

```
typename ( 'string' )
```

lekin har bir tip nomini shu tarzda ishlatib bo'lmaydi; tafsilotlar uchun 4.2.9-bo'limga qarang.

::, CAST() va funksiya-chaqiruv sintaksislari, shuningdek, 4.2.9-bo'limda muhokama qilinganidek, ixtiyoriy ifodalarning ish vaqti (run-time) tip konvertatsiyasini belgilash uchun ham ishlatilishi mumkin. Sintaktik noaniqlikning oldini olish uchun type 'string' sintaksisi faqat oddiy literal konstantaning tipini belgilash uchun ishlatilishi mumkin. type 'string' sintaksisining yana bir cheklovi — u massiv (array) tiplari uchun ishlamaydi; massiv konstantasining tipini belgilash uchun :: yoki CAST() dan foydalaning.

CAST() sintaksisi SQL ga mos keladi. type 'string' sintaksisi standartning umumlashtirilgan shaklidir: SQL bu sintaksisni faqat bir nechta ma'lumot tiplari uchun belgilaydi, PostgreSQL esa uni barcha tiplar uchun ruxsat beradi. :: bilan sintaksis PostgreSQL'ning tarixiy foydalanishi, xuddi funksiya-chaqiruv sintaksisi kabi.

## 4.1.3. Operatorlar (Operators)

Operator nomi quyidagi ro'yxatdagi belgilardan tashkil topgan, ko'pi bilan NAMEDATALEN-1 (standart bo'yicha 63) belgidan iborat ketma-ketlikdir:

```
+ - * / < > = ~ ! @ # % ^ & | ` ?
```

Biroq, operator nomlariga nisbatan bir nechta cheklovlar mavjud:

-- va /* operator nomining hech bir joyida bo'lishi mumkin emas, chunki ular izohning boshlanishi sifatida qabul qilinadi.

Ko'p belgili operator nomi + yoki - bilan tugay olmaydi, agar nom quyidagi belgilardan kamida bittasini o'z ichiga olmasa:

```
~ ! @ # % ^ & | ` ?
```

Masalan, @- ruxsat etilgan operator nomi, lekin *- emas. Bu cheklov PostgreSQL'ga tokenlar orasida bo'shliq talab qilmasdan SQL standartiga mos so'rovlarni tahlil qilishga imkon beradi.

SQL standartiga mos kelmaydigan operator nomlari bilan ishlaganda, odatda noaniqlikning oldini olish uchun qo'shni operatorlarni bo'shliq bilan ajratish kerak bo'ladi. Masalan, agar siz @ nomli prefiks operatorini aniqlagan bo'lsangiz, X*@Y deb yozib bo'lmaydi; PostgreSQL buni ikkita operator nomi sifatida, bittasi emas, o'qishini ta'minlash uchun X* @Y deb yozish kerak.

## 4.1.4. Maxsus belgilar (Special Characters)

Alifbo-raqam (alphanumeric) bo'lmagan ba'zi belgilar operator bo'lishdan farqli maxsus ma'noga ega. Foydalanish tafsilotlarini tegishli sintaksis elementi tavsiflangan joydan topish mumkin. Ushbu bo'lim faqat shu belgilarning mavjudligini ma'lum qilish va maqsadlarini umumlashtirish uchun mavjud.

Dollar belgisi ($), undan keyin raqamlar kelsa, funksiya ta'rifi yoki tayyorlangan bayonot (prepared statement) tanasida pozitsion parametrni ifodalash uchun ishlatiladi. Boshqa kontekstlarda dollar belgisi identifikatorning yoki dollar belgili satr konstantasining bir qismi bo'lishi mumkin.

Qavslar (()) ifodalarni guruhlash va ustunlikni (precedence) ta'minlash uchun odatdagi ma'noga ega. Ba'zi hollarda qavslar muayyan SQL buyrug'ining qat'iy sintaksisining bir qismi sifatida talab qilinadi.

Kvadrat qavslar ([]) massiv (array) elementlarini tanlash uchun ishlatiladi. Massivlar haqida ko'proq ma'lumot uchun 8.15-bo'limga qarang.

Vergullar (,) ba'zi sintaktik qurilmalarda ro'yxat elementlarini ajratish uchun ishlatiladi.

Nuqta-vergul (;) SQL buyrug'ini tugatadi. U satr konstantasi yoki qo'shtirnoqqa olingan identifikator ichidan tashqari, buyruq ichida hech qayerda bo'lishi mumkin emas.

Ikki nuqta (:) massivlardan "kesim" (slice) tanlash uchun ishlatiladi (8.15-bo'limga qarang). Ba'zi SQL dialektlarida (masalan, Embedded SQL) ikki nuqta o'zgaruvchi nomlarini prefikslash uchun ishlatiladi.

Yulduzcha (*) ba'zi kontekstlarda jadval qatori yoki kompozit qiymatning barcha maydonlarini bildirish uchun ishlatiladi. U shuningdek agregat funksiya (aggregate function) argumenti sifatida ishlatilganda maxsus ma'noga ega bo'lib, agregat hech qanday aniq parametrni talab qilmasligini bildiradi.

Nuqta (.) sonli konstantalarda, shuningdek sxema, jadval va ustun nomlarini ajratish uchun ishlatiladi.

## 4.1.5. Izohlar (Comments)

Izoh — ikkita chiziqcha bilan boshlanib, qator oxirigacha davom etadigan belgilar ketma-ketligidir, masalan:

```sql
-- Bu standart SQL izohi
```

Muqobil ravishda, C uslubidagi blok izohlaridan ham foydalanish mumkin:

```
/* ko'p qatorli izoh
 * ichma-ich bilan: /* ichma-ich blok izoh */
 */
```

bu yerda izoh /* bilan boshlanib, mos keluvchi */ uchrashuviga qadar davom etadi. Ushbu blok izohlar, C tilidan farqli o'laroq, SQL standartida belgilangandek ichma-ich (nest) joylashishi mumkin, shunda mavjud blok izohlarni o'z ichiga olishi mumkin bo'lgan kattaroq kod bloklarini izohga aylantirish (comment out) mumkin bo'ladi.

Izoh keyingi sintaksis tahlilidan oldin kirish oqimidan olib tashlanadi va aslida bo'sh joy bilan almashtiriladi.

## 4.1.6. Operatorlar ustunligi (Operator Precedence)

4.2-jadvalda PostgreSQL'dagi operatorlarning ustunligi (precedence) va assotsiativligi (associativity) ko'rsatilgan. Ko'pchilik operatorlar bir xil ustunlikka ega va chapdan-o'ngga assotsiativ (left-associative). Operatorlarning ustunligi va assotsiativligi tahlilchi (parser)ga qat'iy o'rnatilgan. Agar bir nechta operatorli ifoda ustunlik qoidalari nazarda tutgandan boshqacha tarzda tahlil qilinishini xohlasangiz, qavslar qo'shing.

**4.2-jadval. Operatorlar ustunligi (eng yuqoridan eng pastgacha)**

| Operator/Element | Assotsiativlik | Tavsif |
|---|---|---|
| . | chapga | jadval/ustun nomi ajratuvchisi |
| :: | chapga | PostgreSQL uslubidagi tip o'tkazish |
| [ ] | chapga | massiv elementini tanlash |
| + - | o'ngga | unar plyus, unar minus |
| COLLATE | chapga | tartiblashni (collation) tanlash |
| AT | chapga | AT TIME ZONE, AT LOCAL |
| ^ | chapga | darajaga oshirish (exponentiation) |
| * / % | chapga | ko'paytirish, bo'lish, qoldiq (modulo) |
| + - | chapga | qo'shish, ayirish |
| (boshqa har qanday operator) | chapga | boshqa barcha o'rnatilgan va foydalanuvchi tomonidan aniqlangan operatorlar |
| BETWEEN IN LIKE ILIKE SIMILAR | | diapazonga tegishlilik, to'plamga a'zolik, satr mosligi |
| < > = <= >= <> | | taqqoslash operatorlari |
| IS ISNULL NOTNULL | | IS TRUE, IS FALSE, IS NULL, IS DISTINCT FROM va h.k. |
| NOT | o'ngga | mantiqiy inkor |
| AND | chapga | mantiqiy konyunksiya |
| OR | chapga | mantiqiy dizyunksiya |

E'tibor bering, operatorlar ustunligi qoidalari yuqorida sanab o'tilgan o'rnatilgan operatorlar bilan bir xil nomga ega bo'lgan foydalanuvchi tomonidan aniqlangan operatorlarga ham tegishli. Masalan, agar siz maxsus ma'lumot tipi uchun "+" operatorini aniqlasangiz, u sizniki nima qilishidan qat'iy nazar, o'rnatilgan "+" operatori bilan bir xil ustunlikka ega bo'ladi.

Sxema bilan malakalashtirilgan (schema-qualified) operator nomi OPERATOR sintaksisida ishlatilganda, masalan:

```sql
SELECT 3 OPERATOR(pg_catalog.+) 4;
```

OPERATOR qurilmasi 4.2-jadvalda "boshqa har qanday operator" uchun ko'rsatilgan standart ustunlikka ega deb qabul qilinadi. Bu OPERATOR() ichida qaysi aniq operator kelishidan qat'iy nazar to'g'ri.

**Eslatma:** 9.5 dan oldingi PostgreSQL versiyalari bir oz farqli operatorlar ustunligi qoidalaridan foydalangan. Xususan, <= >= va <> avval umumiy operatorlar sifatida ko'rib chiqilgan; IS testlari yuqoriroq ustunlikka ega bo'lgan; NOT BETWEEN va shunga o'xshash qurilmalar izchil bo'lmagan tarzda ishlagan, ba'zi hollarda BETWEEN emas, NOT ustunligiga ega deb qabul qilingan. Bu qoidalar SQL standartiga yaxshiroq mos kelish va mantiqan teng qurilmalarni izchil bo'lmagan tarzda ko'rib chiqishdan kelib chiqadigan chalkashlikni kamaytirish uchun o'zgartirilgan. Ko'p hollarda, bu o'zgarishlar hech qanday xatti-harakat o'zgarishiga olib kelmaydi, yoki ehtimol qavslar qo'shish orqali hal qilinadigan "bunday operator yo'q" xatolariga olib kelishi mumkin. Biroq, so'rov hech qanday tahlil xatosi haqida xabar berilmasdan xatti-harakatini o'zgartirishi mumkin bo'lgan chekka holatlar mavjud.
