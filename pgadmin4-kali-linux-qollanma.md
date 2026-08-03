# PostgreSQL va pgAdmin 4: Kali Linux uchun to'liq qo'llanma

## 1. Kirish

**pgAdmin 4** — PostgreSQL ma'lumotlar bazasini boshqarish uchun eng ko'p ishlatiladigan grafik interfeys (GUI - Graphical User Interface) vositasi. U orqali siz terminalga SQL buyruqlarini yozmasdan ham database yaratish, table (jadval)larni ko'rish, ma'lumotlarni tahrirlash va query (so'rov)larni bajarishingiz mumkin.

Skrinshotingizdan ko'rinishicha, pgAdmin 4 allaqachon o'rnatilgan va ochilgan, lekin chap tomondagi **Servers** bo'limi hali kengaytirilmagan (expand qilinmagan) — ya'ni hech qanday serverga ulanish (connection) hali sozlanmagan. Shu yerdan boshlaymiz.

---

## 2. Kali Linux'da PostgreSQL xizmatini ishga tushirish

pgAdmin 4 — bu faqat interfeys, u PostgreSQL serverining o'zi emas. Shuning uchun avval PostgreSQL server (postgresql service) ishlab turganiga ishonch hosil qilish kerak. Kali Linux'da xizmatlar odatda avtomatik ishga tushmaydi, shuning uchun terminalda quyidagilarni tekshiring:

```bash
# PostgreSQL o'rnatilganini tekshirish
sudo dpkg -l | grep postgresql

# Xizmat holatini tekshirish
sudo systemctl status postgresql

# Agar ishlamayotgan bo'lsa, ishga tushirish
sudo systemctl start postgresql

# Har safar kompyuter yoqilganda avtomatik ishga tushishi uchun
sudo systemctl enable postgresql
```

Agar PostgreSQL umuman o'rnatilmagan bo'lsa:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
```

### `postgres` foydalanuvchisi uchun parol o'rnatish

PostgreSQL o'rnatilganda avtomatik ravishda `postgres` degan superuser (barcha huquqlarga ega foydalanuvchi) yaratiladi. pgAdmin orqali ulanish uchun unga parol belgilash kerak:

```bash
sudo -u postgres psql
```

`psql` konsoliga kirgach, quyidagini yozing:

```sql
ALTER USER postgres WITH PASSWORD 'sizning_parolingiz';
```

So'ngra `\q` yozib chiqing.

---

## 3. pgAdmin 4'da serverga ulanish (Register Server)

Endi pgAdmin 4'ga qaytamiz. Skrinshotdagi holatdan davom etamiz:

1. Chap paneldagi **Servers** yozuvi ustiga **o'ng tugma (right-click)** bilan bosing.
2. Chiqqan menyudan **Register → Server...** ni tanlang.
3. Ochilgan oynada ikkita tab (bo'lim) bo'ladi:

   **General tab:**
   - **Name** — bu ulanishga o'zingiz qo'yadigan nom (masalan: `Local Kali PostgreSQL`). Bu shunchaki belgi, PostgreSQL'ning haqiqiy nomi emas.

   **Connection tab:**
   - **Host name/address** — `localhost` yoki `127.0.0.1`
   - **Port** — `5432` (PostgreSQL default porti)
   - **Maintenance database** — `postgres`
   - **Username** — `postgres`
   - **Password** — yuqorida o'rnatgan parolingiz
   - **Save password?** — belgilab qo'ysangiz, har safar parol kiritmaysiz

4. **Save** tugmasini bosing.

Agar hammasi to'g'ri bo'lsa, chap panelda server nomi paydo bo'ladi va uni ochib (expand qilib) quyidagi tuzilmani ko'rasiz:

```
Servers
 └── Local Kali PostgreSQL
      └── Databases
           ├── postgres
           └── ...
      └── Login/Group Roles
      └── Tablespaces
```

---

## 4. pgAdmin 4 interfeysi bilan tanishish

pgAdmin 4 uchta asosiy qismdan iborat:

- **Browser paneli (chap tomon)** — daraxtsimon (tree) tuzilma: Servers → Databases → Schemas → Tables va hokazo. Bu yerda barcha obyektlarni ko'rasiz va boshqarasiz.
- **Ish maydoni (o'ng tomon, markaz)** — tanlangan obyekt haqida statistika, property (xususiyat)lar, yoki Query Tool ochilganda SQL yozish maydoni.
- **Yuqori menyu paneli** — `File`, `Object`, `Tools`, `Edit`, `View`, `Window`, `Help` — obyektlar ustida amallar bajarish uchun.

Skrinshotdagi yuqori qatordagi ikonkalar (Filter, Grid, Copy, Search, va h.k.) — bu **Object Explorer**ning boshqaruv elementlari bo'lib, tree ichida qidiruv va filtrlash uchun ishlatiladi.

---

## 5. Yangi database (ma'lumotlar bazasi) yaratish

1. Chap panelda server ichidagi **Databases** bo'limiga o'ng tugma bilan bosing.
2. **Create → Database...** ni tanlang.
3. **Database** maydoniga nom bering (masalan: `mening_bazam`).
4. **Owner** sifatida `postgres` (yoki boshqa foydalanuvchi)ni tanlang.
5. **Save** tugmasini bosing.

Endi yangi database chap panelda paydo bo'ladi.

---

## 6. Table (jadval) yaratish

1. Yaratgan database'ingizni oching: `mening_bazam → Schemas → public → Tables`.
2. **Tables** ustiga o'ng tugma bilan bosib, **Create → Table...** ni tanlang.
3. **General** tab'da table nomini kiriting (masalan: `foydalanuvchilar`).
4. **Columns** tab'ga o'ting va **+** tugmasi orqali ustunlar (column)lar qo'shing. Har bir ustun uchun:
   - **Name** — ustun nomi (masalan: `id`, `ism`, `email`)
   - **Data type** — ma'lumot turi (`integer`, `varchar`, `text`, `boolean`, `timestamp` va h.k.)
   - **Not NULL** — agar ustun bo'sh bo'lishi mumkin bo'lmasa, belgilang
   - **Primary key** — `id` kabi ustunlar uchun odatda belgilanadi

5. **Save** tugmasini bosing.

> **Eslatma:** Table'ni GUI orqali emas, balki to'g'ridan-to'g'ri SQL yozib ham yaratishingiz mumkin — buning uchun keyingi bo'limdagi **Query Tool**dan foydalaning. Ko'pchilik professional dasturchilar SQL yozishni afzal ko'radi, chunki u aniqroq va tez-tez takrorlanadigan (repeatable) skript sifatida saqlanishi mumkin.

---

## 7. Query Tool — SQL buyruqlarini bajarish

Bu pgAdmin 4'ning eng ko'p ishlatiladigan qismi.

1. Database'ni tanlang (masalan: `mening_bazam`).
2. Yuqori menyudan **Tools → Query Tool** ni tanlang (yoki `Alt+Shift+Q` klaviatura tugmalari, yoxud skrinshotdagi yuqori panel ikonkalaridan birini bosing).
3. Ochilgan oynaga SQL kodini yozasiz, masalan:

```sql
-- Table yaratish
CREATE TABLE foydalanuvchilar (
    id SERIAL PRIMARY KEY,
    ism VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    yaratilgan_vaqt TIMESTAMP DEFAULT NOW()
);

-- Ma'lumot qo'shish
INSERT INTO foydalanuvchilar (ism, email) VALUES
('Zafar', 'zafar@example.com'),
('Dilnoza', 'dilnoza@example.com');

-- Ma'lumotlarni ko'rish
SELECT * FROM foydalanuvchilar;
```

4. Kodni bajarish uchun **F5** tugmasini bosing, yoki yuqoridagi ▶ (Play) ikonkasini bosing.
5. Natija pastki qismda **Data Output** jadvalida chiqadi.

---

## 8. Ma'lumotlarni ko'rish va tahrirlash (GUI orqali)

SQL yozmasdan ham ma'lumotlarni ko'rish mumkin:

1. Chap panelda table'ni toping (`Tables → foydalanuvchilar`).
2. Table ustiga o'ng tugma bosib, **View/Edit Data → All Rows** ni tanlang.
3. Excel'ga o'xshash jadval ochiladi — bu yerda katakchalarni bosib to'g'ridan-to'g'ri tahrirlashingiz, yangi qator qo'shishingiz (pastdagi **+** tugmasi) yoki o'chirishingiz mumkin.
4. O'zgarishlarni saqlash uchun **Save (disket ikonkasi)** tugmasini bosing.

---

## 9. Backup va Restore (zaxira nusxa olish va tiklash)

### Backup olish

1. Database ustiga o'ng tugma bosing → **Backup...**
2. Fayl saqlanadigan joyni (**Filename**) tanlang.
3. **Format** — odatda `Custom` tanlanadi (bu keyinchalik qisman restore qilish imkonini beradi).
4. **Backup** tugmasini bosing.

### Restore qilish

1. Yangi (bo'sh) database yarating yoki mavjud database'ni tanlang.
2. Database ustiga o'ng tugma bosing → **Restore...**
3. Avval saqlangan backup faylini tanlang.
4. **Restore** tugmasini bosing.

---

## 10. Foydalanuvchi va rollarni (Login/Group Roles) boshqarish

PostgreSQL'da "user" tushunchasi aslida **role** deb ataladi.

1. Server ichidagi **Login/Group Roles** bo'limiga o'ng tugma bosing.
2. **Create → Login/Group Role...** ni tanlang.
3. **General** tab — rol nomi (masalan: `dasturchi1`).
4. **Definition** tab — parol o'rnatish.
5. **Privileges** tab — `Can login?`, `Superuser?`, `Create databases?` kabi huquqlarni yoqish/o'chirish.
6. **Save**.

Bu orqali har xil foydalanuvchilarga turlicha huquqlar berib, xavfsizlikni ta'minlashingiz mumkin.

---

## 11. Tez-tez ishlatiladigan amallar — qisqacha jadval

| Amal | Qanday qilinadi |
|---|---|
| Serverga ulanish | `Servers` → o'ng tugma → `Register → Server` |
| Database yaratish | `Databases` → o'ng tugma → `Create → Database` |
| Table yaratish (GUI) | `Tables` → o'ng tugma → `Create → Table` |
| SQL yozish | `Tools → Query Tool` yoki `Alt+Shift+Q` |
| Ma'lumotni ko'rish/tahrirlash | Table → o'ng tugma → `View/Edit Data → All Rows` |
| Backup olish | Database → o'ng tugma → `Backup...` |
| Restore qilish | Database → o'ng tugma → `Restore...` |
| Rol/foydalanuvchi yaratish | `Login/Group Roles` → o'ng tugma → `Create` |

---

## 12. Keyingi qadam

Bu qo'llanma sizga pgAdmin 4'ning asosiy funksiyalarini o'rgatdi. Keyingi bosqichda quyidagilarni chuqurroq o'rganishingiz mumkin:

- **JOIN**lar bilan bir necha table'dan birgalikda ma'lumot olish
- **Index**lar orqali query tezligini oshirish
- **View** va **Function/Trigger** yaratish
- pgAdmin 4'ning **ERD Tool** (Entity-Relationship Diagram) orqali database sxemasini vizual chizish

Agar xohlasangiz, keyingi qo'llanmani aynan shu mavzulardan biriga bag'ishlab tayyorlab beraman.
