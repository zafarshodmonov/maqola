# Tarmoqlar asoslari — Nazariy darslik

Ushbu darslik "Tarmoqlar asoslari. 1-qism" loyihasini bajarish uchun zarur bo'lgan barcha nazariy tushunchalarni qamrab oladi: OSI va TCP/IP modellari, IP-adressatsiya, ikkilik sanoq sistemasi, subtarmoqlarga bo'lish (subnetting), GNS3 muhiti, broadcast/multicast, ICMP va ARP protokollari.

---

## Mundarija
1. [OSI va TCP/IP modellari](#1-osi-va-tcpip-modellari)
2. [Ikkilik sanoq sistemasi](#2-ikkilik-sanoq-sistemasi)
3. [IP-manzillar va ularning tuzilishi](#3-ip-manzillar-va-ularning-tuzilishi)
4. [Tarmoq maskasi va CIDR-notatsiya](#4-tarmoq-maskasi-va-cidr-notatsiya)
5. [Subtarmoqlarga bo'lish (Subnetting)](#5-subtarmoqlarga-bolish-subnetting)
6. [GNS3 — tarmoq emulyatsiyasi muhiti](#6-gns3--tarmoq-emulyatsiyasi-muhiti)
7. [Broadcast va Multicast](#7-broadcast-va-multicast)
8. [ICMP protokoli](#8-icmp-protokoli)
9. [ARP protokoli](#9-arp-protokoli)
10. [Wireshark bilan ishlash asoslari](#10-wireshark-bilan-ishlash-asoslari)
11. [Amaliy misollar (worked examples)](#11-amaliy-misollar-worked-examples)

---

## 1. OSI va TCP/IP modellari

### 1.1. Nima uchun bu modellar kerak?

Tasavvur qiling: sizning kompyuteringiz boshqa davlatdagi serverga xabar yubormoqchi. Bu jarayonda juda ko'p narsa hal qilinishi kerak — elektr signallari qanday shakllantiriladi, ma'lumot qanday paketlarga bo'linadi, manzil qanday topiladi, xatolar qanday tuzatiladi va h.k. Agar buning barchasini bitta "katta" tizim sifatida qarasak, uni tushunish va rivojlantirish juda qiyin bo'lardi.

**Shuning uchun tarmoq ishi darajalarga (layer) bo'lingan.** Har bir daraja faqat o'z vazifasi bilan shug'ullanadi va o'zidan yuqori hamda pastdagi darajalar bilan qat'iy belgilangan "interfeys" orqali muloqot qiladi. Bu — dasturlashdagi modullashtirish tamoyiliga juda o'xshaydi: har bir modul o'z ishini qiladi va boshqalarning ichki tuzilishi bilan qiziqmaydi.

### 1.2. OSI modeli (7 daraja)

OSI (Open Systems Interconnection) — nazariy, ta'limiy modeldir. U 7 ta darajadan iborat:

| # | Daraja (ingl.) | Vazifasi | Misol |
|---|---|---|---|
| 7 | Application | Foydalanuvchi bilan bevosita ishlaydigan protokollar | HTTP, FTP, DNS |
| 6 | Presentation | Ma'lumotni kodlash, shifrlash, formatlash | SSL/TLS, JPEG |
| 5 | Session | Aloqa sessiyasini o'rnatish va boshqarish | NetBIOS, RPC |
| 4 | Transport | Ma'lumotni ishonchli/tezkor yetkazish | TCP, UDP |
| 3 | Network | Manzillash va marshrutlash (routing) | **IP**, ICMP |
| 2 | Data Link | Bitta jismoniy segment ichida uzatish | Ethernet, **ARP**, MAC-manzillar |
| 1 | Physical | Elektr signallar, kabellar, radiosignal | Ethernet kabeli, Wi-Fi |

**Eslab qolish uchun mnemonika (pastdan yuqoriga):** "**P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way" (Physical, Data Link, Network, Transport, Session, Presentation, Application).

Loyihamiz uchun eng muhimi — **1–4 darajalar**, chunki:
- IP va ICMP — 3-daraja (Network)
- ARP va Ethernet — 2-daraja (Data Link)
- Fizik ulanish GNS3'da — 1-daraja (Physical)

### 1.3. TCP/IP modeli (4 daraja)

Amaliyotda internet aynan TCP/IP modeliga asoslanadi — u OSI'ning soddalashtirilgan, amaliy versiyasi:

| TCP/IP darajasi | OSI darajasiga mosligi |
|---|---|
| Application | Application + Presentation + Session |
| Transport | Transport |
| Internet | Network |
| Network Access (Link) | Data Link + Physical |

**Nega ikkita model bor?** OSI — o'qitish va tushuntirish uchun qulay "xarita", TCP/IP esa haqiqatda internetning ishlash tamoyili. Amaliyotchilar ko'pincha ikkalasini aralashtirib ishlatadi: masalan, "3-daraja qurilmasi" deganda IP asosida marshrutlaydigan qurilmani (router) nazarda tutishadi — bu OSI atamasi, lekin amalda TCP/IP haqida gap ketadi.

### 1.4. Inkapsulyatsiya (encapsulation)

Ma'lumot yuqori darajadan pastga tushar ekan, har bir daraja o'z "sarlavhasini" (header) qo'shib boradi:

```
[Application data]
      ↓ Transport darajasi TCP/UDP header qo'shadi
[TCP header | Application data]
      ↓ Network darajasi IP header qo'shadi
[IP header | TCP header | Application data]
      ↓ Data Link darajasi Ethernet header (va trailer) qo'shadi
[Ethernet header | IP header | TCP header | Application data | Ethernet trailer]
```

Bu jarayon **inkapsulyatsiya** deb ataladi. Qabul qiluvchi tomonda esa teskari jarayon — **dekapsulyatsiya** sodir bo'ladi: har bir daraja o'ziga tegishli header'ni "yechib" oladi va qolganini yuqoriga uzatadi.

Wireshark'da paketni ochganingizda aynan shu qatlamlarni ko'rasiz — Frame → Ethernet → IP → ICMP (yoki TCP/UDP) tartibida.

---

## 2. Ikkilik sanoq sistemasi

### 2.1. Nega tarmoqlarda ikkilik sistema muhim?

Kompyuterlar va tarmoq qurilmalari ma'lumotni bitlar (0 va 1) ko'rinishida qayta ishlaydi. IP-manzil, subnet-maska — bularning barchasi aslida ikkilik sonlar bo'lib, faqat inson uchun qulay bo'lishi uchun o'nlik (decimal) ko'rinishda yoziladi. Subnetting'ni chuqur tushunish uchun ikkilik sistemani "avtomatik" darajada bilish shart.

### 2.2. O'nlikdan ikkilikka o'tkazish

Har bir bit ma'lum bir 2 ning darajasiga mos keladi (8 bitli oktet uchun):

| Bit pozitsiyasi | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Qiymati (2^n) | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

**Algoritm (eng katta qoldiq usuli):**
1. Sondan eng katta 2 ning darajasini ayirib bo'ladimi, tekshiring — bo'lsa, o'sha bitga 1 qo'ying va sonni kamaytiring.
2. Bo'lmasa, o'sha bitga 0 qo'ying.
3. Keyingi (kichikroq) darajaga o'ting va shu jarayonni takrorlang, toki son 0 ga aylanguncha.

**Misol: 178 ni ikkilikka o'tkazamiz**

| Daraja | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
|---|---|---|---|---|---|---|---|---|
| 178 - 128 = 50 → bit=1 | 1 | | | | | | | |
| 50 - 64? Yo'q → bit=0 | | 0 | | | | | | |
| 50 - 32 = 18 → bit=1 | | | 1 | | | | | |
| 18 - 16 = 2 → bit=1 | | | | 1 | | | | |
| 2 - 8? Yo'q → bit=0 | | | | | 0 | | | |
| 2 - 4? Yo'q → bit=0 | | | | | | 0 | | |
| 2 - 2 = 0 → bit=1 | | | | | | | 1 | |
| 0 - 1? Yo'q → bit=0 | | | | | | | | 0 |

Natija: **178 = 10110010**

Tekshirish: 128+32+16+2 = 178 ✓

### 2.3. Ikkilikdan o'nlikka o'tkazish

Teskari amal — har bir "1" turgan pozitsiyaning qiymatini qo'shish kifoya.

**Misol:** 10110010 = 128+0+32+16+0+0+2+0 = **178**

### 2.4. Tezkor usul (ketma-ket bo'lish)

Amaliy jihatdan qulayroq usul — sonni ketma-ket 2 ga bo'lib, qoldiqlarni pastdan yuqoriga yozib chiqish:

```
178 ÷ 2 = 89, qoldiq 0
89  ÷ 2 = 44, qoldiq 1
44  ÷ 2 = 22, qoldiq 0
22  ÷ 2 = 11, qoldiq 0
11  ÷ 2 = 5,  qoldiq 1
5   ÷ 2 = 2,  qoldiq 1
2   ÷ 2 = 1,  qoldiq 0
1   ÷ 2 = 0,  qoldiq 1
```
Qoldiqlarni pastdan yuqoriga o'qib: **10110010** — natija bir xil.

### 2.5. To'liq IP-manzilni ikkilikka o'tkazish

IP-manzil 4 ta oktetdan (har biri 8 bit, 0–255 oralig'ida) iborat bo'lgani uchun har bir oktet alohida-alohida ikkilikka o'tkaziladi va nuqta (`.`) o'rniga bo'sh joy bilan yoziladi:

**178.101.89.7** ni ikkilikka o'tkazamiz:
- 178 = 10110010
- 101 = 01100101
- 89  = 01011001
- 7   = 00000111

Natija: **10110010.01100101.01011001.00000111**

> **Muhim:** har bir oktet har doim 8 xonali qilib yoziladi (kerak bo'lsa, oldiga nol qo'shiladi) — masalan, 7 emas, 00000111.

---

## 3. IP-manzillar va ularning tuzilishi

### 3.1. IPv4-manzil nima?

IPv4-manzil — 32 bitdan (4 oktet × 8 bit) iborat, tarmoqdagi qurilmani noyob tarzda aniqlaydigan raqam. U ikki qismdan iborat:

```
[ Tarmoq qismi (Network) ] [ Xost qismi (Host) ]
```

- **Tarmoq qismi** — qaysi tarmoqqa tegishli ekanligini bildiradi (ko'cha nomi kabi).
- **Xost qismi** — o'sha tarmoq ichidagi aniq qurilmani bildiradi (uy raqami kabi).

Qaysi bitlar tarmoqqa, qaysilari xostga tegishli ekanini **subnet-maska** belgilaydi (4-bo'limga qarang).

### 3.2. IP-manzil klasslari (tarixiy, lekin tushunish uchun foydali)

| Klass | Birinchi oktet | Standart maska |
|---|---|---|
| A | 1–126 | /8 |
| B | 128–191 | /16 |
| C | 192–223 | /24 |
| D (multicast) | 224–239 | — |
| E (rezerv) | 240–255 | — |

Hozirda klasssiz adressatsiya (CIDR) ishlatiladi, lekin bu jadval multicast diapazonini (224–239) tushunish uchun kerak bo'ladi — 3-topshiriqda aynan shu diapazondagi manzilga murojaat qilasiz.

### 3.3. Xususiy (private) IP-manzillar

Quyidagi diapazonlar internetda marshrutlanmaydi va faqat lokal tarmoqlar ichida ishlatiladi (RFC 1918):

| Diapazon | Maska |
|---|---|
| 10.0.0.0 – 10.255.255.255 | /8 |
| 172.16.0.0 – 172.31.255.255 | /12 |
| 192.168.0.0 – 192.168.255.255 | /16 |

Loyihada tavsiya etilgan `10.10.10.0/24` ham aynan shu xususiy diapazondan olingan — GNS3'dagi laboratoriya tarmoqlari uchun bu odatiy amaliyotdir.

---

## 4. Tarmoq maskasi va CIDR-notatsiya

### 4.1. Subnet-maska qanday ishlaydi?

Subnet-maska — IP-manzil bilan bir xil uzunlikdagi (32 bit) son bo'lib, unda:
- **1** bitlar — tarmoq qismini,
- **0** bitlar — xost qismini bildiradi.

Masalan, `255.255.255.0` maskasi ikkilikda:
```
11111111.11111111.11111111.00000000
```
Bu yerda birinchi 24 ta bit (1) — tarmoq qismi, oxirgi 8 ta bit (0) — xost qismi ekanini bildiradi.

### 4.2. CIDR-notatsiya (`/n`)

CIDR (Classless Inter-Domain Routing) — maskani to'liq yozish o'rniga, undagi "1" bitlar sonini `/` belgisidan keyin yozish usuli:

```
192.168.1.0/24  ==  192.168.1.0 maskasi 255.255.255.0
10.10.10.0/24   ==  10.10.10.0 maskasi 255.255.255.0
172.16.0.0/12   ==  172.16.0.0 maskasi 255.240.0.0
```

**Nega bu qulay?** Chunki `/24` yozish `255.255.255.0` yozishdan qisqaroq va tarmoq hajmini tezda "boshda" hisoblash imkonini beradi.

### 4.3. Maskadan tarmoqdagi xostlar sonini hisoblash

Formula:
```
Mavjud xostlar soni = 2^(xost bitlari soni) − 2
```

Nega −2? Chunki har bir tarmoqda ikkita manzil **maxsus** hisoblanadi va xostga berilmaydi:
- **Network address** (barcha xost bitlari 0) — tarmoqning o'zini bildiradi.
- **Broadcast address** (barcha xost bitlari 1) — o'sha tarmoqdagi barcha qurilmalarga murojaat uchun.

**Misol: /24 tarmog'i**
- Xost bitlari: 32 − 24 = 8 bit
- Xostlar soni: 2^8 − 2 = 256 − 2 = **254 ta xost**

**Misol: /29 tarmog'i**
- Xost bitlari: 32 − 29 = 3 bit
- Xostlar soni: 2^3 − 2 = 8 − 2 = **6 ta xost**

---

## 5. Subtarmoqlarga bo'lish (Subnetting)

### 5.1. Nima uchun subnetting kerak?

Tasavvur qiling, tashkilotga bitta katta `/24` tarmoq (254 ta manzil) berilgan, lekin unda faqat 3 ta, 16 ta va 32 ta qurilma bor xolos. Agar hammasiga bitta umumiy tarmoqni bersak:
1. Manzillar behuda sarflanadi (254 tadan atigi ~51 tasi ishlatiladi).
2. Xavfsizlik pasayadi — barcha qurilmalar bir broadcast domenida bo'lib qoladi (masalan, texnologik segmentdagi qurilma foydalanuvchi segmentidagi qurilmalarni "ko'rib" turadi).
3. Trafik boshqarilmaydi — broadcast trafigi keraksiz joylarga ham tarqaladi.

**Subnetting** — katta tarmoqni ehtiyojga qarab kichikroq, mos hajmdagi qism-tarmoqlarga bo'lish jarayonidir.

### 5.2. Kerakli maskani qanday tanlash kerak (VLSM mantiqi)

Qadam-baqadam mantiq:

1. **Kerakli xostlar sonini aniqlang** (masalan, 16 ta).
2. **Formuladan foydalanib**, qaysi xost bitlar soni bu sonni sig'dira olishini toping: `2^n − 2 ≥ kerakli xostlar soni`.
3. **Maska uzunligini hisoblang**: `/(32 − n)`.

**Misol: 16 ta mashina uchun**
- `2^n − 2 ≥ 16`
- `n=4`: 2^4−2 = 14 → yetarli emas
- `n=5`: 2^5−2 = 30 → yetarli (va bироz zaxira ham qoladi)
- Demak, maska: `/(32−5)` = **/27** (255.255.255.224)

**Misol: 3 ta mashina uchun**
- `n=2`: 2^2−2 = 2 → yetarli emas
- `n=3`: 2^3−2 = 6 → yetarli
- Maska: `/(32−3)` = **/29** (255.255.255.248)

**Misol: 32 ta mashina uchun**
- `n=5`: 2^5−2 = 30 → yetarli emas (32 dan kam!)
- `n=6`: 2^6−2 = 62 → yetarli
- Maska: `/(32−6)` = **/26** (255.255.255.192)

> **Muhim tushuncha:** har doim kerakli sondan **kattaroq yoki teng** bo'lgan eng kichik blokni tanlaymiz. Aniq mos kelmasa (masalan, aynan 16 ta uchun /28 = 14 xost yetarli emas), keyingi kattaroq blokka o'tamiz.

### 5.3. Nega bu segmentlarni ajratish xavfsizlik nuqtai nazaridan muhim?

Bu — kiberxavfsizlikning fundamental tamoyili: **tarmoq segmentatsiyasi (network segmentation)**. Har bir segment o'z broadcast domeniga va (keyinchalik) o'z xavfsizlik siyosatiga (ACL, firewall qoidalari) ega bo'lishi mumkin. Masalan, agar server segmentidagi bitta qurilma buzib kirilsa, to'g'ri segmentatsiya tufayli tajovuzkor avtomatik ravishda foydalanuvchi segmentiga "sakrab" o'ta olmaydi.

---

## 6. GNS3 — tarmoq emulyatsiyasi muhiti

### 6.1. GNS3 nima va nega kerak?

GNS3 (Graphical Network Simulator-3) — haqiqiy tarmoq uskunalari (router, switch)ning dasturiy obrazlarini (masalan, Cisco IOS) kompyuteringizda ishga tushirish imkonini beruvchi vosita. Bu virtual mashina tushunchasiga o'xshaydi, faqat "operatsion tizim" o'rniga tarmoq qurilmasining dasturiy ta'minoti (firmware/IOS) ishga tushiriladi.

**Nega bu muhim?** Chunki haqiqiy Cisco routerlarini sotib olish qimmat va noqulay, GNS3 esa haqiqiy buyruqlar qatori (CLI) va haqiqiy protokol xatti-harakati bilan bepul mashq qilish imkonini beradi.

### 6.2. Asosiy tushunchalar

- **Loyiha (project)** — GNS3'dagi ishchi hujjat, unda topologiya (qurilmalar va ularning ulanishlari) saqlanadi.
- **Obraz (image/IOS)** — haqiqiy Cisco qurilmasining dasturiy "nusxasi". Loyihada Cisco 3745 IOS obrazidan foydalanish tavsiya etilgan.
- **Topologiya (topology)** — qurilmalar va ular orasidagi jismoniy/mantiqiy ulanishlarning sxemasi.
- **Konsol (console)** — qurilmaga buyruqlar kiritish uchun matnli interfeys (CLI).

### 6.3. Ishlash jarayonining umumiy mantiqi

1. GNS3 o'rnatiladi va Cisco IOS obrazi import qilinadi.
2. Yangi loyiha yaratiladi.
3. Kerakli qurilmalar (masalan, ikkita router) loyihaga qo'shiladi.
4. Qurilmalar orasida virtual kabel bilan ulanish o'rnatiladi.
5. Har bir qurilmaning konsoliga kirib, interfeyslarga IP-manzil beriladi (`interface`, `ip address` buyruqlari orqali).
6. Konfiguratsiya saqlanadi (`write memory` yoki `copy running-config startup-config`).
7. Emulyatsiya ishga tushiriladi va qurilmalar orasidagi trafik (masalan, `ping` orqali) tekshiriladi.

### 6.4. `dumpcap` uchun huquqlar nega kerak?

GNS3 paket trassasini yig'ish uchun ichki jarayonda Wireshark'ning `dumpcap` komponentidan foydalanadi. Ushbu komponent tarmoq interfeysidan xom (raw) trafikni o'qishi uchun maxsus tizim huquqiga muhtoj — shuning uchun `sudo chmod +x /usr/bin/dumpcap` buyrug'i bajariladi (bu — bajarish huquqini berish, lekin amaliyotda ko'proq huquqlarni to'g'ri sozlash uchun rasmiy hujjatlarga murojaat qilish tavsiya etiladi).

---

## 7. Broadcast va Multicast

### 7.1. Uzatish turlari (transmission types)

Tarmoqda ma'lumot uzatishning uchta asosiy turi mavjud:

| Tur | Kimga yuboriladi | Analogiya |
|---|---|---|
| **Unicast** | Bitta aniq qabul qiluvchiga | Shaxsiy xat |
| **Broadcast** | Tarmoqdagi **barcha** qurilmalarga | Ko'chada baland ovozda e'lon qilish |
| **Multicast** | Oldindan **obuna bo'lgan** qurilmalar guruhiga | Faqat ma'lum radio kanaliga sozlangan qabul qiluvchilar |

### 7.2. Broadcast qanday ishlaydi?

Broadcast-manzil — tarmoqdagi barcha xost bitlari **1** ga tenglashtirilgan manzil. Masalan, `192.168.1.0/24` tarmog'i uchun:
- Tarmoq manzili: `192.168.1.0` (barcha xost bitlari 0)
- Broadcast manzili: `192.168.1.255` (barcha xost bitlari 1)

L2 darajasida broadcast'ga mos MAC-manzil har doim `FF:FF:FF:FF:FF:FF` bo'ladi — bu maxsus, "hammaga" degani.

**Muammosi:** broadcast trafigi tarmoqdagi HAR BIR qurilmaga yetib boradi va uni qayta ishlashga majburlaydi. Katta tarmoqda bu **Broadcast Storm** (broadcast trafigining nazoratsiz ko'payib ketishi) muammosiga olib kelishi mumkin — tarmoq shunchalik broadcast trafigi bilan to'lib ketadiki, foydali trafik uchun joy qolmaydi.

### 7.3. Multicast qanday ishlaydi?

Multicast — broadcast'ning "aqlliroq" versiyasi: xabar faqat unga obuna bo'lgan (join qilgan) qurilmalarga yetib boradi, hammaga emas.

- Multicast IP-manzillar maxsus diapazonda: **224.0.0.0 – 239.255.255.255** (D klassi).
- Qurilma multicast xabarini olishi uchun avval o'sha multicast-guruhga "qo'shilishi" (join) kerak — bu odatda IGMP (Internet Group Management Protocol) orqali amalga oshiriladi.
- L2 darajasida multicast IP-manzillar maxsus formula orqali maxsus MAC-manzillarga (`01:00:5E:xx:xx:xx` diapazonida) map qilinadi.

**Amaliy misol:** video-konferensiya tizimlari yoki marshrutlash protokollari (masalan, OSPF) ko'pincha multicast'dan foydalanadi — chunki xabar faqat tegishli qurilmalarga yetib borishi kerak, tarmoqdagi barcha qurilmalarga emas.

---

## 8. ICMP protokoli

### 8.1. ICMP nima uchun kerak?

ICMP (Internet Control Message Protocol) — IP protokolining "yordamchisi". IP o'zi xatolar haqida xabar berish mexanizmiga ega emas — agar paket yetib bormasa yoki manzil mavjud bo'lmasa, buni kim xabar qiladi? Aynan shu vazifani ICMP bajaradi.

ICMP OSI'ning 3-darajasida (Network) ishlaydi, lekin TCP yoki UDP kabi "port" tushunchasiga ega emas — u to'g'ridan-to'g'ri IP ustida ishlaydi.

### 8.2. `ping` buyrug'i va ICMP Echo

`ping` — ICMP'ning eng mashhur amaliy qo'llanilishi. U ishlash mantig'i:

1. Yuboruvchi qurilma **ICMP Echo Request** (Type 8) paketini manzilga yuboradi.
2. Agar manzil mavjud va yetib borsa, u **ICMP Echo Reply** (Type 0) paketi bilan javob qaytaradi.
3. Yuboruvchi vaqt farqini (RTT — Round Trip Time) o'lchaydi va paket yo'qolganmi yoki yo'qmi aniqlaydi.

`ping` — tarmoq mavjudligini tekshirishning eng oddiy va tez usuli, biroq u faqat "yetib boradi/bormaydi" degan asosiy ma'lumotni beradi (loyiha matnida aytilganidek — bu "barmoq bilan turtish"ga o'xshaydi).

### 8.3. ICMP'ning boshqa muhim turlari

| Type | Nomi | Vazifasi |
|---|---|---|
| 0 | Echo Reply | Ping'ga javob |
| 3 | Destination Unreachable | Manzilga yetib bo'lmasligi haqida xabar |
| 8 | Echo Request | Ping so'rovi |
| 11 | Time Exceeded | TTL tugaganda (traceroute shu asosda ishlaydi) |

### 8.4. ICMP paketining tuzilishi (Wireshark'da ko'rish uchun)

Wireshark'da ICMP paketini ochganingizda quyidagi qatlamlarni ko'rasiz:
```
Frame → Ethernet II (L2, MAC-manzillar) → Internet Protocol Version 4 (L3, IP-manzillar) → Internet Control Message Protocol (ICMP ma'lumotlari: Type, Code, Checksum)
```

---

## 9. ARP protokoli

### 9.1. ARP nega kerak?

Bu — loyihaning eng muhim kontseptual nuqtalaridan biri: **IP-manzil va MAC-manzil ikki xil darajada ishlaydi va ular bir-biriga bog'lanishi kerak.**

- IP-manzil — 3-daraja (Network), mantiqiy manzil, "qaysi tarmoq va qaysi xost" degan ma'noni bildiradi.
- MAC-manzil — 2-daraja (Data Link), jismoniy manzil, tarmoq kartasiga "zavoddan" berilgan noyob identifikator.

Kompyuter boshqa qurilmaga paket yubormoqchi bo'lganda, u IP-manzilni biladi, lekin Ethernet darajasida paketni jo'natish uchun **MAC-manzil ham kerak** (chunki L2 kadr sarlavhasida MAC-manzillar bo'ladi). Aynan shu IP→MAC bog'lanishini aniqlash uchun **ARP (Address Resolution Protocol)** ishlatiladi.

### 9.2. ARP qanday ishlaydi (qadam-baqadam)

1. Kompyuter A, kompyuter B'ning IP-manzilini biladi, lekin uning MAC-manzilini bilmaydi (ARP-jadvalida yozuv yo'q).
2. Kompyuter A **ARP Request** (so'rov) paketini yuboradi. Bu paket **broadcast** tarzida yuboriladi — ya'ni L2 darajasida destination MAC-manzil `FF:FF:FF:FF:FF:FF` bo'ladi, chunki A hali B'ning MAC-manzilini bilmaydi va "hammaga" murojaat qiladi: *"Kimning IP-manzili shu — MAC-manzilingizni ayting!"*
3. Tarmoqdagi barcha qurilmalar ushbu so'rovni oladi, lekin faqat o'z IP-manziliga mos keladigan qurilma (B) javob beradi.
4. Kompyuter B **ARP Reply** (javob) paketini yuboradi — bu safar **unicast** tarzida, to'g'ridan-to'g'ri A'ning MAC-manziliga (chunki ARP Request paketida A o'z MAC-manzilini ham ko'rsatgan edi).
5. Kompyuter A javobni oladi va o'z **ARP-jadvaliga** (ARP cache/table) yangi yozuv qo'shadi: `IP-manzil → MAC-manzil`.
6. Endi A haqiqiy ma'lumot paketini to'g'ridan-to'g'ri B'ning MAC-manziliga yubora oladi.

### 9.3. Nega bu loyihadagi savolga javob beradi?

Loyihada so'raladi: *"Birinchi ARP-paketida so'rov qaysi MAC-manzilga yuboriladi? Bu manzil qanday nomlanadi?"* — Yuqoridagi mantiqdan kelib chiqib, bu **broadcast MAC-manzili** (`FF:FF:FF:FF:FF:FF`) bo'ladi, chunki so'rov yuborilayotganda A hali B'ning haqiqiy MAC-manzilini bilmaydi.

Javob paketi (ARP Reply) esa, aksincha, **unicast** — to'g'ridan-to'g'ri so'rov yuborgan qurilmaning MAC-manziliga yo'naltiriladi.

### 9.4. Gratuitous ARP nima? (mustaqil o'rganish uchun yo'nalish)

Gratuitous ARP — qurilma hech kimdan so'ralmasa ham, o'z IP↔MAC bog'lanishi haqida tarmoqqa "o'z-o'zidan" xabar beradigan maxsus ARP paketi. U odatda quyidagi holatlarda ishlatiladi:
- Qurilma tarmoqqa yangi ulanganda (IP-manzil to'qnashuvini tekshirish uchun).
- Qurilmaning IP-manzili o'zgarganda (boshqalarning ARP-jadvalini yangilash uchun).

---

## 10. Wireshark bilan ishlash asoslari

### 10.1. Wireshark nima qiladi?

Wireshark — tarmoq interfeysidan o'tayotgan barcha paketlarni "ushlab qolib" (capture), ularni inson o'qiy oladigan formatda ko'rsatadigan dastur. U trafikni real vaqtda ham kuzatishi, ham oldindan yozib olingan `.pcap` faylni ochib tahlil qilishi mumkin.

### 10.2. Interfeys tuzilishi

Wireshark oynasi uchta asosiy qismdan iborat:
1. **Packet List** — barcha ushlangan paketlar ro'yxati (vaqt, manba, manzil, protokol, uzunlik).
2. **Packet Details** — tanlangan paketning to'liq qatlamlar bo'yicha tafsiloti (Frame → Ethernet → IP → ICMP/ARP va h.k.) — daraja bo'yicha "ochib" ko'rish mumkin.
3. **Packet Bytes** — paketning xom (raw) baytlar ko'rinishi (hex + ASCII).

### 10.3. Loyihada nimalarga e'tibor berish kerak?

- **Ethernet II qatlamida**: `Source` va `Destination` MAC-manzillar — kimdan kimga jo'natilganini ko'rsatadi.
- **Internet Protocol qatlamida**: `Source` va `Destination` IP-manzillar, TTL qiymati.
- **ARP qatlamida**: `Sender MAC/IP` va `Target MAC/IP` maydonlari — Request paketida `Target MAC` odatda `00:00:00:00:00:00` yoki noma'lum bo'ladi (chunki hali so'ralmoqda).
- Filtrlash uchun quyidagi filtrlardan foydalanish qulay: `arp`, `icmp`, `ip.addr == 10.10.10.1`.

---

## 11. Amaliy misollar (worked examples)

Quyida loyihadagi 1-topshiriqqa o'xshash to'liq yechilgan misol keltirilgan — mantiqni tushunish uchun.

### Misol: N tashkiloti uchun subnetting

**Berilgan:** 3 ta segment — texnologik (3 ta mashina), server (16 ta mashina), foydalanuvchi (32 ta mashina). Boshlang'ich tarmoq: `192.168.10.0/24`.

**Qadam 1 — har bir segment uchun kerakli maskani hisoblash:**

| Segment | Kerakli xostlar | Formula | n (xost bit) | Maska |
|---|---|---|---|---|
| Foydalanuvchi | 32 | 2^n−2 ≥ 32 | n=6 (2^6−2=62) | /26 |
| Server | 16 | 2^n−2 ≥ 16 | n=5 (2^5−2=30) | /27 |
| Texnologik | 3 | 2^n−2 ≥ 3 | n=3 (2^3−2=6) | /29 |

**Qadam 2 — VLSM tamoyili bo'yicha manzillarni taqsimlash (eng kattasidan boshlab):**

```
192.168.10.0/26   → Foydalanuvchi segmenti (192.168.10.0 – 192.168.10.63)
                      Xostlar: .1 – .62, Broadcast: .63

192.168.10.64/27  → Server segmenti (192.168.10.64 – 192.168.10.95)
                      Xostlar: .65 – .94, Broadcast: .95

192.168.10.96/29  → Texnologik segment (192.168.10.96 – 192.168.10.103)
                      Xostlar: .97 – .102, Broadcast: .103
```

**Nega eng kattasidan boshlab taqsimlaymiz?** Bu — VLSM (Variable Length Subnet Mask)ning standart amaliyoti: agar kichik blokdan boshlasak, keyingi kattaroq blok chegaralarga "to'g'ri kelmasligi" (alignment) mumkin va manzillar behuda isrof bo'ladi. Eng kattadan boshlash bu muammoning oldini oladi.

---

## Xulosa

Ushbu darslikda ko'rib chiqilgan tushunchalar loyihaning barcha 4 ta topshirig'i uchun asos bo'ladi:
- **1-topshiriq** — ikkilik sistema va subnetting (2, 3, 4, 5-bo'limlar).
- **2-topshiriq** — GNS3 muhiti (6-bo'lim).
- **3-topshiriq** — broadcast/multicast, ICMP, Wireshark (7, 8, 10-bo'limlar).
- **4-topshiriq** — ARP, OSI 2-darajasi, Wireshark (1, 9, 10-bo'limlar).

Tavsiya: har bir topshiriqni boshlashdan oldin ushbu darslikning tegishli bo'limini qayta o'qib chiqing, so'ngra loyiha topshirig'idagi amaliy qismga o'ting.
