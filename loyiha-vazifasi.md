# Tarmoqlar asoslari. 1-qism — «Birinchi tajriba»

**Annotatsiya:** ushbu loyiha tarmoqlar asoslarini, OSI/TCP-IP modelini GNS3 muhitida ishlash misolida tahlil qilishni, shuningdek IP, ARP, ICMP protokollarini o'rganishni o'z ichiga oladi.

---

> 💡 Agar bu sizning birinchi loyihangiz bo'lsa, ushbu [forma](http://opros.so/kAnXy)ni to'ldiring.

> 💡 Ushbu loyiha bo'yicha fikr-mulohaza bildirish uchun [shu yerni bosing](https://new.oprosso.net/p/4cb31ec3f47a4596bc758ea1861fb624). Bu anonim va bizning jamoamizga ta'limni yaxshilashda yordam beradi. So'rovnomani loyihani bajarishdan darhol so'ng to'ldirishni tavsiya qilamiz.

## Mundarija
1. [I bob](#i-bob) \
   1.1. [Loyihaga oid tavsiyalar](#loyihaga-oid-tavsiyalar)
2. [II bob](#ii-bob) \
   2.1. [Kompyuter tarmog'i](#kompyuter-tarmogi)
3. [III bob](#iii-bob) \
   3.1. [1-topshiriq. Manzillarni hisoblash](#1-topshiriq-manzillarni-hisoblash) \
   3.2. [2-topshiriq. GNS3 bilan tanishuv](#2-topshiriq-gns3-bilan-tanishuv) \
   3.3. [3-topshiriq. Multicast so'rovlar](#3-topshiriq-multicast-sorovlar) \
   3.4. [4-topshiriq. ARP](#4-topshiriq-arp)

### Kirish

> Ulrix uchun kun ertalabdanoq muvaffaqiyatsiz boshlangan edi: mahalliy sehrgarlar laboratoriyani portlatib yuborishgan, o'tgan mashg'ulotda esa u yelkasini shikastlab qo'ygan bo'lib, bu endi uyqusiga xalaqit berardi. Bunga qo'shimcha ravishda, mentori uni faqat katta kurslardan eshitgan qadimiy runalar orasidagi tartibsizlikni tozalashga majbur qildi.

> Yosh yirtqich ovchisi kiyinib, qilichini olib, tez qadamlar bilan nonushta qilish uchun oshxonaga yugurdi. O'quvchilar uni "Terem" deb atashardi, chunki u yerdagi oshpazlar talabalarga "janob" va "xonim" deb murojaat qilishardi. Nonushtadan so'ng, kayfiyati ko'tarilgan holda, Ulrix hamma bo'sh deb hisoblagan burchakdagi kabinetga kirdi. Stol ustida belgilar o'yilgan tosh uyumi yotardi — bu, go'yo, bugun unga o'z to'shagida uxlash nasib etmasligidan darak berardi.

## I bob
### Loyihaga oid tavsiyalar

«21-Maktab»da qanday o'qish kerak:
- Butun kurs davomida siz ma'lumotni mustaqil ravishda topasiz. Ma'lumot izlashning barcha mavjud vositalaridan, masalan, Google va GigaChat'dan foydalaning. Ma'lumot manbalariga ehtiyot bo'ling: tekshiring, o'ylang, tahlil qiling, solishtiring.
- O'zaro o'qitish (P2P, Peer-to-Peer) — bu talabalar bir vaqtning o'zida ham o'qituvchi, ham o'quvchi rolida bo'lib, bilim va tajriba almashadigan jarayondir. Bu yondashuv nafaqat o'qituvchidan, balki bir-biringizdan ham o'rganish imkonini beradi, bu esa materialni chuqurroq tushunishga yordam beradi.
- Yordam so'rashdan tortinmang: atrofingizda xuddi shu yo'lni birinchi marta bosib o'tayotgan tengdoshlaringiz bor. Yordam so'rovlariga javob berishdan qo'rqmang. Sizning tajribangiz qimmatli va foydali — uni boshqa ishtirokchilar bilan baham ko'rishdan tortinmang.
- Ko'chirmang, agar yordamdan foydalansangiz — nima uchun, qanday va nima maqsadda ekanligini oxirigacha tushunib oling. Aks holda o'qishingiz hech qanday ma'noga ega bo'lmaydi.
- Agar biror narsada qotib qolsangiz va hamma narsani sinab ko'rgandek tuyulsa-yu, baribir qayerga borishni tushunmasangiz — shunchaki dam oling! Ishoning, bu maslahat kiberxavfsizlik bo'yicha ko'plab ekspertlarga o'z ishlarida yordam bergan. Bosh havosini almashtiring, miyangizni "qayta yuklang" va, ehtimol, keyingi safar kerakli yechim o'z-o'zidan xayolingizga keladi!
- Muhim bo'lgani faqat o'qishning natijasi emas, balki jarayonning o'zi ham. Vazifani shunchaki yechish emas, balki uni QANDAY yechishni tushunish kerak.
- Kiberxavfsizlik sohasida mahoratga erishish yo'lida siz «21-Maktab» Kiberxavfsizlik bo'yicha qo'llab-quvvatlovchi va ilhomlantiruvchi hamjamiyatining bir qismi bo'lish imkoniyatiga egasiz. Hamjamiyatdan yangi e'lonlarni olish uchun [RocketChat](https://rocketchat-student.21-school.ru/channel/cybersec_21)'ga qo'shiling, muloqot uchun esa [Telegram](https://t.me/+r5wufz8L3mUzOGUy)'ga a'zo bo'ling.

Loyiha bilan qanday ishlash kerak:
- Loyihani bajarishdan oldin uni GitLab'dan bir xil nomdagi repozitoriyga klonlash kerak.
- Barcha kod fayllari klonlangan repozitoriyning `src` papkasida yaratilishi kerak.
- Loyihani klonlagandan so'ng `develop` nomli branch yaratish va ishlanmani shu branchda olib borish kerak. Shundan so'ng GitLab'ga ham aynan `develop` branchini push qilish kerak.
- Sizning katalogingizda topshiriqlarda ko'rsatilganidan boshqa fayllar bo'lmasligi kerak.

Ogohlantirish:
- O'qishni o'yinlashtirish maqsadida loyiha hikoya shaklida taqdim etilgan, shunday qilib sizga murakkab topshiriqlar va Google'dagi tonna nazariyadan bir oz chalg'ish imkoniyati beriladi. Agar o'ylab topilgan hikoya sizga zerikarli va iste'dodsiz tuyulsa, buni fikr-mulohazada bemalol yozib qoldirishingiz mumkin — mualliff o'z yozuvchilik mahorati ustida ko'z yoshi to'ksin. Shuningdek, har bir topshiriqdagi kirish qismini o'tkazib yuborib, faqat mazmunli qismga e'tibor qaratishingiz ham mumkin.

## II bob
### Kompyuter tarmog'i

Kompyuter (hisoblash) tarmog'i — bir-biri bilan mantiqiy yoki jismoniy tarzda bog'langan va o'zaro aloqada bo'lgan qurilmalar hamda tizimlar majmuidir. Tarmoq qurilmalariga serverlar, kompyuterlar, telefonlar va boshqalar kiradi. Bunday tarmoqlarning o'lchami turlicha bo'lishi mumkin — ikki-uch qurilmadan iborat uy tarmog'idan tortib, korxona tarmog'i yoki butun internetgacha. Tarmoqlar — hajmli mavzu bo'lib, ushbu loyiha topshiriqlarini bajarishni ularning har biri bo'yicha asosiy nazariyani mustaqil o'rganishdan boshlash tavsiya etiladi (ushbu loyihada bizga kamida OSI modelining birinchi to'rtta darajasi bo'yicha bilim kerak bo'ladi), shundan so'ng laboratoriya ishiga o'tish kerak.

Tarmoqlar asoslari bilan tanishish uchun eng yaxshi variant — Cisco CCNA topshirishga tayyorgarlik qo'llanmalaridir, chunki ulardagi ma'lumot juda yaxshi tuzilgan va barcha asosiy zarur mavzularni qamrab oladi. Ingliz tilini yetarli darajada bilsangiz, nazariyani o'rganish uchun CCNA'ni video formatda o'rganishni tavsiya qilaman (masalan, CBT Nuggets yoki shunga o'xshash resurslarga e'tibor qaratishingiz mumkin). Cisco CCNA topshirishga tayyorgarlik qo'llanmasining bosma/elektron versiyasi bilan ham ishlash mumkin.

_Eslatma_: CCNA topshirishga tayyorgarlik qo'llanmasining rus tiliga tarjima qilingan versiyasini tarjimadagi xatolar va noaniqliklar tufayli ishlatish qat'iyan tavsiya etilmaydi.

Qo'shimcha ravishda Habr blogidagi [«Eng kichiklar uchun tarmoqlar»](https://habr.com/ru/articles/447080/) maqolasiga e'tibor qaratishingiz mumkin. Ushbu kursni tugatgandan so'ng tarmoqlar va ularning xavfsizligi mavzusidagi materiallarni o'rganishni davom ettirish uchun quyidagi materiallarga e'tibor qaratishni tavsiya qilamiz:
* Implementing and Operating Cisco Security Core Technologies;
* Cisco Enterprise Network Core Technologies.

## III bob
### 1-topshiriq. Manzillarni hisoblash

Ushbu topshiriqni yechish uchun quyidagi mavzularda bilim kerak: IP-manzillar, ikkilik sanoq sistemasi, tarmoqni subtarmoqlarga bo'lish, tarmoq maskasi.

> Ulrix, avvalambor, runalarni qayta sanashga qaror qildi. Qanday xato! Bechora runalar ikkilik sanoq sistemasida sanalishini bilmas edi. U runalarni birma-bir chetga qo'yib, ovoz chiqarib sanay boshladi. Ulrix 2 ta rune chetga qo'yib, allaqachon 10 tagacha sanab bo'lganidan keyingina bu ishning nozik tomonini angladi. Toshlarni to'plamlarga taqsimlash uchun endi o'zi uchun yangi bo'lgan sanoq sistemasidan foydalanishga to'g'ri keladi...

IP (Internet Protocol) — tarmoq darajasidagi protokol. Ha-ha, aynan shu protokol orqali 10 yil oldin internetdagi har qanday buzg'unchini aniqlashgan. Biroq uning asosiy vazifasi — turli IP-tarmoqlar o'rtasida trafik uzatilishini amalga oshirishdir. Bu esa IP-manzillar tufayli sodir bo'ladi. Bu manzillar sizning propiska manzilingizga o'xshaydi. Bunda o'xshatish to'g'ridan-to'g'ri: butun sayyora miqyosida mutlaqo bir xil manzillar bo'lmaydi, biroq turli shaharlarda bir xil uy, ko'cha va xonadon raqamlari bo'lishi mumkin (masalan, Moskva sh., Sovetskaya ko'chasi, 12-uy yoki Sankt-Peterburg sh., Sovetskaya ko'chasi, 12-uy).

[Mavzuga oid maqola](https://habr.com/ru/articles/134892/).

CIDR-notatsiya va tarmoq adressatsiyasi nima ekanligini mustahkamlash uchun quyidagi amaliy topshiriqlarni bajarishingiz kerak:

1. Quyidagi manzillar uchun ikkilik notatsiyani hisoblang:
    * 178.101.89.7
    * 201.57.153.161
2. Tasavvur qiling, N tashkilotining turli segmentlari uchun subtarmoqlarni ajratishingiz kerak: texnologik (3 ta mashina), server (16 ta mashina) va foydalanuvchi (32 ta mashinadan iborat) segmentlari.
3. Har bir segment uchun qanday maskali tarmoqlarni ajratish kerak? Javobingizni asoslab bering.

Topshiriqlar yechimi va asoslamasini `ip-1` nomli matn faylga yozing.

### 2-topshiriq. GNS3 bilan tanishuv

Ushbu topshiriqni yechish uchun quyidagi mavzularda bilim kerak: GNS3, Cisco obrazini GNS3'ga import qilish.

**Muhim**: topshiriqlarni bajarishda qurilmalar konfiguratsiyasini saqlashni unutmang.

> Ulrix barcha runalarni to'plamlarga ajratib bo'lgandan so'ng, ular uchun saqlash joyi tayyorlamaganini anglab, umidsizlikning butun og'rig'ini his qildi. Mentordan aniq ko'rsatmalarni eslay olmasa-da, toshlar saralangan va biror idishlarga ehtiyotkorlik bilan joylashtirilishi kerakligi aniq edi. Kabinetda esa chiroyli javon va shkaflar yo'q edi. Dastlab bu uni xafa qildi, ammo keyin u mentor bu idishlarni maxsus tayyorlamaganini — yosh shogirdni notrivial fikrlashga o'rgatish uchun qilganini angladi. Shuning uchun Ulrix javon va shkaflarni sehrli qo'ziqorin qoplaridan foydalanib emulyatsiya qilishga qaror qildi.

Ushbu va keyingi uchta blok bo'yicha laboratoriya ishlarini bajaradigan virtual muhitni tayyorlash uchun sizga GNS3 test muhitini sozlash va tayyorlash, shuningdek uning ishlash prinsiplarini o'rganish kerak bo'ladi. GNS3 — virtual mashinalarga o'xshab, tarmoq uskunalarining ishlashini emulyatsiya qilish imkonini beruvchi virtual muhitdir. GNS3 virtual muhitini tayyorlash uchun uni o'rnatish (agar oldindan o'rnatilmagan bo'lsa), loyiha yaratish va Cisco IOS obrazlarini import qilish kerak.

Cisco IOS obrazlarini ushbu [havola](https://blog.netskills.ru/2011/12/ios-gns3-ios-for-gns3.html) orqali yuklab oling, IOS 3745.

[Mavzuga oid maqola](https://docs.gns3.com/docs/getting-started/your-first-gns3-topology).

O'rnatishdan so'ng dumpcap uchun huquqlarni belgilang:
```
sudo chmod +x /usr/bin/dumpcap
```

Qiyinchiliklar yuzaga kelsa, quyidagi havolalar yordam beradi:
- [Github GNS3](https://github.com/GNS3/gns3-gui).

**Muhim**: GNS3'ni systemd xizmati sifatida o'rnatmang, chunki bunday o'rnatishda ba'zi versiyalarda loyiha yaratish bilan bog'liq xato (bug) mavjud.

[O'rnatish bo'yicha ko'rsatmalar](https://docs.gns3.com/docs/getting-started/installation/linux).

Javob sifatida Cisco 3745 qurilmasi import qilingan GNS3 loyihasini repozitoriyga yuklang.

### 3-topshiriq. Multicast so'rovlar

Ushbu topshiriqni yechish uchun quyidagi mavzularda bilim kerak: broadcast, multicast, ICMP, Wireshark.

> Barcha runalar qoplarga joylashtirilgandan so'ng, faqat mayda ish qoldi: yuzlab kilogramm toshlarni podvalga tashish, toki kabinet birdaniga omborxonaga aylanib qolmasin. Afsuski, og'riyotgan yelka Ulrixga hatto bitta qopni ko'tarish imkonini bermadi.

> «Ish yomon», — deb o'yladi Ulrix va shundan so'ng o'ziga dedi: «Yordam chaqirishga to'g'ri keladi».

> Kabinet yonidan o'tayotgan o'quvchilar uni eshitishi uchun, bo'lajak yuk tashuvchi bor ovozda hammaga birdan murojaat qilib qichqirdi:
> — Yordam bering! Kimdir!

Broadcast — tarmoqdagi barcha tugunlarga trafik uzatish uchun ishlatiladigan keng translyatsiyali manzil (ham L2, ham L3 darajasida amalga oshiriladi). Bu, o'z navbatida, faqat siz bilan bir tarmoqda bo'lganlargina eshitadigan, karnayga baqirishga o'xshaydi. Bunday «baqirish» uchun har bir tarmoqda doim alohida manzil ajratiladi, odatda eng oxirgisi (masalan, 192.168.1.0/24 tarmog'i uchun bu 192.168.1.255).

Multicast — tarmoqdagi tugunlar guruhiga trafik uzatish uchun ishlatiladigan manzil. Aslida, broadcast'ning qisqartirilgan varianti.

ICMP — Internet Control Message Protocol, trafik uzatishdagi xatoliklar haqidagi ma'lumotni uzatish uchun ishlatiladigan protokol. Tarmoq mavjudligini tekshirishning mashhur usuli — `ping` buyrug'i bo'lib, u har bir operatsion tizimda mavjud va ICMP echo so'rovlarini yuborish imkonini beradi. Ular o'zida hech qanday foydali ma'lumotni saqlamaydi, bu qurilmaga «barmoq bilan turtish»ga o'xshaydi.

[Mavzuga oid maqola](https://habr.com/ru/articles/217585/).

Sizning vazifangiz:
- Loyiha yarating.
- Unga ikkita tarmoq qurilmasini qo'shing (Cisco 3745 obrazidan foydalaning), ularni ulang, tarmoq interfeyslarini sozlang.
- Emulyatsiyani ishga tushiring va multicast IP-manzilga ICMP orqali murojaat qilib, paket trassasini yig'ing.
- Wireshark\* yordamida natijaviy trassirovka faylini o'rganing, L2, L3 va ICMP ma'lumotlarini aks ettiruvchi maydonlarga e'tibor bering.
- Savolga javob bering: ICMP so'rovi qaysi destination MAC-manzilga yuboriladi? Javobni `multicast` nomli matn faylga yozing.
- Yig'ilgan paket trassirovkasi natijaviy faylini .pcap kengaytmali `multicast` nomli faylga saqlang.

\* Wireshark — trafikni tahlil qilish uchun dastur. U ham trafikni yozib olish, ham allaqachon yozib olingan trafikni ko'rish imkonini beradi. Ayniqsa tarmoqda nima sodir bo'layotganini qo'lda ko'rish kerak bo'lganda tez-tez ishlatiladi. Interfeysi ancha "og'ir" bo'lsa-da, [rasmiy hujjatlar](https://wiki.wireshark.org/) tushunib olishga yordam beradi.

Yakuniy javob sifatida repozitoriyga `multicast.pcap` fayli va savolga javob bilan `multicast` matn faylini joylashtiring.

_Maslahat_: qurilma multicast-manzil bo'yicha murojaatlarga javob berishi uchun interfeysni multicast-guruhga qo'shish kerak.

Ishda qiyinchiliklar yuzaga kelsa, ushbu [Yordam](https://docs.gns3.com/docs/getting-started/your-first-cisco-topology) sahifasidan foydalanishingiz mumkin.

**Maslahatlar**:
* Tarmoq interfeyslariga "murakkab" IP-manzillar bermang, oddiyroqlaridan foydalaning, masalan, 10.10.10.0/24.
* Buyruqlar qatori interfeysida yordam chiqarish uchun «?» kiriting.
* Buyruqni kiritishni tugatish uchun TAB tugmasidan foydalaning.

Qo'shimcha ravishda, ushbu topshiriqni bajargandan so'ng, Broadcast Storm nima ekanligini mustaqil o'rganishni tavsiya qilamiz.

### 4-topshiriq. ARP

Ushbu topshiriqni yechish uchun quyidagi mavzularda bilim kerak: ARP, OSI modelining 2-darajasi, Wireshark, GNS3.

> Yordam darhol kelmadi. Ammo qanday yordam! Maftunkor qiz kabinetga mo'ralab, savol nazari bilan Ulrixga qaradi. Keyin — qoplarga. Keyin — yana Ulrixga. Uning ko'zlarida avval hafsalasizlik, keyin qiziqish, g'azab va, nihoyat, qochib ketish istagi yaltirab o'tdi. Ammo altruizm g'olib chiqdi. Ikkovlashib bir necha soatda barcha qoplarni podvalga tashib bo'lishdi. Keyinchalik topish osonroq bo'lishi uchun har bir qopni ma'lum raqamli katakka joylashtirish qoldi.

ARP — Address Resolution Protocol, ma'lum IP-manzil asosida MAC-manzilni aniqlash imkonini beradi. Har bir qurilmada ARP-jadval mavjud bo'lib, u "kundalik" vazifasini bajaradi — unda IP-manzillar ularga bog'langan MAC-manzillar bilan birga yozib qo'yiladi.

[Mavzuga oid maqola](https://habr.com/ru/articles/138043/).

ARP misolida asosiy tarmoq protokollarini o'rganishni davom ettiramiz. Uni ham loyihamizdagi GNS3'da ikki tarmoq tuguni orasidagi trafikni tahlil qilish formatida ko'rib chiqamiz.

Sizning topshirig'ingiz:
- Paket trassirovkasini yig'ishni boshlang, tugunlardan birida qo'shni qurilmaning IP-manziligacha `ping` buyrug'ini bajaring. Shundan so'ng ARP-jadvalda tegishli ARP-yozuv mavjudligiga ishonch hosil qiling.
- Wireshark'da ARP va ICMP paketlarining tarkibi bilan tanishing.
- Birinchi ARP-paketida so'rov qaysi MAC-manzilga yuboriladi? Bu manzil qanday nomlanadi? Javoblarni `arp` nomli matn faylga yozing.
- Javob paketi qaysi MAC-manzildan kelayotganiga e'tibor bering. Wireshark'ning natijaviy .pcap faylini `arp` nomi bilan saqlang.

Yakunda repozitoriyga uchta faylni yuklang: `gns3` loyihasi, `arp.pcap` fayli va `arp` matn fayli.

Qo'shimcha ravishda, ushbu topshiriqni bajarishda Gratuitous ARP nima ekanligini mustaqil o'rganishni tavsiya qilamiz.
