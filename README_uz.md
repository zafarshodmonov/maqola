# Model quality estimation (Model sifatini baholash)

Qisqacha mazmun: Ushbu loyihada turli xil validation (tekshirish/tasdiqlash) texnikalari muhokama qilinadi. Model sifatini qanday to'g'ri o'lchash va leak (ma'lumot sizib chiqishi)dan qanday qochish kerakligini ko'rib chiqamiz. Shuningdek, model hyperparameter'larini optimallashtirishning bir nechta usullari va feature selection (belgilarni tanlash)ning turli metodlarini ham muhokama qilamiz.

💡 [Bu yerga bosing](https://new.oprosso.net/p/4cb31ec3f47a4596bc758ea1861fb624) **loyiha bo'yicha fikr-mulohazangizni qoldirish uchun**. Bu anonim so'rovnoma bo'lib, bizning jamoamizga sizning o'quv tajribangizni yaxshilashda yordam beradi. Loyihani tugatgandan so'ng darhol so'rovnomani to'ldirishni tavsiya qilamiz.

## Mundarija

1. [I bob. Kirish so'zi (Preamble)](#chapter-i-preamble)
2. [II bob. Muqaddima (Introduction)](#chapter-ii-introduction)
    2.1. [Bitta fold bilan validation (One fold validation)](#one-fold-validation)
    2.2. [Cross-validation (N Fold Validation)](#cross-validation-n-fold-validation)
    2.3. [Hyperparameter optimallashtirish (Hyperparameter optimization)](#hyperparameter-optimization)
    2.4. [Feature selection (Belgilarni tanlash)](#feature-selection)
3. [III bob. Maqsad (Goal)](#chapter-iii-goal)
4. [IV bob. Ko'rsatmalar (Instructions)](#chapter-iv-instructions)
5. [V bob. Vazifa (Task)](#chapter-v-task)

## I bob. Kirish so'zi (Preamble)

Oldingi loyihalarda biz machine learning (mashinaviy o'qitish) qo'llanilishining ko'plab turli misollarini ko'rib chiqdik va linear regression (chiziqli regressiya) modellarini qurishga chuqur kirib bordik. Lekin bizda har doim tayyorlangan training (o'qitish) va test namunalari bor edi. Ushbu loyihada esa dataset (ma'lumotlar to'plami)ni modelni to'g'ri fit (moslashtirish/o'qitish) qilish uchun qismlarga qanday bo'lish kerakligini muhokama qilamiz.

Validation jarayoni ehtimol modellashtirish pipeline (jarayon zanjiri)ning eng qiyin va muhim qismlaridan biridir. Bir nechta misollarga qarang: tasavvur qiling, siz katta oziq-ovqat retail (chakana savdo) kompaniyasisiz. Va yozning boshidan boshlab, sizga keyingi 3 oy uchun muzqaymoq sotuvini bashorat qilish vazifasi berildi. Siz oxirgi yarim yillik ma'lumotlarni to'pladingiz. Test to'plami sifatida oxirgi mavjud uchta oyni ishlatasiz. Lekin biz hozircha kuzata olmaydigan 3 ta yozgi oy, mavjud ma'lumotlardagi 3 ta bahorgi va 3 ta qishki oy — bularning tabiati bir-biridan farq qiladi. Hamma biladiki, sovuq oylarda muzqaymoq sotuvi kamayadi.

Keling, yanada murakkabroq misolni ko'rib chiqamiz. Bizga bankda kreditlarning defolt (to'lanmay qolishi)ni bashorat qilish kerak. Bu amaliyotda keng tarqalgan vazifa. Agar biz ushbu vazifada training va test namunalarimizni kredit arizasining identifikatori bo'yicha guruhlasak, katta xatoga yo'l qo'yamiz. Nima uchun? Chunki mijoz bankdan bir necha marta kredit so'rashi mumkin. Va shunday bo'lishi mumkinki, mijozning 2020-yildagi krediti training qismida, xuddi shu mijozning 2018-yildagi krediti esa test qismida bo'lib qoladi. Bu shuni anglatadiki, o'qitilgan model mijoz 2018-yildagi kreditni to'lab bo'lganini allaqachon "biladi". Albatta, bu bilim modelga to'g'ridan-to'g'ri kiritilmagan, lekin bu ma'lumot feature'larda saqlanishi juda ehtimoldan xoli emas. Bu esa modelga chalg'ituvchi pattern (naqsh)larni o'rganish imkonini beradi va underfitting (yetarli darajada o'rganmaslik)ga olib keladi.
Bunday holat odatda information leakage (ma'lumot sizib chiqishi) deb ataladi.

Va bu hammasi emas. Eslaysizmi, oldingi loyihada biz modellarimiz uchun maxsus feature'larni tanlagan edik? Xo'sh, agar biz turli xil mumkin bo'lgan subset (kichik to'plam)larni sinab ko'rib, ortiqcha feature'lar bo'lmagan eng яхшisini tanlamoqchi bo'lsak-chi? Yoki loss (yo'qotish funksiyasi)ga regularization qo'shganimizda, unda biz ko'paytiradigan weight (og'irlik) borligini ham payqagan bo'lishingiz mumkin. Bu weight ham modelning performance (samaradorligi)ga ta'sir qiladi. Xo'sh, uni qanday optimallashtirish kerak? Spoyler: bunday ehtiyojlar uchun biz dataset'imizni yana validation qismiga ham bo'lishimiz kerak, ya'ni training, valid va test qismlariga.

Shunday qilib, validation jarayonining asosiy maqsadi — training va testing jarayonini shunday tashkil qilishki, u model production (ishlab chiqarish/real muhit)da ishlayotganda hech qanday xatoga sabab bo'lmasin. Keling, validation texnikalarini ko'rib chiqamiz.

## II bob. Muqaddima (Introduction)

### Bitta fold bilan validation (One fold validation)

Ma'lumotlarimizni ikki qismga — training va test to'plamiga — bo'lish oddiy fikrdek tuyuladi. Lekin buni amalga oshirishning ko'plab yo'llari bor. Birinchi va eng sodda usul — bu train/test to'plamining belgilangan nisbati bilan biror identifikator bo'yicha tasodifiy split (bo'lish) qilishdir. Masalan, dataset namunasining indeksi bo'yicha yoki dataset'imizdagi bitta namunaga mos keladigan foydalanuvchi identifikatori bo'yicha tasodifiy bo'lish. Bu usul ko'p ma'lumotga ega bo'lgan va **ma'lumotlarda vaqt bog'liqligi bo'lmagan** holatlarda keng qo'llaniladi. Bunday holatda test qismi ko'pincha out-of-fold deb ataladi. Bu yerda fold — "dataset'ning bir qismi" degan ma'noni anglatuvchi sinonim. Demak, "out-of-fold" — biz training uchun ishlatgan qismga kirmaydigan namunalarda performance'ni tekshirishimizni bildiradi.

Dataset'ni 2 ta namunaga bo'lishning ikkinchi usuli — ma'lumotlarimizni vaqt yoki sana bo'yicha saralab, oxirgi davrning bir qismini test sifatida olishdir. Yuqoridagi misolga ehtiyot bo'ling. Tabiiyki, bu usul faqat ma'lumotlarda vaqt bog'liqligi mavjud bo'lganda qo'llanilishi mumkin. Bunday holatda bizning bo'lish usulimiz out-of-time deb ataladi.

Amaliyotda ikkita fold — training va testing — yetarli emas. Ushbu bobning davomida biz feature selection va hyperparameter optimallashtirish kabi modellashtirish pipeline'ining qismlarini ko'rib chiqamiz, ular model sifatini baholash uchun maxsus fold talab qiladi, va bu to'plam validation to'plami deb ataladi. Bu training'dan out-of-fold yoki out-of-time strategiyasi yordamida yaratilgan fold bo'lishi mumkin. Muhim narsa shundaki:
* Dataset'ning training qismida biz modelimizni o'qitamiz (train qilamiz).
* Dataset'ning validation qismida biz o'qitilgan modelning sifatini o'lchaymiz va preprocessing (dastlabki ishlov berilgan) ma'lumotlarning turli shartlarini yoki modelning hyperparameter'larini o'zgartirish orqali uning performance'ini sozlaymiz (tune qilamiz).
* Dataset'ning test qismida biz modelimizning yakuniy sifatini o'lchaymiz, bu bizga modelimizning real foydasini tushunish imkonini beradi.

Demak, siz test ma'lumotlarini modellashtirish pipeline'ida yakuniy metrikani o'lchashdan tashqari boshqa hech qayerda ishlata olmaysiz.

Quyida biz training, validation va test'ga bo'lish jarayonini vizualizatsiya qilamiz.

![Classic approach](misc/images/classic_approach.png)

Klassik yondashuv (Classic approach)

![Out-of-time for test part](misc/images/out_of_time_for_test_part.png)

Test qismi uchun Out-of-time

![Out-of-time both for test and valid parts](misc/images/out_of_time_both_and_valid_parts.png)

Ham test, ham valid qismlari uchun Out-of-time

Manba: https://muse.union.edu/dvorakt/train-validate-and-test-with-time-series/

Yuqorida ko'rganimizdek, biz bu usullarni birlashtira olamiz. Lekin agar bizda modellashtirish uchun unchalik ko'p ma'lumot bo'lmasa-chi, overfitting (haddan tashqari moslashish)dan qanday qochish mumkin?

### Cross-validation (N Fold Validation)

Amaliyotda biz model training uchun ko'p ma'lumot to'play olmaydigan ko'plab masalalarni topishimiz mumkin, masalan, ma'lumot to'plash qimmat va murakkab bo'lgan tibbiy masalalar.

Bunday holatlar uchun biz cross-validation sxemasidan foydalanishimiz mumkin. Birinchidan, biz ma'lumotlarimizni N ta fold'ga bo'lamiz. Ikkinchidan, birinchi fold'ni test qismi sifatida olamiz, qolgan fold'larni esa modelimizni o'qitish uchun ishlatamiz. Keyin bu jarayonni keyingi fold uchun takrorlaymiz va hokazo. Nihoyat, barcha fold'lardan metrikalarni yig'ib, model performance'ini baholash uchun o'rtachasini olishimiz kerak. Eng ko'p ishlatiladigan fold'lar soni 3 dan 10 gachadir.

Chuqurroq tushunish uchun quyidagi rasmga qarang:

![Cross-validation](misc/images/grid_search_cross_validation.png)

Manba: https://scikit-learn.org/stable/modules/cross_validation.html

Cross-validation'ning leave-one-out validation sxemasi deb ataladigan alohida holati mavjud. Bu sizning vazifangiz bo'ladi — ushbu sxema uchun ta'rif toping va uning cheklovlari (limitations) hamda kuchli tomonlarini bering.

Sklearn'da cross-validation uchun bir nechta maxsus metod mavjud: K-fold, grouped K-fold, stratified K-fold va TimeSeriesSplit. Keling, ular orasidagi farqlarni tushunish uchun ularga chuqurroq to'xtalamiz.

![K-Fold](misc/images/k_fold.png)

**K-Fold** yuqorida tasvirlangan usulni takrorlaydi. Ko'k rang training to'plamini, qizil rang esa testni bildiradi. Performance'ni olish uchun biz modelimizni 4 ta turli split'da o'qitamiz va baholaymiz, so'ngra o'rtacha ballni olamiz. Muqobil yo'l sifatida, har bir qizil bo'lak uchun tegishli modelning bashoratini eslab qolishimiz mumkin. Agar biz bu bashoratlarni birlashtirsak, out-of-fold predictions (bashoratlar) deb ataladigan vektorga ega bo'lamiz. Shu tariqa, biz performance metrikamizni out-of-fold bashoratlar va haqiqiy label (yorliq)larni funksiyaga uzatish orqali hisoblashimiz mumkin.

Nima uchun biz bu muqobil usulni tilga oldik? Chunki out-of-fold model performance'ini yaxshilash uchun foydali bo'lishi mumkin, lekin bu mavzu ushbu loyiha doirasidan tashqarida. Agar chuqurroq bilishni istasangiz, stacking haqida o'qing.

K-Fold metodining bitta jiddiy kamchiligi bor: ma'lumotlarda umumiy guruhga tegishli kuzatuvlar mavjud bo'lganda. Bu yerda "guruh" — siz ma'lumotlar shu bo'yicha bo'linishi kerak deb hisoblagan har qanday muhim xususiyat bo'lishi mumkin. Bu turli vaqtlardagi mijozning kuzatuvlari yoki siz uzilishlarni aniqlashingiz kerak bo'lgan turli samolyotlarning ID'lari bo'lishi mumkin. Bunday holatlarda biz train-test namunalarini shunday qismlarga bo'lishimiz kerakki, mijoz/samolyot faqat train yoki faqat test namunasida bo'lsin — train va test to'plamlarida mijoz ID'si kesishmasligi kerak.

Bu metod **Group K-Fold** deb ataladi. Avvalgidek, biz namunamizni K-Fold'larga bo'lamiz, lekin buning ustiga uni maxsus parametr bo'yicha ham guruhlaymiz. Quyida "group" ustuni bo'yicha group K-Fold'ning vizualizatsiyasini ko'rishingiz mumkin.

![Group-K-Fold](misc/images/group_k_fold.png)

Cross-validation sxemalari uchun yana bir nechta qiziqarli va muhim yo'nalishlar mavjud. Ular **Stratified K-Fold** va **Stratified Group K-Fold** deb ataladi. Target (maqsad) o'zgaruvchilarini fold'lar bo'yicha stratifikatsiya qilishimiz kerak bo'lgan misollarni keltirish — sizning vazifangiz. Bu metodlarning kuchli va zaif tomonlarini bering.

Endi ma'lumotlarda vaqt bog'liqligi mavjud bo'lgandagi oxirgi cross-validation sxemasini ko'rib chiqamiz. U time series splitting (vaqt qatorlarini bo'lish) deb ataladi. Avvalambor, biz ma'lumotlarimizni sana yoki belgilangan timeline (vaqt chizig'i) bo'yicha saralashimiz kerak. k — split'lar sonini belgilaydi.

![TimeSeriesSplit](misc/images/time_series_split.png)

Birinchi model uchun biz ma'lumotning 1/k qismini training namunasi sifatida, so'ngra yana 1/k qismini test namunasi sifatida olamiz. Ikkinchi model uchun training namunasini ma'lumotning 2/k qismigacha kengaytiramiz va sliding window (siljuvchi oyna)ni keyingi 1/k qismga o'tkazamiz — bu test uchun. Va hokazo. Bu metodda biz k ta emas, k-1 ta model o'qitamiz.

### Hyperparameter optimallashtirish (Hyperparameter optimization)

Ushbu qismda biz hyperparameter optimallashtirishni muhokama qilamiz — bu yaxshiroq performance va kamroq overfitting uchun model parametrlarining eng яxshi kombinatsiyasini topish jarayonidir. Model parametrlarining 2 turi mavjud — internal (ichki) — modelning o'zi bu parametrlarni fitting jarayonida optimallashtiradi, va external (tashqi) — bular fitting jarayonida yangilanmaydi (biz ularni gradient yoki boshqa hech qanday usul bilan yangilamaymiz). Bunday external parametrlar hyperparameter deb ataladi, va biz bu yerda faqat ular haqida gaplashamiz. Ikkala tur uchun ham misollar keltirishga harakat qiling.

Hyperparameter optimallashtirish — bu tsiklik (loop) jarayon. Siz bir yoki bir nechta model parametrini o'zgartirasiz, modelni training to'plamiga fit qilasiz va sifatni validation to'plamida o'lchaysiz, agar metrikalar oshsa — o'sha yo'nalishda davom etasiz, agar oshmasa — ularni o'zgartirishga harakat qilasiz.

Ba'zida aniq mantiq (clear logic) sizga optimallashtirish uchun mos hyperparameter'larni tanlashga yordam beradi. Masalan, aytaylik, bizda polynomial regression bazaviy algoritm sifatida bor. Va train hamda validation to'plamidagi metrikalar orasida katta farq (gap) bor. Shunda biz modelimiz overfit bo'lganini xulosa qilamiz. Aniq mantiq bizga polynomial feature'larning darajasini kamaytirishni tavsiya qiladi — bu holatda ularning soni hyperparameter hisoblanadi. Lekin 5 yoki 10 ta bog'liq bo'lmagan hyperparameter'ning optimal to'plamini qanday topish mumkin?

Afsuski, hyperparameter'larning barcha mazmunli kombinatsiyalarini tekshirishdan boshqa deyarli yaxshiroq usul yo'q. Bu metod Grid Search deb ataladi. Lekin bu juda ko'p vaqt oladi. Agar vaqtimiz cheklangan bo'lsa, biz Randomized Grid Search'dan foydalanishimiz mumkin. Ularning qanday ishlashini tushunish — sizning vazifangizning bir qismi.

Lekin bu ikkala yondashuvning ham bitta kamchiligi bor — ular parametrlar orasidagi bog'liqlikni hisobga olmaydi. Agar biz polynomial feature'larning bir xil darajasi bilan 3 ta model fit qilsak, boshqa hyperparameter'larni o'zgartirsak va yomon performance olsak — biz boshqa darajani sinab ko'rishimiz kerakligi qanchalik ehtimoldan xoli? Buni hal qiluvchi g'oya Bayesian optimization'da qo'llaniladi.

Python'da bu yechimni amalga oshiruvchi ikkita kutubxona mavjud: hyperopt va optuna (optuna yaxshiroq ko'rinadi). Bu yondashuvning tagida qanday matematika yotishini tushuntirish ham sizning vazifangizning bir qismi.

### Feature selection (Belgilarni tanlash)

Modellashtirish jarayonidagi keyingi muhim qadam, bu hyperparameter optimallashtirish kabi ham o'ylanishi mumkin bo'lgan — feature selection. Ko'pincha bizda xom ma'lumot manbalaridan minglab feature'lar bo'ladi, va biz ularning ulkan miqdorini o'zimiz ham generatsiya qila olamiz. Signal beruvchi muhimroq feature'larni qanday topish va shovqinli hamda keraksiz ustunlarni qanday olib tashlash kerakligi — tabiiy savol. Natijada biz nafaqat modelni tezlashtirishimiz, balki uning performance'ini ham oshirishimiz mumkin. Lekin buni qanday amalga oshirish mumkin?

Hyperparameter'lardagi kabi, biz feature'larning barcha mumkin bo'lgan kombinatsiyalarini "kuch bilan" (brute force) tekshirib, optimalini topishimiz mumkin, lekin bu juda ko'p vaqt oladi. Yaxshiyamki, hyperparameter tuning bilan solishtirganda, feature selection uchun ko'plab yondashuvlar mavjud. Ularning barchasini tushunish uchun biror klassifikatsiyadan foydalanish yaxshiroq:

![Feature Selection Methods](misc/images/feature_selection_methods.png)

Manba: https://neptune.ai/blog/feature-selection-methods (bu ro'yxat to'liq emas va klassifikatsiya juda ham mukammal bo'lmasligi mumkin).

Supervised (nazoratli) va unsupervised (nazoratsiz) o'rtasidagi farq machine learning vazifalaridagi bilan bir xil. Wrapper, filter va embedded metodlar o'rtasidagi farqni tushunish — bu loyihada sizning vazifangiz. Iltimos, quyidagilar bilan tanish bo'lishingizga ishonch hosil qiling:
* Barcha unsupervised texnikalar;
* Barcha wrapper metodlar;
* Filterlar:
  * Pearson,
  * Chi2;
* Embedded:
  * Lasso,
  * Ridge.
* Quyidagi metodlar wrapper va filter o'rtasida bir joyda turadi va yuqoridagi rasmda ko'rsatilmagan. Lekin bu metodlar juda tavsiya qilinadi:
  * **permutation importance**;
  * **shap** — https://shap.readthedocs.io/en/latest/.

Amaliyotga o'tishdan oldin ta'kidlamoqchi bo'lgan oxirgi narsa shuki, ham hyperparameter optimallashtirish, ham feature selection cross-validation bilan birlashtirilishi mumkin. Bu jarayonlarni adolatli qilishga va modellarning overfit bo'lishiga yo'l qo'ymaslikka yordam beradi.

## III bob. Maqsad (Goal)

Ushbu vazifaning maqsadi — validation sxemalari, hyperparameter optimallashtirish va feature selection haqida chuqur tushunchaga ega bo'lishdir.

## IV bob. Ko'rsatmalar (Instructions)

"School 21"da qanday o'qish kerak:

- Bu yerda siz erkinlik ko'p bo'lgan noyob o'quv tajribasini topasiz. Sizga vazifa beriladi va siz uni o'zingizga qulay bo'lgan har qanday resurslar yordamida — Internet bo'ladimi yoki GigaChat kabi AI vositalari bo'ladimi — o'zingiz hal qilish yo'lini topishingiz kerak. Faqat ma'lumot sifatiga e'tiborli bo'ling: tekshiring, tanqidiy fikrlang, tahlil qiling va solishtiring.
- Peer-to-peer (P2P) o'qitish — bu tengdoshlar bilan bilim va tajriba almashinuvi bo'lib, unda har kim ham mentor, ham talaba rolini o'ynaydi. Bu yondashuv materialni bir-biridan o'rganish orqali chuqurroq tushunish imkonini beradi.
- Yordam so'rashdan tortinmang: atrofingizda bu yo'lni birinchi marta bosib o'tayotgan tengdoshlaringiz bor. O'z tajribangiz va g'oyalaringizni boshqalar bilan baham ko'ring. Jamoaning so'nggi yangiliklaridan xabardor bo'lish uchun Rocket.Chat'ga qo'shiling.
- Agar siz shunchaki boshqa birovning yechimini nusxalasangiz, o'qishingiz ma'nosiz bo'ladi. Boshqalardan yordam olganingizda, har doim yechim ortidagi "nima uchun", "qanday" va "maqsad"ni to'liq tushunganingizga ishonch hosil qiling. Xato qilishdan qo'rqmang.
- Vazifa imkonsizdek tuyulyaptimi? Tanaffus qiling, tashqariga chiqib toza havodan nafas oling va fikringizni tiniqlashtiring — bu ko'plarga yordam bergan. Balki shundan so'ng yechim o'z-o'zidan xayolingizga keladi.
- O'qish jarayoni natija qadar muhimdir. Bu shunchaki vazifani bajarish emas — bu uni QANDAY hal qilishni tushunish haqida.

Loyiha bilan qanday ishlash kerak:

* Bu loyihani faqat odamlar baholaydi. Fayllaringizni istagancha tashkil qilish va nomlashda erkinsiz.
* Bu yerda va bundan buyon biz faqat Python 3'ni to'g'ri Python versiyasi sifatida ishlatamiz.
* Deep learning algoritmlarini o'qitish uchun [Google Colab](https://colab.research.google.com)dan foydalanib ko'rishingiz mumkin. U bunday vazifalar uchun CPU'dan tezroq bo'lgan GPU'li bepul kernel (Runtime)larni taklif qiladi.
* Standart ushbu loyihaga tatbiq etilmaydi. Biroq, sizdan source code (manba kodi) dizaynida aniq va tuzilgan bo'lishingiz so'raladi.
* Dataset'larni data papkachasida saqlang.

## V bob. Vazifa (Task)

Kaggle.com'dagi masala bilan o'quvimizni davom ettiramiz.
Ushbu bobda biz yuqorida tasvirlangan barcha validation sxemalarini, ba'zi hyperparameter tuning metodlarini va feature selection metodlarini amalga oshiramiz. Training va test namunalarida sifat metrikalarini o'lchaymiz. Overfit bo'lgan modellarni aniqlaymiz va ularni regularize qilamiz. Va native model estimation (baholash) hamda taqqoslash bilan chuqurroq shug'ullanamiz.

1. Kirish qismidagi savollarga javob bering
   1. Leave-one-out nima? Uning cheklovlari va kuchli tomonlarini keltiring.
   2. Grid Search, Randomized Grid Search va Bayesian optimization qanday ishlaydi?
   3. Feature selection metodlarining klassifikatsiyasini tushuntiring. Pearson va Chi2 qanday ishlashini tushuntiring. Lasso qanday ishlashini tushuntiring. Permutation significance nima ekanligini tushuntiring. SHAP bilan tanishib chiqing.

2. Muqaddima — oldingi darsdagi barcha preprocessing'ni bajaring
   1. Barcha ma'lumotlarni o'qing.
   2. Quyidagi feature'larni yarating: 'Elevator', 'HardwoodFloors', 'CatsAllowed', 'DogsAllowed', 'Doorman', 'Dishwasher', 'NoFee', 'LaundryinBuilding', 'FitnessCenter', 'Pre-War', 'LaundryinUnit', 'RoofDeck', 'OutdoorSpace', 'DiningRoom', 'HighSpeedInternet', 'Balcony', 'SwimmingPool', 'LaundryInBuilding', 'NewConstruction', 'Terrace'.

3. Quyidagi metodlarni amalga oshiring:
   1. Ma'lumotlarni test_size parametri (0 dan 1 gacha nisbat) bilan tasodifiy ravishda 2 qismga bo'ling, training va test namunalarini qaytaring.
   2. Ma'lumotlarni validation_size va test_size parametrlari bilan tasodifiy ravishda 3 qismga bo'ling, train, validation va test namunalarini qaytaring.
   3. Ma'lumotlarni date_split parametri bilan 2 qismga bo'ling, date_split parametri bo'yicha bo'lingan train va test namunalarini qaytaring.
   4. Ma'lumotlarni validation_date va test_date parametrlari bilan 3 qismga bo'ling, kiritilgan parametrlar bo'yicha bo'lingan train, validation va test namunalarini qaytaring.
   5. Bo'lish (split) jarayonini determenistik qiling. Bu nimani anglatadi?

4. Quyidagi cross-validation metodlarini amalga oshiring:
   1. K-Fold, bu yerda k — kirish parametri, train va test indekslari ro'yxatini qaytaradi.
   2. Grouped K-Fold, bu yerda k va group_field — kirish parametrlari, train va test indekslari ro'yxatini qaytaradi.
   3. Stratified K-fold, bu yerda k va stratify_field — kirish parametrlari, train va test indekslari ro'yxatini qaytaradi.
   4. Time series split, bu yerda k va date_field — kirish parametrlari, train va test indekslari ro'yxatini qaytaradi.

5. Cross-validation'larni taqqoslash
   1. Yuqorida amalga oshirilgan barcha validation metodlarini dataset'imizga qo'llang. Stratified algoritmni qo'llash uchun target'ni preprocessing qilishingiz kerak.
   2. Sklearn'dagi mos metodlarni qo'llang.
   3. Dataset'ning training qismi uchun sklearn va sizning implementatsiyangiz o'rtasidagi feature taqsimotlarini (distributions) taqqoslang.
   4. Barcha validation sxemalarini taqqoslang. Eng яxshisini tanlang. Tanlovingizni tushuntiring.

6. Feature Selection
   1. Normallashtirilgan (normalized) feature'lar bilan Lasso regression modelini fit qiling. Namunalarni 60/20/20 nisbatda — train/validation/test — 3 qismga bo'lish uchun o'zingiz yaratgan metoddan foydalaning.
   2. Feature'larni model'dan olingan weight koeffitsientlari bo'yicha saralang, top 10 ta feature'ga modelni fit qiling va sifatni taqqoslang.
   3. Feature'dagi nan-ratio (bo'sh qiymatlar nisbati) va korrelyatsiya bo'yicha oddiy feature selection metodini amalga oshiring. Bu metodni feature to'plamiga qo'llang va top 10 ta feature'ni oling, modelni qayta fit qiling va sifatni o'lchang.
   4. Permutation importance metodini amalga oshiring va top 10 ta feature'ni oling, modelni qayta fit qiling va sifatni o'lchang.
   5. Shap'ni import qiling va modelni ham top 10 ta feature'da qayta fit qiling.
   6. Bu metodlarning sifatini turli jihatlar bo'yicha — tezlik, metrikalar va barqarorlik (stability) — taqqoslang.

7. Hyperparameter optimallashtirish
   1. Sklearn'ning ElasticNet modeli uchun alpha va l1_ratio bo'yicha grid search va random search metodlarini amalga oshiring.
   2. Model hyperparameter'larining eng яxshi kombinatsiyasini toping.
   3. Natijaviy modelni fit qiling.
   4. Optuna'ni import qiling va ElasticNet bilan xuddi shu eksperimentni sozlang.
   5. Metrikalarni baholang va yondashuvlarni taqqoslang.
   6. Optuna'ni cross-validation sxemalaridan birida ishga tushiring.

### Topshirish (Submission)

Kodingizni Python JupyterNotebook'da saqlang. Sizning tengdoshingiz uni yuklab, bazaviy yechim bilan taqqoslaydi. Kodingizda barcha majburiy savollarga javoblar bo'lishi kerak. Qo'shimcha vazifa esa ixtiyoriy.

>Loyiha bo'yicha fikr-mulohazangizni [feedback formasi]da(https://forms.yandex.ru/cloud/646b46f7d046882ee5a0b173/) qoldiring.
