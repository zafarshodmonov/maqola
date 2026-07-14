# Docker Compose bo'yicha Qo'llanma

Ushbu darslik **Docker Compose**ning asosiy tushunchalarini oddiy bir Python web-ilova yaratish orqali o'rgatadi.

Biz **Flask** freymvorkidan foydalanamiz va **Redis**da saqlanadigan tashrif hisoblagichi (hit counter) yasaymiz. Bu Docker Compose'ning real web-development ssenariylarida qanday qo'llanilishini ko'rsatadi. Python bilan tanish bo'lmasangiz ham, bu yerdagi kontseptsiyalarni tushunish qiyin bo'lmaydi.

## Talablar (Prerequisites)

Boshlashdan oldin quyidagilarga ega bo'lishingiz kerak:

- Docker Compose'ning eng so'nggi versiyasi o'rnatilgan bo'lishi
- Docker qanday ishlashi haqida asosiy tushunchaga ega bo'lish

## 1-qadam: Loyihani sozlash

1. Loyiha uchun papka yarating:

   ```console
   $ mkdir compose-demo
   $ cd compose-demo
   ```

2. Loyiha papkasida `app.py` faylini yarating va quyidagini yozing:

   ```python
   import os
   import redis
   from flask import Flask

   app = Flask(__name__)
   cache = redis.Redis(
       host=os.getenv("REDIS_HOST", "redis"),
       port=int(os.getenv("REDIS_PORT", "6379")),
   )

   @app.route("/")
   def hello():
       count = cache.incr("hits")
       return f"Hello from Docker! I have been seen {count} time(s).\n"
   ```

   Bu ilova Redis'ga ulanish uchun kerakli ma'lumotlarni environment variable'lardan o'qiydi, va agar ular berilmagan bo'lsa, standart (default) qiymatlardan foydalanadi — shu sababli ilova "qutidan chiqarilgandayoq" ishlayveradi.

3. Loyiha papkasida `requirements.txt` faylini yarating va quyidagini yozing:

   ```text
   flask
   redis
   ```

4. `Dockerfile` yarating:

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM python:3.12-alpine  # Python 3.12 image asosida image quradi
   WORKDIR /code  # Working directory'ni `/code` qilib belgilaydi
   ENV FLASK_APP=app.py  # `flask` buyrug'i ishlatadigan environment variable'larni belgilaydi
   ENV FLASK_RUN_HOST=0.0.0.0
   RUN apk add --no-cache gcc musl-dev linux-headers  # `gcc` va boshqa dependency'larni o'rnatadi
   COPY requirements.txt .  # `requirements.txt` faylini nusxalaydi
   RUN pip install -r requirements.txt  # Python dependency'larini o'rnatadi
   COPY . .  # Loyihadagi joriy papka (`.`) mazmunini image ichidagi workdir'ga (`.`) nusxalaydi
   EXPOSE 5000
   CMD ["flask", "run", "--debug"]  # Konteyner uchun standart buyruqni belgilaydi
   ```

   > **Muhim:**
   > Fayl nomi aynan `Dockerfile` bo'lishi, hech qanday kengaytma (extension) bo'lmasligi kerak. Ba'zi tahrirlovchilar (editor) avtomatik ravishda `.txt` qo'shib qo'yishi mumkin — bu build'ni ishlamay qolishiga sabab bo'ladi.

5. Konfiguratsiya qiymatlarini saqlash uchun `.env` faylini yarating:

   ```text
   APP_PORT=8000
   REDIS_HOST=redis
   REDIS_PORT=6379
   ```

   Compose avtomatik ravishda `.env` faylini o'qiydi va bu qiymatlarni `compose.yaml` ichida interpolatsiya qilish uchun mavjud qiladi. Bu misolda foyda unchalik katta ko'rinmasligi mumkin, lekin amaliyotda konfiguratsiyani YAML fayldan alohida saqlash quyidagilarga yordam beradi:
   - Qiymatlarni turli environment'larda YAML'ni tahrirlamasdan o'zgartirish
   - Maxfiy ma'lumotlarni (secrets) version control'ga (masalan, git) tushirib qo'ymaslik
   - Qiymatlarni bir nechta service'larda qayta ishlatish

6. Build context'dan keraksiz fayllarni chiqarib tashlash uchun `.dockerignore` faylini yarating:

   ```text
   .env
   *.pyc
   __pycache__
   redis-data
   ```

   Image quryotganda Docker loyiha papkasidagi barcha narsani daemon'ga yuboradi. `.dockerignore` bo'lmasa, bunga sizning `.env` faylingiz (unda maxfiy ma'lumotlar bo'lishi mumkin) va Python bytecode cache'lari ham kiradi. Ularni chiqarib tashlash build'larni tezlashtiradi va maxfiy qiymatlar tasodifan image layer'iga "yopishib qolishi"ning oldini oladi.

## 2-qadam: Service'larni aniqlash va ishga tushirish

Compose butun ilova stack'ingizni boshqarishni soddalashtiradi — service'lar, network'lar va volume'larni bitta YAML konfiguratsiya faylida boshqarish imkonini beradi.

1. Loyiha papkasida `compose.yaml` faylini yarating va quyidagini joylashtiring:

   ```yaml
   services:
     web:
       build: .
       ports:
         - "${APP_PORT}:5000"
       environment:
         - REDIS_HOST=${REDIS_HOST}
         - REDIS_PORT=${REDIS_PORT}

     redis:
       image: redis:alpine
   ```

   Bu Compose fayli ikkita service'ni aniqlaydi:

   - `web` service'i joriy papkadagi `Dockerfile`dan quriladigan image'dan foydalanadi. U host'dagi `8000`-portni konteynerdagi `5000`-portga bog'laydi — Flask standart holatda shu portda tinglaydi.
   - `redis` service'i Docker Hub registry'sidan tortib olinadigan ochiq **Redis** image'idan foydalanadi.

2. Ilovangizni ishga tushiring:

   ```console
   $ docker compose up
   ```

   Bitta buyruq bilan konfiguratsiya faylingizdagi barcha service'lar yaratiladi va ishga tushiriladi. Compose `web` image'ini quradi, Redis image'ini tortib oladi va ikkala konteynerni ham ishga tushiradi.

3. `http://localhost:8000` manzilini oching. Quyidagini ko'rishingiz kerak:

   ```text
   Hello from Docker! I have been seen 1 time(s).
   ```

   Sahifani yangilang — hisoblagich (counter) har safar oshib boradi.

   Bu minimal sozlash ishlaydi, ammo keyingi qadamlarda hal qilinadigan ikkita muammosi bor:

   - **Startup race (ishga tushish poygasi):** `web` va `redis` bir vaqtda ishga tushadi. Agar Redis hali tayyor bo'lmasa, Flask ilovasi ulana olmay, ishdan chiqadi (crash).
   - **Ma'lumotlar saqlanmasligi:** Agar `docker compose down`, so'ng `docker compose up` buyrug'ini bajarsangiz, hisoblagich nolga qaytadi. `docker compose down` konteynerlarni o'chiradi, ular bilan birga konteynerning yozib bo'ladigan (writable) qatlamiga yozilgan barcha ma'lumotlar ham o'chib ketadi. `docker compose stop` konteynerlarni saqlab qoladi, shu sababli ma'lumotlar ham saqlanadi — lekin production'da konteynerlar muntazam ravishda almashtirib turilgani uchun bunga tayanib bo'lmaydi.

4. Keyingi qadamga o'tishdan oldin stack'ni to'xtating:

   ```console
   $ docker compose down
   ```

## 3-qadam: Health check yordamida startup race'ni tuzatish

Startup race'ni tuzatish uchun Compose `web`ni ishga tushirishdan oldin `redis`ning sog'lom (healthy) ekanligiga ishonch hosil qilishi kerak.

1. `compose.yaml` faylini yangilang:

   ```yaml
   services:
     web:
       build: .
       ports:
         - "${APP_PORT}:5000"
       environment:
         - REDIS_HOST=${REDIS_HOST}
         - REDIS_PORT=${REDIS_PORT}
       depends_on:
         redis:
           condition: service_healthy

     redis:
       image: redis:alpine
       healthcheck:
         test: ["CMD", "redis-cli", "ping"]
         interval: 5s
         timeout: 3s
         retries: 5
         start_period: 10s
   ```

   `healthcheck` bloki Compose'ga Redis'ning tayyorligini qanday tekshirishni aytadi:

   - `test` — Compose konteyner ichida bajaradigan, sog'liqni tekshiruvchi buyruq. `redis-cli ping` Redis'ga ulanadi va `PONG` javobini kutadi — agar shu javob kelsa, konteyner sog'lom hisoblanadi.
   - `start_period` — Redis'ga ishga tushish uchun 10 soniya beradi. Shu vaqt oralig'idagi xatoliklar retry limitiga qo'shilmaydi.
   - `interval` — start_period tugagach, har 5 soniyada tekshiruv o'tkazadi.
   - `timeout` — har bir tekshiruvga javob berish uchun 3 soniya beradi, aks holda xatolik deb hisoblanadi.
   - `retries` — konteynerni "unhealthy" deb belgilashdan oldin nechta ketma-ket xatolikka yo'l qo'yilishini belgilaydi. `interval: 5s` va `retries: 5` bilan Compose voz kechishdan oldin 25 soniyagacha kutadi.

2. Tartib to'g'ri ishlashini tekshirish uchun stack'ni ishga tushiring:

   ```console
   $ docker compose up
   ```

   Quyidagiga o'xshash natija ko'rinishi kerak:

   ```text
   [+] Running 2/2
   ✔ Container compose-demo-redis-1  Healthy                       0.0s
   ```

3. Ilova hali ham ishlayotganini tekshirish uchun `http://localhost:8000` manzilini oching, so'ng keyingi qadamdan oldin stack'ni to'xtating:

   ```console
   $ docker compose down
   ```

## 4-qadam: Jonli yangilanishlar uchun Compose Watch'ni yoqish

Compose Watch bo'lmasa, har bir kod o'zgarishida stack'ni to'xtatish, image'ni qayta qurish va konteynerlarni qayta ishga tushirish kerak bo'ladi. Compose Watch faylni saqlaganingizda o'zgarishlarni avtomatik ravishda ishlab turgan konteyneringizga sinxronlab, shu tsiklni bartaraf qiladi.

1. `compose.yaml` faylidagi `web` service'iga `develop.watch` blokini qo'shing:

   ```yaml
   services:
     web:
       build: .
       ports:
         - "${APP_PORT}:5000"
       environment:
         - REDIS_HOST=${REDIS_HOST}
         - REDIS_PORT=${REDIS_PORT}
       depends_on:
         redis:
           condition: service_healthy
       develop:
         watch:
           - action: sync+restart
             path: .
             target: /code
           - action: rebuild
             path: requirements.txt

     redis:
       image: redis:alpine
       healthcheck:
         test: ["CMD", "redis-cli", "ping"]
         interval: 5s
         timeout: 3s
         retries: 5
         start_period: 10s
   ```

   `watch` bloki ikkita qoidani belgilaydi:
   - `sync+restart` action'i host'dagi loyiha papkangizni (`.`) kuzatib turadi. Fayl o'zgarganda, Compose o'zgargan fayllarni ishlab turgan konteyner ichidagi `/code`ga nusxalaydi, so'ng konteynerni qayta ishga tushiradi. Konteyner allaqachon yangilangan fayllar bilan qayta tushgani uchun, Flask darhol yangi kodni o'qib boshlaydi — qo'lda rebuild yoki restart qilish shart emas.
   - `requirements.txt`dagi `rebuild` action'i yangi dependency qo'shganingizda to'liq image rebuild'ini ishga tushiradi, chunki paketlarni o'rnatish shunchaki fayllarni sinxronlashdan farqli o'laroq, rebuild talab qiladi.

2. Watch yoqilgan holda stack'ni ishga tushiring:

   ```console
   $ docker compose up --watch
   ```

3. Jonli o'zgarish kiriting. `app.py`ni oching va salomlashuv matnini yangilang:

   ```python
   return f"Hello from Compose Watch! I have been seen {count} time(s).\n"
   ```

4. Faylni saqlang. Compose Watch o'zgarishni aniqlaydi va darhol sinxronlaydi:

   ```text
   Syncing service "web" after changes were detected
   ```

5. `http://localhost:8000`ni yangilang. Yangi salomlashuv matni hech qanday restart'siz ko'rinadi va hisoblagich hamon oshib boraveradi.

6. Keyingi qadamdan oldin stack'ni to'xtating:

   ```console
   $ docker compose down
   ```

## 5-qadam: Named volume yordamida ma'lumotlarni saqlab qolish

Har safar stack'ni to'xtatib, qayta ishga tushirganingizda tashrif hisoblagichi nolga qaytadi. Redis ma'lumotlari konteyner ichida saqlanadi, shu sababli konteyner o'chirilganda ular ham yo'qoladi. **Named volume** ma'lumotlarni konteyner hayot tsiklidan tashqarida, host'da saqlash orqali bu muammoni hal qiladi.

1. `compose.yaml` faylini yangilang:

   ```yaml
   services:
     web:
       build: .
       ports:
         - "${APP_PORT}:5000"
       environment:
         - REDIS_HOST=${REDIS_HOST}
         - REDIS_PORT=${REDIS_PORT}
       depends_on:
         redis:
           condition: service_healthy
       develop:
         watch:
           - action: sync+restart
             path: .
             target: /code
           - action: rebuild
             path: requirements.txt

     redis:
       image: redis:alpine
       volumes:
         - redis-data:/data
       healthcheck:
         test: ["CMD", "redis-cli", "ping"]
         interval: 5s
         timeout: 3s
         retries: 5
         start_period: 10s

   volumes:
     redis-data:
   ```

   `redis.volumes` ostidagi `redis-data:/data` yozuvi named volume'ni Redis o'z ma'lumot fayllarini yozadigan `/data` yo'liga ulaydi (mount qiladi). Eng yuqori darajadagi (top-level) `volumes` kaliti uni Docker'da ro'yxatdan o'tkazadi, shu sababli u `compose down` va `compose up` tsikllari orasida saqlanib qoladi.

2. `docker compose up --watch` bilan stack'ni ishga tushiring va hisoblagichni oshirish uchun `http://localhost:8000`ni bir necha marta yangilang.

3. `docker compose down` bilan stack'ni buzing (tear down), so'ng `docker compose up --watch` bilan yana qayta ishga tushiring.

4. `http://localhost:8000`ni oching — hisoblagich to'xtagan joyidan davom etadi.

5. Endi hisoblagichni `docker compose down -v` bilan qaytadan nolga tushiring.

   `-v` flag'i named volume'larni ham konteynerlar bilan birga o'chiradi. Bundan ongli ravishda foydalaning — bu saqlangan ma'lumotlarni butunlay o'chirib tashlaydi.

## 6-qadam: Loyihangizni bir nechta Compose fayllari bilan tashkil qilish

Ilovalar kattalashgan sari, yagona `compose.yaml` faylini boshqarish qiyinlashadi. Eng yuqori darajadagi `include` elementi service'larni bitta ilova doirasida turib, bir nechta faylga bo'lib tashlash imkonini beradi.

Bu, ayniqsa, turli jamoalar stack'ning turli qismlariga egalik qilganda yoki infratuzilma ta'riflarini turli loyihalarda qayta ishlatmoqchi bo'lganingizda foydali.

1. Loyiha papkangizda `infra.yaml` nomli yangi fayl yarating va Redis service'i hamda volume'ini shu faylga ko'chiring:

   ```yaml
    services:
     redis:
       image: redis:alpine
       volumes:
         - redis-data:/data
       healthcheck:
         test: ["CMD", "redis-cli", "ping"]
         interval: 5s
         timeout: 3s
         retries: 5
         start_period: 10s

   volumes:
     redis-data:
   ```

2. `compose.yaml` faylini `infra.yaml`ni include qilish uchun yangilang:

   ```yaml
   include:
      - path: ./infra.yaml
   services:
     web:
       build: .
       ports:
         - "${APP_PORT}:5000"
       environment:
         - REDIS_HOST=${REDIS_HOST}
         - REDIS_PORT=${REDIS_PORT}
       depends_on:
         redis:
           condition: service_healthy
       develop:
         watch:
           - action: sync+restart
             path: .
             target: /code
           - action: rebuild
             path: requirements.txt
   ```

3. Hammasi hali ham ishlashini tekshirish uchun ilovani ishga tushiring:

   ```console
   $ docker compose up --watch
   ```

   Compose ishga tushganda ikkala faylni ham birlashtiradi. `web` service'i hamon `redis`ga nomi orqali murojaat qila oladi, chunki include qilingan barcha service'lar bitta standart network'ni bo'lishadi.

   Bu soddalashtirilgan misol, lekin `include`ning asosiy printsipini va u murakkab ilovalarni sub-Compose fayllariga bo'lib, modullashtirishni qanday osonlashtirishini ko'rsatadi.

4. Keyingi qadamdan oldin stack'ni to'xtating:

   ```console
   $ docker compose down
   ```

## 7-qadam: Ishlab turgan stack'ni ko'zdan kechirish va debug qilish

To'liq sozlangan stack bilan, hech narsani to'xtatmasdan turib konteynerlar ichida nima bo'layotganini kuzatishingiz mumkin. Bu qadam yechilgan konfiguratsiyani ko'zdan kechirish, log'larni oqim (stream) tarzida kuzatish va ishlab turgan konteyner ichida buyruqlarni bajarish uchun asosiy buyruqlarni qamrab oladi.

Stack'ni ishga tushirishdan oldin, Compose `.env` o'zgaruvchilarini to'g'ri yechganini va barcha fayllarni to'g'ri birlashtirganini tekshiring:

```console
$ docker compose config
```

`docker compose config` stack ishlab turishini talab qilmaydi — u faqat sizning fayllaringiz asosida ishlaydi. Natijada e'tibor berish kerak bo'lgan bir nechta narsa bor:

- `${APP_PORT}`, `${REDIS_HOST}`, va `${REDIS_PORT}` `.env` faylidagi qiymatlar bilan almashtirilgan bo'ladi.
- Qisqa formatdagi port yozuvi (`"8000:5000"`) o'zining kanonik maydonlariga (`target`, `published`, `protocol`) kengaytirilgan bo'ladi.
- Standart network va volume nomlari aniq ko'rsatiladi, ular loyiha nomi `compose-demo` bilan boshlanadi (prefiks).
- Natija — `include` orqali kiritilgan barcha fayllar bitta yagona ko'rinishga birlashtirilgan, to'liq yechilgan konfiguratsiya.

`docker compose config`dan istalgan vaqtda, ayniqsa variable substitution'ni debug qilayotganda yoki bir nechta Compose fayl bilan ishlayotganda, Compose aslida nimani qo'llashini tasdiqlash uchun foydalaning.

Endi keyingi buyruqlar uchun terminal bo'sh qolishi uchun stack'ni detached (fon) rejimida ishga tushiring:

```console
$ docker compose up -d
```

### Barcha service'lardan log'larni oqim tarzida kuzatish

```console
$ docker compose logs -f
```

`-f` flag'i log oqimini real vaqtda kuzatib boradi, ikkala konteynerning natijasini rangli (color-coded) service nomi prefikslari bilan bir-biriga aralashtirib ko'rsatadi. `http://localhost:8000`ni bir necha marta yangilang va Flask so'rov (request) log'lari paydo bo'lishini kuzating. Faqat bitta service'ning log'ini kuzatish uchun, uning nomini bering:

```console
$ docker compose logs -f web
```

Log'larni kuzatishni to'xtatish uchun `Ctrl+C` bosing. Konteynerlar ishlashda davom etadi.

### Ishlab turgan konteyner ichida buyruqlarni bajarish

`docker compose exec` yangi konteyner ishga tushirmasdan, allaqachon ishlab turgan konteyner ichida buyruq bajaradi. Bu jonli debug qilishning asosiy vositasi.

#### Environment variable'lar to'g'ri o'rnatilganini tekshirish

```console
$ docker compose exec web env | grep REDIS
```

```text
REDIS_HOST=redis
REDIS_PORT=6379
```

#### `web` konteynerining service nomi orqali Redis'ga ulana olishini tekshirish

```console
$ docker compose exec web python -c "import redis; r = redis.Redis(host='redis'); print(r.ping())"
```

```text
True
```

Bu yerda ilovangiz foydalanadigan xuddi shu `redis` kutubxonasi ishlatiladi, shu sababli `True` javobi service discovery, networking va Redis ulanishining boshdan-oyoq to'g'ri ishlayotganini tasdiqlaydi.

#### Redis'dagi hisoblagichning joriy qiymatini ko'zdan kechirish

```console
$ docker compose exec redis redis-cli GET hits
```

## Keyingi qadamlar

Docker Compose'ni yanada chuqurroq o'rganish uchun quyidagilarni ko'rib chiqishingiz mumkin:

- Compose buyruqlarining to'liq ro'yxati
- Compose fayl reference'i (barcha kalitlar va sozlamalar)
- `docker compose exec`, `docker compose config` kabi buyruqlarni amaliyotda mashq qilish
- Environment variable'larni Compose'da qanday sozlash kerakligi
- Compose ilovangizni qanday paketlab, tarqatish (distribute) mumkinligi

---

### Qisqacha xulosa

| Muammo | Yechim | Compose xususiyati |
|---|---|---|
| `web` va `redis` bir vaqtda ishga tushishi | `redis` sog'lom bo'lguncha kutish | `healthcheck` + `depends_on: condition: service_healthy` |
| Har bir o'zgarishda qo'lda rebuild qilish | Faylni saqlaganda avtomatik sinxronlash | `develop.watch` (`sync+restart`, `rebuild`) |
| `down`dan keyin ma'lumot yo'qolishi | Ma'lumotni host'da saqlash | Named volume (`redis-data:/data`) |
| Bitta katta compose fayl | Faylni bir nechtaga bo'lish | `include` |
| Nima ishlayotganini bilmaslik | Konfiguratsiya va log'larni tekshirish | `docker compose config`, `logs -f`, `exec` |
