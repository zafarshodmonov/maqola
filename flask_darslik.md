# Flask bilan Web Dasturlash Asoslari

Flask — Python tilida yozilgan **micro web framework** bo'lib, kichik va sodda web dasturlardan tortib, katta va murakkab tizimlargacha qurish uchun ishlatiladi. "Micro" so'zi Flask'ning yadrosi minimal ekanligini bildiradi — u sizga faqat routing, request/response, va templating kabi asosiy narsalarni beradi, qolgan hamma narsani (database, forma validatsiyasi, autentifikatsiya) siz o'zingiz extension sifatida qo'shasiz. Bu Django'dan farqli o'laroq, Flask'ga to'liq erkinlik va nazorat beradi.

---

## Mundarija

1. O'rnatish va muhitni sozlash
2. Birinchi Flask dasturi
3. Routing (marshrutlash)
4. HTTP metodlar: GET va POST
5. Dinamik URL'lar
6. Templates (Jinja2)
7. Static fayllar (CSS, JS, rasmlar)
8. Formalar bilan ishlash
9. Redirect va `url_for()`
10. Session va cookie
11. Ma'lumotlar bazasi (SQLite + Flask-SQLAlchemy)
12. Xatoliklarni boshqarish
13. Blueprint — loyihani bo'laklarga bo'lish
14. Amaliy loyiha: To-Do List ilovasi

---

## 1. O'rnatish va muhitni sozlash

Avval virtual muhit (virtual environment) yaratish tavsiya etiladi — bu loyihangiz kutubxonalarini tizimning boshqa Python loyihalaridan izolyatsiya qiladi.

```bash
# Virtual muhit yaratish
python -m venv venv

# Faollashtirish (Linux/Mac)
source venv/bin/activate

# Faollashtirish (Windows)
venv\Scripts\activate

# Flask'ni o'rnatish
pip install flask
```

O'rnatilganini tekshirish:

```bash
python -c "import flask; print(flask.__version__)"
```

---

## 2. Birinchi Flask dasturi

`app.py` faylini yarating:

```python
from flask import Flask

# Flask ilovasini yaratish
app = Flask(__name__)

# Route (marshrut) — brauzer qaysi manzilga kirganda nima ko'rsatilishini belgilaydi
@app.route('/')
def home():
    return "Salom, Flask dunyosi!"

if __name__ == '__main__':
    app.run(debug=True)
```

Ishga tushirish:

```bash
python app.py
```

Brauzerda `http://127.0.0.1:5000` manzilini oching — "Salom, Flask dunyosi!" matnini ko'rasiz.

**Muhim tushunchalar:**
- `Flask(__name__)` — ilova obyektini yaratadi, `__name__` orqali Flask joriy modulning joylashuvini bilib, static va template papkalarini topadi.
- `debug=True` — kod o'zgarganda serverni avtomatik qayta yuklaydi va xatolik yuz berganda batafsil traceback ko'rsatadi. **Production'da hech qachon `debug=True` qoldirmang** — bu xavfsizlik zaifligiga olib keladi.
- `@app.route('/')` — bu **decorator**, ya'ni pastdagi funksiyani ma'lum URL manziliga bog'laydi.

---

## 3. Routing (marshrutlash)

Har bir `@app.route()` bitta URL yo'lini bitta funksiyaga (view function) bog'laydi:

```python
@app.route('/')
def home():
    return "Bosh sahifa"

@app.route('/about')
def about():
    return "Biz haqimizda"

@app.route('/contact')
def contact():
    return "Aloqa uchun"
```

Endi `/about` yoki `/contact` manzillariga kirsangiz, mos funksiya ishga tushadi.

---

## 4. HTTP metodlar: GET va POST

Odatda `@app.route()` faqat GET so'rovlarini qabul qiladi. Boshqa metodlarni ruxsat berish uchun `methods` parametrini ko'rsatish kerak:

```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form.get('username')
        return f"Xush kelibsiz, {username}!"
    return "Login formasi shu yerda bo'ladi (GET so'rov)"
```

- **GET** — ma'lumot so'rash uchun (masalan, sahifani ochish).
- **POST** — serverga ma'lumot yuborish uchun (masalan, forma to'ldirish).
- `request.form` — POST orqali yuborilgan forma ma'lumotlariga kirish imkonini beradi.
- `request.args` — URL query parametrlariga kirish imkonini beradi (masalan, `/search?q=flask` dagi `q`).

---

## 5. Dinamik URL'lar

URL ichida o'zgaruvchan qismlarni belgilash mumkin — bu **path parameter** deb ataladi:

```python
@app.route('/user/<username>')
def show_user(username):
    return f"Foydalanuvchi profili: {username}"

@app.route('/post/<int:post_id>')
def show_post(post_id):
    return f"Post raqami: {post_id}"
```

Converter turlari:

| Converter | Tavsif |
|---|---|
| `string` | Standart, `/` bo'lmagan har qanday matn |
| `int` | Faqat butun sonlar |
| `float` | O'nlik kasr sonlar |
| `path` | `/` belgisini ham qabul qiladigan matn |

Masalan: `http://127.0.0.1:5000/user/zafar` manziliga kirsangiz, "Foydalanuvchi profili: zafar" chiqadi.

---

## 6. Templates (Jinja2)

HTML kodni Python funksiyalar ichida yozish noqulay va chalkash. Shuning uchun Flask **Jinja2** template engine'ini ishlatadi. HTML fayllar `templates/` papkasida saqlanadi.

**Loyiha strukturasi:**
```
myapp/
├── app.py
├── templates/
│   └── index.html
└── static/
    └── style.css
```

**`app.py`:**
```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html', name='Zafar', items=['C', 'C++', 'Python'])

if __name__ == '__main__':
    app.run(debug=True)
```

**`templates/index.html`:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Bosh sahifa</title>
</head>
<body>
    <h1>Salom, {{ name }}!</h1>

    <ul>
        {% for item in items %}
            <li>{{ item }}</li>
        {% endfor %}
    </ul>

    {% if name == 'Zafar' %}
        <p>Siz admin sifatida ro'yxatdan o'tgansiz.</p>
    {% else %}
        <p>Oddiy foydalanuvchi.</p>
    {% endif %}
</body>
</html>
```

**Jinja2 sintaksisi:**
- `{{ ... }}` — o'zgaruvchini chop etish uchun.
- `{% ... %}` — mantiqiy amallar uchun (`if`, `for`, `block` va h.k.).
- Template'lar avtomatik ravishda HTML'ni **escape** qiladi — bu XSS hujumlaridan himoyalaydi.

### Template Inheritance (meros olish)

Ko'p sahifalarda umumiy header/footer bo'lsa, `base.html` yaratib, undan meros olish mumkin:

**`templates/base.html`:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Sayt{% endblock %}</title>
</head>
<body>
    <nav>Navigatsiya menyu</nav>
    {% block content %}{% endblock %}
</body>
</html>
```

**`templates/index.html`:**
```html
{% extends "base.html" %}

{% block title %}Bosh sahifa{% endblock %}

{% block content %}
    <h1>Bu bosh sahifa kontenti</h1>
{% endblock %}
```

---

## 7. Static fayllar (CSS, JS, rasmlar)

CSS, JavaScript va rasm fayllar `static/` papkasida saqlanadi va `url_for('static', filename='...')` orqali chaqiriladi:

**`static/style.css`:**
```css
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
}
```

**Template ichida ulash:**
```html
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
```

`url_for()` funksiyasini ishlatish tavsiya etiladi, chunki agar loyiha boshqa serverga ko'chirilsa yoki URL prefiksi o'zgarsa, yo'llar avtomatik to'g'ri hisoblanadi.

---

## 8. Formalar bilan ishlash

**`templates/form.html`:**
```html
<form method="POST" action="{{ url_for('submit') }}">
    <input type="text" name="username" placeholder="Ismingiz">
    <input type="submit" value="Yuborish">
</form>
```

**`app.py`:**
```python
@app.route('/form')
def form():
    return render_template('form.html')

@app.route('/submit', methods=['POST'])
def submit():
    username = request.form['username']
    return f"Rahmat, {username}!"
```

**Validatsiya haqida eslatma:** Kichik loyihalarda `request.form` bilan qo'lda tekshirish yetarli, lekin murakkabroq formalar uchun **Flask-WTF** kutubxonasidan foydalanish tavsiya etiladi — u avtomatik validatsiya, CSRF himoya va xatolik xabarlarini beradi.

---

## 9. Redirect va `url_for()`

Foydalanuvchini bir sahifadan boshqasiga yo'naltirish uchun `redirect()` ishlatiladi, `url_for()` esa funksiya nomi orqali URL yaratadi (bu URL'larni qo'lda yozishdan ko'ra ancha ishonchli):

```python
from flask import redirect, url_for

@app.route('/old-page')
def old_page():
    return redirect(url_for('home'))

@app.route('/')
def home():
    return "Bosh sahifa"
```

`url_for('home')` — `home` nomli funksiyaga bog'langan URL'ni (`/`) qaytaradi. Agar keyinchalik `/` manzilini `/main` ga o'zgartirsangiz, `url_for()` ishlatilgan barcha joylarda avtomatik to'g'rilanadi.

---

## 10. Session va cookie

Session — foydalanuvchi haqidagi ma'lumotni so'rovlar orasida saqlash imkonini beradi (masalan, login holatini eslab qolish):

```python
from flask import session

app.secret_key = 'juda-maxfiy-kalit'  # session'larni shifrlash uchun majburiy

@app.route('/login', methods=['POST'])
def login():
    session['username'] = request.form['username']
    return redirect(url_for('profile'))

@app.route('/profile')
def profile():
    if 'username' in session:
        return f"Xush kelibsiz, {session['username']}!"
    return redirect(url_for('login'))

@app.route('/logout')
def logout():
    session.pop('username', None)
    return redirect(url_for('home'))
```

**Muhim:** `app.secret_key` session ma'lumotlarini xavfsiz imzolash (sign) uchun ishlatiladi. Uni hech qachon kodga ochiq yozib, GitHub'ga yuklamang — environment variable orqali saqlang:

```python
import os
app.secret_key = os.environ.get('SECRET_KEY', 'dev-kalit')
```

---

## 11. Ma'lumotlar bazasi (SQLite + Flask-SQLAlchemy)

Flask'da ma'lumotlar bazasi bilan ishlashning eng qulay yo'li — **Flask-SQLAlchemy** ORM (Object-Relational Mapping) kutubxonasi. U SQL yozish o'rniga Python klasslari orqali jadval bilan ishlash imkonini beradi.

```bash
pip install flask-sqlalchemy
```

```python
from flask import Flask, request, redirect, url_for, render_template
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///todo.db'
db = SQLAlchemy(app)

# Model — jadval strukturasini belgilaydi
class Task(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    done = db.Column(db.Boolean, default=False)

    def __repr__(self):
        return f'<Task {self.title}>'

# Jadvallarni yaratish (bir marta ishga tushiriladi)
with app.app_context():
    db.create_all()

@app.route('/tasks')
def tasks():
    all_tasks = Task.query.all()
    return render_template('tasks.html', tasks=all_tasks)

@app.route('/tasks/add', methods=['POST'])
def add_task():
    new_task = Task(title=request.form['title'])
    db.session.add(new_task)
    db.session.commit()
    return redirect(url_for('tasks'))

@app.route('/tasks/delete/<int:task_id>')
def delete_task(task_id):
    task = Task.query.get_or_404(task_id)
    db.session.delete(task)
    db.session.commit()
    return redirect(url_for('tasks'))
```

**Asosiy CRUD amallari:**

| Amal | Kod |
|---|---|
| Yaratish (Create) | `db.session.add(obj)` + `db.session.commit()` |
| O'qish (Read) | `Model.query.all()`, `Model.query.get(id)`, `Model.query.filter_by(...)` |
| Yangilash (Update) | Obyekt maydonini o'zgartirib, `db.session.commit()` |
| O'chirish (Delete) | `db.session.delete(obj)` + `db.session.commit()` |

---

## 12. Xatoliklarni boshqarish

Maxsus xatolik sahifalarini (masalan, 404) sozlash mumkin:

```python
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404

@app.errorhandler(500)
def server_error(e):
    return "Serverda xatolik yuz berdi", 500
```

`abort()` funksiyasi orqali qo'lda xatolik chaqirish mumkin:

```python
from flask import abort

@app.route('/admin')
def admin():
    if not is_admin_user():
        abort(403)  # Forbidden
    return "Admin panel"
```

---

## 13. Blueprint — loyihani bo'laklarga bo'lish

Loyiha kattalashganda barcha route'larni bitta `app.py` faylida saqlash noqulay bo'ladi. **Blueprint** loyihani mantiqiy modullarga ajratish imkonini beradi:

**`auth.py`:**
```python
from flask import Blueprint

auth_bp = Blueprint('auth', __name__)

@auth_bp.route('/login')
def login():
    return "Login sahifasi"

@auth_bp.route('/register')
def register():
    return "Ro'yxatdan o'tish sahifasi"
```

**`app.py`:**
```python
from flask import Flask
from auth import auth_bp

app = Flask(__name__)
app.register_blueprint(auth_bp, url_prefix='/auth')
```

Endi `auth.py` ichidagi route'lar `/auth/login` va `/auth/register` manzillarida ishlaydi. Bu katta loyihalarda kodni tartibli va boshqarish oson qiladi.

---

## 14. Amaliy loyiha: To-Do List ilovasi

Yuqoridagi bilimlarni birlashtirib, to'liq ishlaydigan mini-loyiha yasaymiz.

**Loyiha strukturasi:**
```
todo_app/
├── app.py
└── templates/
    └── tasks.html
```

**`app.py`:**
```python
from flask import Flask, request, redirect, url_for, render_template
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///todo.db'
db = SQLAlchemy(app)


class Task(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(200), nullable=False)
    done = db.Column(db.Boolean, default=False)


with app.app_context():
    db.create_all()


@app.route('/')
def index():
    tasks = Task.query.all()
    return render_template('tasks.html', tasks=tasks)


@app.route('/add', methods=['POST'])
def add():
    title = request.form.get('title')
    if title:
        db.session.add(Task(title=title))
        db.session.commit()
    return redirect(url_for('index'))


@app.route('/toggle/<int:task_id>')
def toggle(task_id):
    task = Task.query.get_or_404(task_id)
    task.done = not task.done
    db.session.commit()
    return redirect(url_for('index'))


@app.route('/delete/<int:task_id>')
def delete(task_id):
    task = Task.query.get_or_404(task_id)
    db.session.delete(task)
    db.session.commit()
    return redirect(url_for('index'))


if __name__ == '__main__':
    app.run(debug=True)
```

**`templates/tasks.html`:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>To-Do List</title>
</head>
<body>
    <h1>Vazifalar ro'yxati</h1>

    <form method="POST" action="{{ url_for('add') }}">
        <input type="text" name="title" placeholder="Yangi vazifa" required>
        <input type="submit" value="Qo'shish">
    </form>

    <ul>
        {% for task in tasks %}
            <li>
                <span style="text-decoration: {{ 'line-through' if task.done else 'none' }}">
                    {{ task.title }}
                </span>
                <a href="{{ url_for('toggle', task_id=task.id) }}">✔</a>
                <a href="{{ url_for('delete', task_id=task.id) }}">✘</a>
            </li>
        {% endfor %}
    </ul>
</body>
</html>
```

Ishga tushirish:

```bash
python app.py
```

`http://127.0.0.1:5000` manziliga kirib, vazifa qo'shish, belgilash va o'chirish imkoniyatlarini sinab ko'rishingiz mumkin.

---

## Xulosa va keyingi qadamlar

Ushbu darslikda Flask'ning asosiy tushunchalari — routing, templates, formalar, session va ma'lumotlar bazasi — ko'rib chiqildi. Keyingi bosqichda o'rganish tavsiya etiladigan mavzular:

- **Flask-WTF** — formalar va CSRF himoyasi uchun
- **Flask-Login** — foydalanuvchi autentifikatsiyasi uchun
- **Flask-Migrate** — ma'lumotlar bazasi migratsiyalari uchun
- **REST API** yaratish (Flask yoki Flask-RESTful bilan)
- **Deployment** — Gunicorn + Nginx yordamida production serverga chiqarish

Flask'ning kuchi uning soddaligida — kichik boshlab, kerak bo'lgan sari extension qo'shib borish mumkin.

