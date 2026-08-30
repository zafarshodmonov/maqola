# RSA Shifrlash Algoritmi — To'liq Darslik

## Mundarija

1. [Kirish: Simmetrik vs Asimmetrik shifrlash](#1-kirish)
2. [Matematik asos](#2-matematik-asos)
3. [RSA algoritmi: qadam-qadam](#3-rsa-algoritmi)
4. [Nega RSA ishlaydi (isbot)](#4-nega-ishlaydi)
5. [To'liq raqamli misol](#5-toliq-misol)
6. [Pseudocode](#6-pseudocode)
7. [Murakkablik (Complexity) tahlili](#7-murakkablik)
8. [Xavfsizlik: hujumlar va himoya](#8-xavfsizlik)
9. [Implementatsiyalar: Python, C, C++](#9-implementatsiyalar)
10. [Amaliy RSA: OAEP, hibrid shifrlash, kalit o'lchamlari](#10-amaliy-rsa)
11. [Xulosa va qo'shimcha manbalar](#11-xulosa)

---

## 1. Kirish

### 1.1 Simmetrik shifrlash muammosi

Simmetrik shifrlashda (masalan, AES) shifrlash va deshifrlash uchun **bitta umumiy kalit (shared secret key)** ishlatiladi. Bu tez va samarali, lekin katta muammosi bor: **kalitni qanday xavfsiz uzatish mumkin?**

Tasavvur qiling — Alice Bob'ga shifrlangan xabar yubormoqchi. Ular oldindan umumiy kalitga kelishishlari kerak. Agar ular hech qachon xavfsiz kanalda uchrashmagan bo'lsa (masalan, internet orqali birinchi marta gaplashishyapti), bu kalitni qanday almashish mumkin? Kalitni ochiq kanal orqali yuborish — uni tutib olish (intercept) xavfini tug'diradi.

Bu muammo **kalit taqsimlash muammosi (key distribution problem)** deb ataladi.

### 1.2 Yechim: Asimmetrik (Public-Key) shifrlash

1970-yillarda Whitfield Diffie va Martin Hellman, keyinroq esa Ron **R**ivest, Adi **S**hamir va Leonard **A**dleman (RSA nomi shu uchun harflardan olingan, 1977-yil) — bu muammoni hal qiluvchi g'oyani ishlab chiqishdi: **ikkita bog'liq kalit** ishlatish.

- **Public key (ochiq kalit)** — hammaga ma'lum, hatto dushmanga ham. Shifrlash uchun ishlatiladi.
- **Private key (maxfiy kalit)** — faqat egasida saqlanadi. Deshifrlash uchun ishlatiladi.

Muhim xususiyat: **public key orqali shifrlangan narsani faqat mos private key bilan deshifrlash mumkin**, va public key'dan private key'ni hisoblash (amaliy jihatdan) imkonsiz bo'lishi kerak.

Bu quti va qulf metaforasi bilan tushuntiriladi: Bob ochiq qulfini (public key) hammaga tarqatadi. Alice xabarni qutiga solib, Bob'ning ochiq qulfi bilan qulflaydi. Endi bu qutini faqat Bob o'zining maxfiy kalitli (private key) bilan ocha oladi — hatto Alice o'zi ham qayta ocholmaydi.

### 1.3 RSA nima uchun muhim

RSA — birinchi va eng mashhur amaliy public-key kriptotizim. U ikki narsa uchun ishlatiladi:
1. **Shifrlash/deshifrlash** — maxfiylikni ta'minlash.
2. **Raqamli imzo (digital signature)** — autentifikatsiya va integrity uchun (private key bilan "imzolash", public key bilan tekshirish).

RSA'ning xavfsizligi bitta matematik muammoning qiyinligiga asoslangan: **katta sonni ikkita katta tub songa (prime factors) ajratish (factoring) amaliy jihatdan juda qiyin**, lekin ikki tub sonni ko'paytirish oson.

---

## 2. Matematik asos

RSA'ni tushunish uchun quyidagi tushunchalarni bilish shart.

### 2.1 Modular arifmetika (Modular Arithmetic)

`a mod n` — `a` sonini `n`ga bo'lganda qolgan qoldiq. Masalan: `17 mod 5 = 2`, chunki `17 = 3×5 + 2`.

**Kongruentsiya (congruence):** `a ≡ b (mod n)` degani `a` va `b` `n`ga bo'linganda bir xil qoldiq beradi (ya'ni `n | (a-b)`).

Modular arifmetikaning muhim xossalari (shifrlash/deshifrlashda ishlatiladi):
```
(a + b) mod n = ((a mod n) + (b mod n)) mod n
(a × b) mod n = ((a mod n) × (b mod n)) mod n
(a^k) mod n   = ((a mod n)^k) mod n
```
Bu xossalar tufayli katta sonlar bilan ishlaganda, har bir qadamda natijani `mod n` qilib kichraytirib borish mumkin — bu RSA'da amaliy jihatdan **hal qiluvchi** ahamiyatga ega, chunki shundamasa sonlar astronomik darajada katta bo'lib ketardi.

### 2.2 Tub sonlar (Prime Numbers)

Tub son — faqat 1 va o'ziga bo'linadigan, 1dan katta son (2, 3, 5, 7, 11, 13, ...). RSA ikkita katta tub sonni tanlashdan boshlanadi. Nega tub sonlar? Chunki tub sonlarning ko'paytmasini asl ko'paytuvchilarga ajratish (factorization) — bu hisoblash nazariyasida "qiyin muammo" hisoblanadi, agar sonlar yetarlicha katta bo'lsa.

### 2.3 Eyler totient funksiyasi (Euler's Totient Function) — φ(n)

`φ(n)` — `1` dan `n` gacha bo'lgan, `n` bilan **o'zaro tub (coprime, ya'ni GCD(a,n)=1)** bo'lgan sonlar soni.

Muhim formulalar:
- Agar `p` tub son bo'lsa: `φ(p) = p - 1` (chunki 1 dan (p-1) gacha barcha son p bilan o'zaro tub).
- Agar `n = p × q`, bunda `p` va `q` — turli tub sonlar: 
  ```
  φ(n) = φ(p) × φ(q) = (p-1)(q-1)
  ```
  Bu formula **totient funksiyaning multiplikativligi (multiplicative property)** natijasi — RSA'ning yuragi aynan shu yerda.

**Misol:** `p=5, q=11` bo'lsa, `n=55`, `φ(55) = 4×10 = 40`.

### 2.4 Eng katta umumiy bo'luvchi (GCD) va Evklid algoritmi

`GCD(a, b)` — `a` va `b` ni birgalikda bo'ladigan eng katta son. Evklid algoritmi (Euclidean Algorithm) buni tez topadi:
```
GCD(a, b) = GCD(b, a mod b),   GCD(a, 0) = a
```

**Kengaytirilgan Evklid algoritmi (Extended Euclidean Algorithm)** esa faqat GCD'ni emas, balki quyidagi tenglamani qanoatlantiruvchi `x, y` butun sonlarni ham topadi:
```
a×x + b×y = GCD(a, b)
```
Bu RSA'da **modular inverse** (`d = e⁻¹ mod φ(n)`) topish uchun ishlatiladi — buni 2.5-bo'limda ko'ramiz.

### 2.5 Modular multiplikativ teskari (Modular Multiplicative Inverse)

`a`ning `n` moduli bo'yicha teskarisi — shunday `x` soniki:
```
(a × x) ≡ 1 (mod n)
```
Bu `x` faqat `GCD(a, n) = 1` bo'lgandagina mavjud bo'ladi, va Kengaytirilgan Evklid algoritmi orqali topiladi.

**Misol:** `3`ning `mod 11` bo'yicha teskarisi nima? `3 × 4 = 12 ≡ 1 (mod 11)`. Demak, `3⁻¹ mod 11 = 4`.

RSA'da xususiy kalit `d` aynan shu — `e`ning `φ(n)` moduli bo'yicha teskarisi:
```
d = e⁻¹ mod φ(n)     ya'ni     e×d ≡ 1 (mod φ(n))
```

### 2.6 Eylerning teoremasi (Euler's Theorem)

RSA'ning ishlashini isbotlash uchun asosiy tayanch shu teorema:

> Agar `GCD(a, n) = 1` bo'lsa, u holda: `a^φ(n) ≡ 1 (mod n)`

Bu Fermaning kichik teoremasining (Fermat's Little Theorem) umumlashtirilgan holati (Ferma teoremasi faqat `n` tub son bo'lgandagi xususiy holat: `a^(p-1) ≡ 1 (mod p)`).

### 2.7 Tezkor modular darajaga ko'tarish (Fast Modular Exponentiation / Square-and-Multiply)

RSA'da `c = m^e mod n` va `m = c^d mod n` kabi amallar bajariladi, bunda `e`, `d`, `n` — yuzlab raqamli katta sonlar bo'lishi mumkin. Agar `m^e`ni to'g'ridan-to'g'ri hisoblab, keyin `mod n` qilsak — bu amaliy jihatdan imkonsiz (natija astronomik katta bo'ladi).

Yechim — **"square and multiply"** usuli: `e`ning ikkilik (binary) ko'rinishidan foydalanib, darajaga ko'tarishni `O(log e)` ta ko'paytirish-modul olish amaliga tushiramiz.

**G'oya:** `e`ni ikkilik sanoq sistemasida yozamiz. Masalan `e = 13 = 1101₂ = 8+4+1`. U holda:
```
m^13 = m^8 × m^4 × m^1
```
Har bir `m^(2^k)` avvalgisini kvadratga ko'tarish orqali topiladi (`m^2, m^4, m^8, ...`), va faqat `e`ning ikkilik yozuvida `1` turgan pozitsiyalardagilar ko'paytiriladi. Har qadamda natija darhol `mod n` qilinadi — shu sababli sonlar hech qachon juda katta bo'lib ketmaydi.

---

## 3. RSA algoritmi

RSA uchta asosiy bosqichdan iborat: **kalit generatsiyasi**, **shifrlash**, **deshifrlash**.

### 3.1 Bosqich 1 — Kalit generatsiyasi (Key Generation)

1. Ikkita katta, **tasodifiy va turli** tub son tanlanadi: `p` va `q`.
2. Modul hisoblanadi: `n = p × q`. Bu `n` ham public, ham private kalitning bir qismi bo'ladi.
3. Eyler totienti hisoblanadi: `φ(n) = (p-1)(q-1)`.
4. **Public exponent** `e` tanlanadi, shunday shartlar bilan: `1 < e < φ(n)` va `GCD(e, φ(n)) = 1` (ya'ni `e` va `φ(n)` o'zaro tub). Amaliyotda deyarli har doim `e = 65537` (`= 2^16 + 1`) ishlatiladi — chunki bu tub son, ikkilik ko'rinishida faqat 2ta `1` bit bor (tez hisoblash uchun qulay), va yetarlicha katta (kichik hujumlardan himoyalaydi).
5. **Private exponent** `d` hisoblanadi — `e`ning `φ(n)` moduli bo'yicha teskarisi (Kengaytirilgan Evklid algoritmi orqali): `d ≡ e⁻¹ (mod φ(n))`.
6. Natija:
   - **Public key**: `(n, e)` — hammaga tarqatiladi.
   - **Private key**: `(n, d)` — maxfiy saqlanadi.
   - `p`, `q`, va `φ(n)` — generatsiyadan so'ng **yo'q qilinishi kerak** yoki qattiq himoyalanishi kerak, chunki ular orqali `d`ni qayta hisoblash mumkin.

### 3.2 Bosqich 2 — Shifrlash (Encryption)

Xabar `m` (butun son sifatida ifodalangan, `0 ≤ m < n`) public key `(n, e)` bilan shifrlanadi:
```
c = m^e mod n
```
`c` — shifrmatn (ciphertext), u yuboriladi.

### 3.3 Bosqich 3 — Deshifrlash (Decryption)

Qabul qiluvchi shifrmatn `c`ni o'zining private key `(n, d)` bilan deshifrlaydi:
```
m = c^d mod n
```

Shu bilan asl xabar `m` tiklanadi.

---

## 4. Nega ishlaydi (matematik isbot)

Savol: nega `c^d mod n` aynan asl `m`ni beradi?

**Isbot:**

Shifrlash va deshifrlashni birlashtiramiz:
```
c^d mod n = (m^e)^d mod n = m^(e×d) mod n
```

`d` ning tanlanish shartiga ko'ra: `e×d ≡ 1 (mod φ(n))`, ya'ni shunday butun son `k` mavjudki:
```
e×d = 1 + k×φ(n)
```

Buni o'rniga qo'yamiz:
```
m^(e×d) mod n = m^(1 + k×φ(n)) mod n = m × (m^φ(n))^k mod n
```

Endi Eylerning teoremasidan foydalanamiz (2.6-bo'lim): agar `GCD(m, n) = 1` bo'lsa, `m^φ(n) ≡ 1 (mod n)`. Demak:
```
m × (m^φ(n))^k mod n = m × 1^k mod n = m mod n = m
```
(chunki `0 ≤ m < n` deb faraz qilingan).

**Natija:** `c^d mod n = m`. Aynan shu isbotlaymiz kerak edi. ∎

*Eslatma:* Agar `GCD(m, n) ≠ 1` bo'lsa ham (ya'ni `m` tasodifan `p` yoki `q`ga karrali bo'lsa), Xitoy qoldiqlar teoremasi (Chinese Remainder Theorem) yordamida isbot kengaytiriladi va natija baribir to'g'ri chiqadi — bu holat amaliyotda ehtimoli deyarli nolga teng (chunki `p, q` juda katta).

---

## 5. To'liq misol

Kichik sonlar bilan (faqat tushunish uchun — amaliyotda bunday kichik sonlar **mutlaqo xavfsiz emas**):

**1-qadam — Tub sonlarni tanlash:**
```
p = 61,  q = 53
```

**2-qadam — Modul:**
```
n = p × q = 61 × 53 = 3233
```

**3-qadam — Totient:**
```
φ(n) = (p-1)(q-1) = 60 × 52 = 3120
```

**4-qadam — `e`ni tanlash** (`GCD(e, 3120) = 1` bo'lishi kerak):
```
e = 17   (GCD(17, 3120) = 1 ✓)
```

**5-qadam — `d`ni hisoblash** (`17 × d ≡ 1 mod 3120`):

Kengaytirilgan Evklid algoritmi orqali:
```
d = 2753   (tekshiruv: 17 × 2753 = 46801 = 15×3120 + 1 ✓)
```

**Natija:**
- Public key: `(n=3233, e=17)`
- Private key: `(n=3233, d=2753)`

**6-qadam — Shifrlash**, faraz qilaylik `m = 65`:
```
c = 65^17 mod 3233 = 2790
```
(Bu hisob "square and multiply" usulida bajariladi — 6.4-bo'limdagi pseudocode'ga qarang.)

**7-qadam — Deshifrlash:**
```
m = 2790^2753 mod 3233 = 65
```

Asl xabar `65` muvaffaqiyatli tiklandi. ✓

---

## 6. Pseudocode

### 6.1 Eng katta umumiy bo'luvchi (Evklid algoritmi)

```
function GCD(a, b):
    while b != 0:
        a, b = b, a mod b
    return a
```

### 6.2 Kengaytirilgan Evklid algoritmi (modular inverse uchun)

```
function EXTENDED_GCD(a, b):
    if b == 0:
        return (a, 1, 0)          // gcd, x, y — bunda a*1 + b*0 = a
    (gcd, x1, y1) = EXTENDED_GCD(b, a mod b)
    x = y1
    y = x1 - (a div b) * y1
    return (gcd, x, y)

function MOD_INVERSE(e, phi):
    (g, x, y) = EXTENDED_GCD(e, phi)
    if g != 1:
        error "modular inverse mavjud emas"     // e va phi o'zaro tub bo'lishi shart
    return ((x mod phi) + phi) mod phi           // natijani musbat qilamiz
```

### 6.3 Tub sonlikni tekshirish (Miller–Rabin primality test)

Katta tasodifiy sonning tub yoki tub emasligini samarali tekshirish uchun ishlatiladi (probabilistik algoritm):

```
function IS_PRIME(n, k=40):           // k — aniqlik darajasi (iteratsiyalar soni)
    if n < 2: return false
    if n in {2, 3}: return true
    if n mod 2 == 0: return false

    // n - 1 = 2^r * d  ko'rinishida yozamiz, d — toq son
    r = 0; d = n - 1
    while d mod 2 == 0:
        d = d / 2
        r = r + 1

    repeat k times:
        a = RANDOM_INTEGER(2, n - 2)
        x = MOD_EXP(a, d, n)
        if x == 1 or x == n - 1:
            continue                   // bu iteratsiya "tub" deb hisoblaydi, davom etamiz
        composite = true
        repeat (r - 1) times:
            x = MOD_EXP(x, 2, n)
            if x == n - 1:
                composite = false
                break
        if composite:
            return false                // aniq murakkab (composite) son
    return true                          // katta ehtimol bilan tub son
```

### 6.4 Tezkor modular darajaga ko'tarish (Square-and-Multiply)

```
function MOD_EXP(base, exponent, modulus):
    if modulus == 1: return 0
    result = 1
    base = base mod modulus
    while exponent > 0:
        if exponent is odd:                       // eng kichik bit = 1
            result = (result * base) mod modulus
        exponent = exponent >> 1                   // eng kichik bitni tashlaymiz
        base = (base * base) mod modulus            // kvadratga ko'taramiz
    return result
```

### 6.5 Kalit generatsiyasi

```
function GENERATE_KEYPAIR(bit_length):
    p = GENERATE_RANDOM_PRIME(bit_length / 2)
    q = GENERATE_RANDOM_PRIME(bit_length / 2)
    while p == q:
        q = GENERATE_RANDOM_PRIME(bit_length / 2)

    n = p * q
    phi = (p - 1) * (q - 1)

    e = 65537                                   // standart tanlov
    while GCD(e, phi) != 1:
        e = NEXT_ODD_NUMBER(e)                  // deyarli hech qachon kerak bo'lmaydi

    d = MOD_INVERSE(e, phi)

    public_key  = (n, e)
    private_key = (n, d)
    return (public_key, private_key)

function GENERATE_RANDOM_PRIME(bits):
    repeat:
        candidate = RANDOM_ODD_INTEGER_WITH_BITLENGTH(bits)
        if IS_PRIME(candidate):
            return candidate
```

### 6.6 Shifrlash va deshifrlash

```
function ENCRYPT(message, public_key):
    (n, e) = public_key
    assert 0 <= message < n
    return MOD_EXP(message, e, n)

function DECRYPT(ciphertext, private_key):
    (n, d) = private_key
    return MOD_EXP(ciphertext, d, n)
```

---

## 7. Murakkablik tahlili

Bu yerda `k = n`ning bit uzunligi (masalan, 2048-bit RSA uchun `k=2048`).

| Amal | Vaqt murakkabligi | Izoh |
|---|---|---|
| GCD (Evklid) | `O(log(min(a,b)))` | juda tez |
| Extended GCD | `O(log(min(a,b)))` | GCD bilan bir xil tartibda |
| Modular exponentiation (square-and-multiply) | `O(log e)` ko'paytirish, har biri `O(k^2)` yoki tezroq | jami `O(k^3)` (naive multiplication bilan), yoki `O(k^2 log k)` (FFT-asosli ko'paytirish bilan) |
| Miller–Rabin primality test | `O(k^3)` bitta tekshiruv uchun (naive), amalda `O(k^2 · t)` t-iteratsiya bilan | probabilistik, xato ehtimoli `4^(-t)` dan kichik |
| Tub son generatsiyasi | `O(k^4)` atrofida | chunki `O(k)` ta nomzod sinaladi (Prime Number Theorem'ga ko'ra), har biri `O(k^3)` |
| **Key generation (to'liq)** | `O(k^4)` | eng qimmat bosqich, lekin faqat bir marta bajariladi |
| **Encryption** (`e=65537` bilan) | `O(k^2)` — `17` bit uzunlikdagi `e` uchun juda kam ko'paytirish kerak | tez |
| **Decryption** | `O(k^3)` (yoki CRT optimizatsiyasi bilan ~4x tezroq) | `d` odatda to'liq `k`-bitli, shuning uchun sekinroq |

### 7.1 Nega shifrlash deshifrlashdan tezroq

`e = 65537 = 10000000000000001₂` — atigi 2ta bit `1`. Shuning uchun `MOD_EXP(m, e, n)` faqat ~17 marta kvadratga ko'tarish va 1 marta qo'shimcha ko'paytirish talab qiladi. Ammo `d` — odatda `n` kabi to'liq (masalan, 2048 bit) tasodifiy son, ya'ni ~2048 marta amal kerak bo'ladi. Shu sababli **deshifrlash odatda shifrlashdan sezilarli darajada sekinroq**.

### 7.2 CRT optimizatsiyasi (Chinese Remainder Theorem)

Amaliy implementatsiyalarda (masalan, OpenSSL) deshifrlash tezligini oshirish uchun private key ichida `p, q, dp=d mod(p-1), dq=d mod(q-1), qinv=q⁻¹ mod p` ham saqlanadi. Bu orqali `m mod p` va `m mod q` alohida (kichikroq sonlar bilan, demak tezroq) hisoblanadi, so'ng CRT orqali birlashtiriladi — bu usul ~**4 barobar tezlashtiradi**.

### 7.3 Faktorlash muammosining murakkabligi (xavfsizlik asosi)

`n`ni `p×q`ga ajratishning eng yaxshi ma'lum klassik algoritmi — **General Number Field Sieve (GNFS)**, uning murakkabligi:
```
O(exp((64/9)^(1/3) × (ln n)^(1/3) × (ln ln n)^(2/3)))
```
Bu **sub-eksponensial**, ya'ni polinomial emas — shuning uchun `n` yetarlicha katta bo'lsa (masalan, 2048 bit), amaldagi kompyuterlar bilan faktorlash amaliy jihatdan imkonsiz (yuz minglab yillar talab qiladi, hozirgi texnologiya bilan).

*Eslatma:* Agar yetarlicha katta **kvant kompyuter** yaratilsa, Shor algoritmi (Shor's Algorithm) RSA'ni **polinomial vaqtda** (`O((log n)^3)`) buzishi mumkin. Shuning uchun kelajakda "post-quantum cryptography" ga o'tish rejalashtirilmoqda.

---

## 8. Xavfsizlik: hujumlar va himoya

RSA matematik jihatdan mustahkam bo'lsa-da, **noto'g'ri implementatsiya qilinsa** ko'plab hujumlarga ochiq bo'lib qoladi.

### 8.1 Kichik `e` hujumi (Håstad's Broadcast Attack)

Agar bitta xabar bir nechta qabul qiluvchiga bir xil kichik `e` (masalan `e=3`) bilan, lekin turli `n` bilan yuborilsa va **padding ishlatilmasa**, CRT yordamida `m^3`ni to'g'ridan-to'g'ri hisoblab, kub ildiz olish orqali `m`ni tiklash mumkin. **Yechim:** har doim tasodifiy padding (OAEP) ishlatish.

### 8.2 Umumiy modul hujumi (Common Modulus Attack)

Agar ikki foydalanuvchi bir xil `n`, lekin turli `e` bilan bitta xabarni shifrlasa, shifrmatnlarni bilgan uchinchi tomon (private key'siz ham) xabarni tiklashi mumkin (Extended Euclidean Algorithm orqali). **Yechim:** har bir foydalanuvchi o'z alohida `n`siga ega bo'lishi kerak.

### 8.3 Timing hujumlari (Timing Attacks)

Agar deshifrlash vaqti kiritilgan ma'lumotga (masalan, bit qiymatlariga) qarab farq qilsa, hujumchi vaqt o'lchovlari orqali `d`ni bosqichma-bosqich tiklashi mumkin. **Yechim:** **constant-time** implementatsiya va **blinding** (tasodifiy niqoblash) texnikasi.

### 8.4 Chosen Ciphertext Attack va Padding Oracle

Agar tizim "bu shifrmatn to'g'ri padding'ga egami yoki yo'qmi" degan ma'lumotni oshkor qilsa (masalan, xato xabari orqali), hujumchi ko'p so'rov yuborib xabarni tiklashi mumkin (Bleichenbacher hujumi, 1998). **Yechim:** **OAEP** padding sxemasi va xatolarni bir xil tarzda qaytarish.

### 8.5 Zaif tub sonlar / Wiener hujumi

Agar `p` va `q` bir-biriga juda yaqin bo'lsa, **Fermat factorization** orqali oson faktorlanadi. Agar `d` juda kichik bo'lsa (`d < n^0.25` atrofida), **Wiener's Attack** davomli kasrlar (continued fractions) orqali `d`ni tiklashi mumkin. **Yechim:** `p, q` yetarlicha katta va tasodifiy tanlanishi, `d` yetarlicha katta bo'lishi kerak.

### 8.6 Amaliy tavsiyalar (2026-yil holatiga ko'ra)

- **Kalit uzunligi:** kamida **2048 bit**, uzoq muddatli xavfsizlik uchun **3072–4096 bit**.
- **Padding:** har doim **OAEP** (Optimal Asymmetric Encryption Padding) ishlatish — hech qachon "raw"/"textbook" RSA ishlatmaslik.
- **`e` qiymati:** standart `65537`.
- **Tasodifiylik:** `p, q` generatsiyasi kriptografik jihatdan xavfsiz tasodifiy son generatori (CSPRNG) orqali bo'lishi shart.
- **Kutubxonalar:** o'z RSA implementatsiyangizni ishlab chiqarish (production) muhitida hech qachon ishlatmang — sinalgan kutubxonalardan foydalaning (OpenSSL, `cryptography` (Python), libsodium va h.k.).

---

## 9. Implementatsiyalar

Quyida uchta tilda RSA'ning to'liq ishlaydigan (kichik/demo o'lchamli, ta'lim maqsadida) implementatsiyasi keltirilgan. **Diqqat:** bu kodlar RSA'ning ichki mexanizmini tushunish uchun yozilgan — production'da hech qachon ishlatilmasin (padding yo'q, kalit generatsiyasi soddalashtirilgan).

### 9.1 Python implementatsiyasi

Python'da katta butun sonlar (arbitrary-precision integers) tildan tashqarida hech qanday kutubxonasiz mavjud — shuning uchun RSA'ni to'liq, real hajmdagi (masalan, 1024-bit) kalitlar bilan yozish mumkin.

```python
import random

# ---------- Miller-Rabin tub sonlikni tekshirish ----------
def is_prime(n, k=40):
    if n < 2:
        return False
    for p in (2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31):
        if n == p:
            return True
        if n % p == 0:
            return False

    r, d = 0, n - 1
    while d % 2 == 0:
        d //= 2
        r += 1

    for _ in range(k):
        a = random.randrange(2, n - 1)
        x = pow(a, d, n)
        if x == 1 or x == n - 1:
            continue
        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False
    return True


def generate_prime(bits):
    while True:
        candidate = random.getrandbits(bits) | (1 << bits - 1) | 1  # eng yuqori va eng past bitni 1 qilamiz
        if is_prime(candidate):
            return candidate


# ---------- Kengaytirilgan Evklid algoritmi ----------
def extended_gcd(a, b):
    if b == 0:
        return a, 1, 0
    g, x1, y1 = extended_gcd(b, a % b)
    x = y1
    y = x1 - (a // b) * y1
    return g, x, y


def mod_inverse(e, phi):
    g, x, _ = extended_gcd(e, phi)
    if g != 1:
        raise ValueError("Modular inverse mavjud emas")
    return x % phi


# ---------- Kalit generatsiyasi ----------
def generate_keypair(bits=1024):
    p = generate_prime(bits // 2)
    q = generate_prime(bits // 2)
    while p == q:
        q = generate_prime(bits // 2)

    n = p * q
    phi = (p - 1) * (q - 1)

    e = 65537
    if extended_gcd(e, phi)[0] != 1:
        raise ValueError("e va phi o'zaro tub emas, boshqa p,q tanlang")

    d = mod_inverse(e, phi)

    return (n, e), (n, d)   # (public_key, private_key)


# ---------- Shifrlash / Deshifrlash ----------
def encrypt(message: int, public_key):
    n, e = public_key
    assert 0 <= message < n, "Xabar n dan kichik bo'lishi kerak"
    return pow(message, e, n)


def decrypt(ciphertext: int, private_key):
    n, d = private_key
    return pow(ciphertext, d, n)


# ---------- Matnni butun songa aylantirish ----------
def text_to_int(text: str) -> int:
    return int.from_bytes(text.encode("utf-8"), "big")


def int_to_text(number: int) -> str:
    length = (number.bit_length() + 7) // 8
    return number.to_bytes(length, "big").decode("utf-8")


if __name__ == "__main__":
    print("Kalitlar generatsiya qilinmoqda (1024 bit)...")
    public_key, private_key = generate_keypair(bits=1024)
    print(f"Public key:  (n, e) = ({public_key[0]}, {public_key[1]})")
    print(f"Private key: (n, d) = (..., {private_key[1]})")

    message = "Salom, RSA!"
    m_int = text_to_int(message)

    ciphertext = encrypt(m_int, public_key)
    print(f"\nShifrmatn (ciphertext): {ciphertext}")

    decrypted_int = decrypt(ciphertext, private_key)
    decrypted_text = int_to_text(decrypted_int)
    print(f"Deshifrlangan xabar: {decrypted_text}")

    assert decrypted_text == message
    print("\n✓ Muvaffaqiyatli!")
```

### 9.2 C implementatsiyasi

C'da native katta son (bignum) turi yo'q, shuning uchun bu implementatsiya `unsigned long long` (64-bit) doirasidagi **kichik, faqat demo uchun** kalitlar bilan ishlaydi. Production'da **GMP** (`libgmp`) yoki **OpenSSL** kutubxonasi ishlatiladi.

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

typedef unsigned long long u64;
typedef __int128 u128;   // ko'paytirishda overflow bo'lmasligi uchun

// ---------- Tezkor modular darajaga ko'tarish ----------
u64 mod_exp(u64 base, u64 exp, u64 mod) {
    u64 result = 1;
    base %= mod;
    while (exp > 0) {
        if (exp & 1) {
            result = (u64)(((u128)result * base) % mod);
        }
        exp >>= 1;
        base = (u64)(((u128)base * base) % mod);
    }
    return result;
}

// ---------- Miller-Rabin tub sonlikni tekshirish ----------
int miller_rabin_trial(u64 n, u64 a, u64 d, int r) {
    u64 x = mod_exp(a, d, n);
    if (x == 1 || x == n - 1) return 1;   // "ehtimol tub"
    for (int i = 0; i < r - 1; i++) {
        x = (u64)(((u128)x * x) % n);
        if (x == n - 1) return 1;
    }
    return 0;   // aniq murakkab (composite)
}

int is_prime(u64 n) {
    if (n < 2) return 0;
    for (u64 p = 2; p <= 31 && p < n; p++) {
        if (n == p) return 1;
        if (n % p == 0) return 0;
    }

    u64 d = n - 1;
    int r = 0;
    while (d % 2 == 0) { d /= 2; r++; }

    u64 witnesses[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37};
    for (size_t i = 0; i < sizeof(witnesses) / sizeof(witnesses[0]); i++) {
        u64 a = witnesses[i];
        if (a >= n) continue;
        if (!miller_rabin_trial(n, a, d, r)) return 0;
    }
    return 1;
}

u64 random_prime(int bits) {
    u64 candidate;
    do {
        candidate = 0;
        for (int i = 0; i < bits; i++)
            candidate = (candidate << 1) | (rand() & 1);
        candidate |= (1ULL << (bits - 1)) | 1;   // yuqori va past bitni 1 qilamiz
    } while (!is_prime(candidate));
    return candidate;
}

// ---------- Kengaytirilgan Evklid algoritmi ----------
long long extended_gcd(long long a, long long b, long long *x, long long *y) {
    if (b == 0) { *x = 1; *y = 0; return a; }
    long long x1, y1;
    long long g = extended_gcd(b, a % b, &x1, &y1);
    *x = y1;
    *y = x1 - (a / b) * y1;
    return g;
}

u64 mod_inverse(u64 e, u64 phi) {
    long long x, y;
    long long g = extended_gcd((long long)e, (long long)phi, &x, &y);
    if (g != 1) { fprintf(stderr, "Modular inverse mavjud emas\n"); exit(1); }
    long long result = x % (long long)phi;
    if (result < 0) result += phi;
    return (u64)result;
}

int main(void) {
    srand((unsigned)time(NULL));

    // Demo uchun kichik (16-bit) tub sonlar — FAQAT O'RGANISH UCHUN, XAVFSIZ EMAS!
    u64 p = random_prime(16);
    u64 q = random_prime(16);
    while (q == p) q = random_prime(16);

    u64 n = p * q;
    u64 phi = (p - 1) * (q - 1);

    u64 e = 65537;
    long long x, y;
    if (extended_gcd((long long)e, (long long)phi, &x, &y) != 1) {
        fprintf(stderr, "e=%llu va phi=%llu o'zaro tub emas, dasturni qayta ishga tushiring\n", e, phi);
        return 1;
    }
    u64 d = mod_inverse(e, phi);

    printf("p = %llu, q = %llu\n", p, q);
    printf("n = %llu, phi(n) = %llu\n", n, phi);
    printf("Public key:  (n=%llu, e=%llu)\n", n, e);
    printf("Private key: (n=%llu, d=%llu)\n", n, d);

    u64 message = 12345 % n;   // demo xabar, n dan kichik bo'lishi shart
    printf("\nAsl xabar:      %llu\n", message);

    u64 ciphertext = mod_exp(message, e, n);
    printf("Shifrmatn:      %llu\n", ciphertext);

    u64 decrypted = mod_exp(ciphertext, d, n);
    printf("Deshifrlangan:  %llu\n", decrypted);

    printf(decrypted == message ? "\n✓ Muvaffaqiyatli!\n" : "\n✗ Xato!\n");
    return 0;
}
```

**Kompilyatsiya:** `gcc -O2 -o rsa_demo rsa_demo.c && ./rsa_demo`

### 9.3 C++ implementatsiyasi

C++ versiyasi xuddi shu mantiqni sinf (class) shaklida, biroz zamonaviyroq uslubda taqdim etadi.

```cpp
#include <iostream>
#include <random>
#include <cstdint>
#include <stdexcept>

using u64 = uint64_t;
using u128 = __uint128_t;

class RSA {
public:
    struct KeyPair {
        u64 n, e, d;
    };

    static KeyPair generate(int bits = 32) {
        u64 p = randomPrime(bits / 2);
        u64 q = randomPrime(bits / 2);
        while (q == p) q = randomPrime(bits / 2);

        u64 n = p * q;
        u64 phi = (p - 1) * (q - 1);

        u64 e = 65537;
        long long x, y;
        if (extendedGcd((long long)e, (long long)phi, x, y) != 1)
            throw std::runtime_error("e va phi(n) o'zaro tub emas");

        u64 d = modInverse(e, phi);
        return {n, e, d};
    }

    static u64 encrypt(u64 message, u64 n, u64 e) {
        if (message >= n) throw std::invalid_argument("Xabar n dan kichik bo'lishi kerak");
        return modExp(message, e, n);
    }

    static u64 decrypt(u64 ciphertext, u64 n, u64 d) {
        return modExp(ciphertext, d, n);
    }

private:
    static u64 modExp(u64 base, u64 exp, u64 mod) {
        u64 result = 1;
        base %= mod;
        while (exp > 0) {
            if (exp & 1)
                result = static_cast<u64>((u128)result * base % mod);
            exp >>= 1;
            base = static_cast<u64>((u128)base * base % mod);
        }
        return result;
    }

    static bool millerRabinTrial(u64 n, u64 a, u64 d, int r) {
        u64 x = modExp(a, d, n);
        if (x == 1 || x == n - 1) return true;
        for (int i = 0; i < r - 1; ++i) {
            x = static_cast<u64>((u128)x * x % n);
            if (x == n - 1) return true;
        }
        return false;
    }

    static bool isPrime(u64 n) {
        if (n < 2) return false;
        static const u64 smallPrimes[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31};
        for (u64 sp : smallPrimes) {
            if (n == sp) return true;
            if (n % sp == 0) return false;
        }

        u64 d = n - 1;
        int r = 0;
        while (d % 2 == 0) { d /= 2; ++r; }

        for (u64 a : smallPrimes) {
            if (a >= n) continue;
            if (!millerRabinTrial(n, a, d, r)) return false;
        }
        return true;
    }

    static u64 randomPrime(int bits) {
        static std::mt19937_64 rng(std::random_device{}());
        u64 candidate;
        do {
            candidate = rng() & ((1ULL << bits) - 1);
            candidate |= (1ULL << (bits - 1)) | 1ULL;
        } while (!isPrime(candidate));
        return candidate;
    }

    static long long extendedGcd(long long a, long long b, long long &x, long long &y) {
        if (b == 0) { x = 1; y = 0; return a; }
        long long x1, y1;
        long long g = extendedGcd(b, a % b, x1, y1);
        x = y1;
        y = x1 - (a / b) * y1;
        return g;
    }

    static u64 modInverse(u64 e, u64 phi) {
        long long x, y;
        long long g = extendedGcd((long long)e, (long long)phi, x, y);
        if (g != 1) throw std::runtime_error("Modular inverse mavjud emas");
        long long result = x % (long long)phi;
        if (result < 0) result += phi;
        return (u64)result;
    }
};

int main() {
    auto keys = RSA::generate(32);   // demo uchun kichik, XAVFSIZ EMAS

    std::cout << "n = " << keys.n << ", e = " << keys.e << ", d = " << keys.d << "\n";

    u64 message = 42;
    u64 ciphertext = RSA::encrypt(message, keys.n, keys.e);
    u64 decrypted = RSA::decrypt(ciphertext, keys.n, keys.d);

    std::cout << "Asl xabar:     " << message << "\n";
    std::cout << "Shifrmatn:     " << ciphertext << "\n";
    std::cout << "Deshifrlangan: " << decrypted << "\n";
    std::cout << (decrypted == message ? "✓ Muvaffaqiyatli!\n" : "✗ Xato!\n");

    return 0;
}
```

**Kompilyatsiya:** `g++ -O2 -std=c++17 -o rsa_demo rsa_demo.cpp && ./rsa_demo`

---

## 10. Amaliy RSA

Yuqoridagi "toza" ("textbook") RSA faqat ta'lim maqsadida. Real dunyoda ishlatiladigan RSA quyidagi qo'shimchalarga ega:

### 10.1 OAEP Padding

**OAEP (Optimal Asymmetric Encryption Padding)** — xabarni shifrlashdan oldin tasodifiy qo'shimcha ma'lumot bilan "to'ldiradi". Bu:
- Bir xil xabarni ikki marta shifrlaganda **turli** shifrmatn olishni ta'minlaydi (determinizmni yo'qotadi — bu xavfsizlik uchun muhim).
- Chosen-ciphertext hujumlariga qarshi himoya beradi.
- Xesh funksiyalar (masalan SHA-256) va maskalash funksiyasidan (MGF1) foydalanadi.

### 10.2 Hibrid shifrlash (Hybrid Encryption)

RSA — **sekin** (simmetrik algoritmlarga nisbatan yuzlab marta sekinroq) va faqat **cheklangan hajmdagi** ma'lumotni shifrlay oladi (`n`dan kichik bo'lishi kerak). Shuning uchun amaliyotda RSA **hech qachon** katta fayllarni to'g'ridan-to'g'ri shifrlash uchun ishlatilmaydi. Buning o'rniga:

1. Tasodifiy **simmetrik kalit** (masalan, AES-256 uchun 256-bit) generatsiya qilinadi.
2. Haqiqiy ma'lumot shu simmetrik kalit bilan **AES** orqali tez shifrlanadi.
3. Faqat **kichik simmetrik kalitning o'zi** RSA orqali shifrlanadi va yuboriladi.
4. Qabul qiluvchi RSA private key bilan simmetrik kalitni deshifrlaydi, so'ng AES bilan asl ma'lumotni ochadi.

Bu usul **TLS/HTTPS**, **PGP/GPG**, va deyarli barcha zamonaviy shifrlash tizimlarida ishlatiladi.

### 10.3 Raqamli imzo (Digital Signature)

RSA yana **imzolash** uchun ham ishlatiladi — bu shifrlashning "teskarisi":
1. Yuboruvchi xabarning **xesh (hash)** qiymatini hisoblaydi (masalan SHA-256 bilan).
2. Xeshni o'zining **private key** bilan "shifrlaydi" (aslida `hash^d mod n`) — bu **imzo**.
3. Qabul qiluvchi imzoni yuboruvchining **public key** bilan deshifrlaydi va olingan qiymatni xabarning o'z xeshi bilan solishtiradi. Mos kelsa — imzo haqiqiy va xabar o'zgartirilmagan.

Bu yerda ham **PSS (Probabilistic Signature Scheme)** kabi padding sxemasi ishlatiladi (xuddi OAEP shifrlash uchun bo'lgani kabi).

### 10.4 Kalit o'lchamlari bo'yicha tavsiyalar

| Kalit uzunligi | Xavfsizlik darajasi | Tavsiya |
|---|---|---|
| 512 bit | Buzilgan (soatlar ichida faktorlanadi) | Ishlatilmasin |
| 1024 bit | Zaif (yirik tashkilotlar buzishi mumkin) | Ishlatilmasin |
| 2048 bit | Amaldagi minimal standart | Ko'pchilik holat uchun yetarli |
| 3072 bit | Yuqori xavfsizlik | Uzoq muddatli maxfiylik uchun |
| 4096 bit | Juda yuqori, lekin sekin | Maxsus holatlar uchun |

---

## 11. Xulosa

RSA — public-key kriptografiyaning asosini tashkil etuvchi, oddiy va nafis matematik g'oyaga (tub sonlarni ko'paytirish oson, lekin ko'paytmani ajratish qiyin) asoslangan algoritm. Uning yuragida:

1. **Kalit generatsiyasi** — `p, q` tub sonlaridan `n, φ(n), e, d` hisoblash.
2. **Shifrlash/deshifrlash** — modular darajaga ko'tarish (`m^e mod n` va `c^d mod n`).
3. **Xavfsizlik** — katta sonni faktorlashning hisoblash jihatidan qiyinligiga tayanadi.
4. **Amaliyotda** — hech qachon "toza" holda emas, balki OAEP padding va hibrid shifrlash bilan birga ishlatiladi.

### Qo'shimcha o'rganish uchun mavzular

- **Diffie-Hellman kalit almashinuvi** — boshqa public-key yondashuv, kalit almashish uchun.
- **Elliptic Curve Cryptography (ECC)** — xuddi shu maqsadlarni ancha kichikroq kalitlar bilan bajaradigan zamonaviyroq alternativa.
- **Chinese Remainder Theorem (CRT)** — RSA deshifrlashni tezlashtirishda ishlatiladigan chuqurroq matematik vosita.
- **Shor's Algorithm** — kvant kompyuterlar RSA'ga nima uchun tahdid solishini tushunish uchun.
- **Post-Quantum Cryptography** (masalan, lattice-based algoritmlar, NIST tomonidan standartlashtirilgan Kyber/Dilithium) — kelajakdagi almashtiruvchi algoritmlar.
