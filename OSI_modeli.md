# OSI Model nima?

Open Systems Interconnection (OSI) model — bu tarmoq aloqasi funksiyalarini yettita layer (qatlam)ga bo'ladigan konseptual freymvork (framework). Tarmoq orqali ma'lumot yuborish murakkab jarayon, chunki turli xil hardware va software texnologiyalari geografik va siyosiy chegaralar bo'ylab birgalikda ishlashi kerak. OSI data model computer networking uchun universal "til" beradi — shu tufayli har xil texnologiyalar standart protokollar yoki aloqa qoidalari orqali bir-biri bilan gaplasha oladi. Har bir layerdagi texnologiya networkingda foydali bo'lishi uchun muayyan imkoniyatlarni ta'minlashi va aniq funksiyalarni bajarishi shart. Yuqori layerlardagi texnologiyalar abstraction (mavhumlashtirish)dan foyda ko'radi — ular quyi darajadagi texnologiyalarning ichki implementatsiya detallari haqida qayg'urmasdan ulardan foydalanishi mumkin.

## OSI model nima uchun muhim?

Open Systems Interconnection (OSI) modelining layerlari software va hardware komponentlaridagi har qanday turdagi tarmoq aloqasini o'z ichiga oladi (encapsulate qiladi). Model ikkita mustaqil sistema joriy ishlayotgan layerga asoslangan standartlashtirilgan interfeyslar yoki protokollar orqali bir-biri bilan aloqa qila olishi uchun yaratilgan.

OSI modelining afzalliklari quyida keltirilgan.

### **Murakkab sistemalarni umumiy tushunish**

Muhandislar murakkab tarmoq sistema arxitekturalarini tashkillashtirish va modellashtirish uchun OSI modeldan foydalanishlari mumkin. Ular har bir sistema komponentining ishlash layerini uning asosiy funksionalligiga qarab ajratib olishlari mumkin. Sistemani abstraction orqali kichikroq, boshqarish mumkin bo'lgan qismlarga bo'lish qobiliyati odamlarga uni yaxlit holda tasavvur qilishni osonlashtiradi.

### **Tezroq research va development**

OSI reference model yordamida muhandislar o'z ishlarini yaxshiroq tushunishlari mumkin. Ular bir-biri bilan aloqa qilishi kerak bo'lgan yangi, tarmoqlangan sistemalarni yaratayotganda qaysi texnologik layer (yoki layerlar) uchun development qilayotganlarini bilishadi. Muhandislar tarmoqlangan sistemalarni ishlab chiqishlari va bir qator takrorlanadigan (repeatable) jarayonlar hamda protokollardan foydalanishlari mumkin.

### **Moslashuvchan standartlashtirish**

OSI model layerlar orasida qaysi protokollardan foydalanish kerakligini emas, balki protokollar bajaradigan vazifalarni belgilaydi. U tarmoq aloqasi developmentini standartlashtiradi — shu tufayli odamlar sistema haqida oldindan hech qanday bilimga ega bo'lmasdan ham murakkab sistemalarni tez tushunishi, qurishi va qismlarga ajratishi mumkin. Bundan tashqari, u detallarni abstraction qiladi, shuning uchun muhandislar modelning har bir jihatini tushunishlari shart emas. Zamonaviy ilovalarda networking va protokollarning quyi darajalari sistema dizayni va developmentini soddalashtirish uchun abstraction qilinadi. Quyidagi rasmda OSI modeli zamonaviy application developmentda qanday qo'llanilishi ko'rsatilgan.

![OSI modelining zamonaviy application developmentda qo'llanilishi](media/image1.png)

## OSI modelining yettita layeri qanday?

Open Systems Interconnection (OSI) model International Organization for Standardization va boshqalar tomonidan 1970-yillarning oxirida ishlab chiqilgan. U birinchi marta 1984-yilda ISO 7498 sifatida nashr etilgan, joriy versiyasi esa ISO/IEC 7498-1:1994 hisoblanadi. Modelning yettita layeri quyida keltirilgan.

### **Physical layer**

Physical layer fizik aloqa muhitiga va shu muhit orqali ma'lumot uzatadigan texnologiyalarga tegishli. Aslida, data communication — bu turli fizik kanallar, masalan, fiber-optic kabellar, mis (copper) kabellash va havo orqali digital va elektron signallarni uzatishdir. Physical layer shu kanallarga bevosita bog'liq bo'lgan texnologiyalar va metrikalar uchun standartlarni o'z ichiga oladi — masalan, Bluetooth, NFC va ma'lumot uzatish tezliklari.

### **Data link layer**

Data link layer physical layer allaqachon mavjud bo'lgan tarmoqda ikkita qurilmani bog'lash uchun ishlatiladigan texnologiyalarga tegishli. U data frame'larni boshqaradi — bular data paketlariga o'ralgan digital signallardir. Ma'lumotning flow control va error control'i ko'pincha data link layerning asosiy e'tibor markazida bo'ladi. Ethernet — shu darajadagi standartga misol. Data link layer ko'pincha ikkita sub-layerga bo'linadi: Media Access Control (MAC) layer va Logical Link Control (LLC) layer.

### **Network layer**

Network layer tarqoq tarmoq yoki bir-biriga ulangan bir nechta tarmoqlar (nodelar yoki qurilmalar) bo'ylab routing, forwarding va addressing kabi konsepsiyalar bilan shug'ullanadi. Network layer flow control'ni ham boshqarishi mumkin. Internet bo'ylab Internet Protocol v4 (IPv4) va IPv6 asosiy network layer protokollari sifatida ishlatiladi.

### **Transport layer**

Transport layerning asosiy vazifasi — data paketlarining to'g'ri tartibda, yo'qotishlarsiz yoki xatoliklarsiz yetib borishini, yoki zarur bo'lganda muammosiz tiklanishini ta'minlashdir. Flow control, error control bilan birgalikda, ko'pincha transport layerning asosiy e'tibor markazida bo'ladi. Bu layerda keng qo'llaniladigan protokollarga Transmission Control Protocol (TCP) — deyarli yo'qotishsiz, connection-based protokol — va User Datagram Protocol (UDP) — yo'qotishli, connectionless protokol — kiradi. TCP odatda barcha ma'lumot butun holda yetib borishi kerak bo'lgan holatlarda (masalan, fayl almashish) ishlatiladi, UDP esa barcha paketlarni saqlab qolish unchalik muhim bo'lmagan holatlarda (masalan, video streaming) qo'llaniladi.

### **Session layer**

Session layer bitta session ichidagi ikkita alohida ilova (application) o'rtasidagi tarmoq muvofiqlashuvi (coordination) uchun javobgar. Session bir-biriga o'zaro (one-to-one) application ulanishining boshlanishi va tugashini hamda sinxronizatsiya konfliktlarini boshqaradi. Network File System (NFS) va Server Message Block (SMB) session layerda keng qo'llaniladigan protokollardir.

### **Presentation layer**

Presentation layer asosan ilovalar yuborish va iste'mol qilish uchun ma'lumotning sintaksisi bilan shug'ullanadi. Masalan, Hypertext Markup Language (HTML), JavaScript Object Notation (JSON) va Comma Separated Values (CSV) — barchasi presentation layerda ma'lumot strukturasini tasvirlaydigan modellashtirish tillaridir.

### **Application layer**

Application layer ilovaning o'zi va uning standartlashtirilgan aloqa usullariga tegishli. Masalan, brauzerlar HyperText Transfer Protocol Secure (HTTPS) orqali, HTTP va email klientlar esa POP3 (Post Office Protocol version 3) hamda SMTP (Simple Mail Transfer Protocol) orqali aloqa qilishi mumkin.

OSI modeldan foydalanadigan barcha sistemalar har bir layerni implement qilavermaydi.

## OSI modelda aloqa qanday amalga oshadi?

Open Systems Interconnection (OSI) modelidagi layerlar shunday tuzilganki, ilova qanchalik murakkab bo'lmasin va tagidagi sistemalar qanday bo'lishidan qat'i nazar, u boshqa qurilmadagi boshqa ilova bilan tarmoq orqali aloqa qila oladi. Buning uchun turli xil standartlar va protokollar yuqoridagi yoki pastdagi layer bilan aloqa qilish uchun ishlatiladi. Har bir layer mustaqil bo'lib, faqat o'zidan yuqori va past layer bilan aloqa qilish uchun interfeyslardan xabardor.

Barcha shu layerlar va protokollarni zanjir kabi bog'lash orqali murakkab data communication'lar bitta yuqori darajadagi ilovadan boshqasiga yuborilishi mumkin. Jarayon quyidagicha ishlaydi:

1. Yuboruvchining application layeri data communication'ni bir pastroq layerga uzatadi.
2. Har bir layer ma'lumotni uzatishdan oldin unga o'zining header'lari va addressing'ini qo'shadi.
3. Data communication fizik muhit orqali uzatilguncha layerlar bo'ylab pastga siljib boradi.
4. Muhitning boshqa uchida har bir layer o'sha darajadagi tegishli header'larga muvofiq ma'lumotni qayta ishlaydi.
5. Qabul qiluvchi tomonda ma'lumot layerlar bo'ylab yuqoriga ko'tariladi va oxir-oqibat boshqa uchdagi ilova uni qabul qilguncha bosqichma-bosqich "ochiladi" (unpack qilinadi).

## OSI modeliga alternativalar qanday?

O'tmishda turli xil networking modellari, masalan, Sequenced Packet Exchange/Internet Packet Exchange (SPX/IPX) va Network Basic Input Output System (NetBIOS) ishlatilgan. Bugungi kunda Open Systems Interconnection (OSI) modeliga asosiy alternativa — TCP/IP model.

### **TCP/IP model**

TCP/IP model beshta turli layerdan iborat:

- Physical layer
- Data link layer
- Network layer
- Transport layer
- Application layer

Physical layer, network layer va application layer kabi layerlar OSI modelga to'g'ridan-to'g'ri mos kelayotgandek ko'rinsa-da, bu unchalik ham to'g'ri emas. Aksincha, TCP/IP model internetning strukturasi va protokollariga eng aniq mos keladi.

OSI model ta'lim maqsadlarida networking qanday ishlashini yaxlit nuqtai nazardan tasvirlash uchun mashhur networking modeli bo'lib qolmoqda. Biroq, amaliyotda hozirda TCP/IP model ko'proq qo'llaniladi.

### **Proprietary protokollar va modellar haqida eslatma**

Shuni ta'kidlash kerakki, barcha internetga asoslangan sistemalar va ilovalar TCP/IP model yoki OSI modelga amal qilavermaydi. Xuddi shunday, barcha oflayn asoslangan tarmoq sistemalari va ilovalari ham OSI model yoki boshqa biror modeldan foydalanavermaydi.

OSI va TCP/IP modellarining ikkalasi ham ochiq standartlardir. Ular har kim foydalanishi yoki o'z ehtiyojlariga moslab kengaytirishi mumkin bo'lishi uchun yaratilgan.

Tashkilotlar shuningdek o'zlarining ichki, proprietary standartlarini, jumladan protokollar va modellarni ham loyihalashtiradi — bular closed-source bo'lib, faqat o'z sistemalari ichida foydalanish uchun mo'ljallangan. Ba'zida ular keyinchalik bularni interoperability va keyingi community development uchun ommaga taqdim etishi mumkin. Bunga misol qilib s2n-tls'ni keltirish mumkin — bu dastlab Amazon Web Services (AWS)ning proprietary protokoli bo'lgan, hozirda esa open source hisoblanadigan TLS protokolidir.

## AWS computer networking talablaringizni qanday qondirishi mumkin?

AWS tashkilotlarga tarmoqlangan sistemalar va ilovalarni kamroq to'siq bilan loyihalashtirish, deploy qilish va scale qilishda yordam beradi.

AWS Networking and Content Delivery yo'nalishida keng qamrovli takliflar mavjud. Ular sizning ichki ilovalaringiz va servislaringizni tarmoq operatsiyalarining barcha darajalarida to'ldirish va integratsiya qilish uchun mo'ljallangan. Mana ba'zi misollar:

- **AWS App Mesh** — barcha servislaringiz uchun xavfsiz, application-level networking'ni, o'rnatilgan aloqa monitoring va boshqaruvi bilan ta'minlaydi.
- **Amazon CloudFront** — yuqori performance, xavfsizlik va developer uchun qulaylik maqsadida qurilgan content delivery network (CDN) servisi.
- **AWS Direct Connect** — internetga tegmaydigan, tashkilotingizdan AWS resurslaringizga to'g'ridan-to'g'ri ulanishni taklif qiladi.
- **Elastic Load Balancing (ELB)** — application scalability'ni yaxshilash uchun kiruvchi tarmoq trafigini AWS target'lari bo'ylab taqsimlaydi.
