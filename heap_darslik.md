# HEAP MA'LUMOT TUZILMASI — TO'LIQ DARSLIK

## Mundarija

1. [Kirish](#1-kirish)
2. [Heap nima?](#2-heap-nima)
3. [Heap turlari: Min-Heap va Max-Heap](#3-heap-turlari-min-heap-va-max-heap)
4. [Heap qanday ifodalanadi (Array asosida)](#4-heap-qanday-ifodalanadi-array-asosida)
5. [Asosiy operatsiyalar — nazariy tushuntirish](#5-asosiy-operatsiyalar--nazariy-tushuntirish)
6. [Heap qurish (Build Heap)](#6-heap-qurish-build-heap)
7. [Heap Sort algoritmi](#7-heap-sort-algoritmi)
8. [Vaqt va xotira murakkabligi](#8-vaqt-va-xotira-murakkabligi)
9. [Heapning qo'llanilish sohalari](#9-heapning-qollanilish-sohalari)
10. [Python'da amalga oshirish](#10-pythonda-amalga-oshirish)
11. [C tilida amalga oshirish](#11-c-tilida-amalga-oshirish)
12. [C++ tilida amalga oshirish](#12-c-tilida-amalga-oshirish)
13. [Mashqlar](#13-mashqlar)
14. [Xulosa](#14-xulosa)

---

## 1. Kirish

Dasturlashda ma'lumotlarni saqlash va ular ustida tez amallar bajarish katta ahamiyatga ega. Ba'zi masalalarda bizga eng katta yoki eng kichik elementni tezda topish, uni o'chirish va yangi element qo'shish imkoniyati kerak bo'ladi. Aynan shu masalani yechish uchun **Heap** (uyum) deb ataluvchi ma'lumot tuzilmasi yaratilgan.

Heap — bu maxsus turdagi **to'liq ikkilik daraxt (complete binary tree)** bo'lib, u ustida eng katta (yoki eng kichik) elementni doimiy ravishda `O(1)` vaqtda topish, va uni qo'shish/o'chirish amallarini `O(log n)` vaqtda bajarish mumkin.

Heap tushunchasini chuqur o'zlashtirish uchun quyidagi mavzularni bilish foydali:
- Massivlar (arrays)
- Ikkilik daraxtlar (binary trees)
- Rekursiya
- Algoritmlarning vaqt murakkabligi (Big-O notatsiyasi)

---

## 2. Heap nima?

**Heap** — bu quyidagi ikkita shartni qanoatlantiruvchi ikkilik daraxt:

### 2.1. Struktura sharti (Shape Property)

Heap har doim **to'liq ikkilik daraxt** bo'lishi kerak. Bu degani:
- Daraxtning barcha qatorlari (level) oxirgi qatordan tashqari to'liq to'ldirilgan bo'lishi kerak.
- Oxirgi qatordagi tugunlar esa **chapdan o'ngga** qarab, bo'sh joy qoldirmasdan to'ldirilishi shart.

Bu xususiyat heapni oddiy massiv (array) yordamida, hech qanday pointer yoki qo'shimcha xotirasiz ifodalash imkonini beradi — bu heapning eng katta afzalliklaridan biri.

### 2.2. Heap sharti (Heap Property)

Har bir tugun (node) bilan uning bolalari (children) o'rtasida ma'lum bir tartib munosabati saqlanishi kerak:

- **Max-Heap**da: har bir ota-tugun qiymati o'z bolalari qiymatidan **katta yoki teng** bo'lishi kerak.
- **Min-Heap**da: har bir ota-tugun qiymati o'z bolalari qiymatidan **kichik yoki teng** bo'lishi kerak.

**Muhim izoh:** Heap — bu **saralangan** tuzilma emas! Faqat ota va bola o'rtasidagi munosabat kafolatlanadi, lekin masalan, chap va o'ng bola o'rtasida hech qanday tartib talab qilinmaydi. Shu sababli heapni "yarim tartiblangan daraxt" deb ham atash mumkin.

### 2.3. Vizual misol (Max-Heap)

```
              50
            /    \
          30      40
         /  \    /  \
       10   20  35   15
```

Bu yerda har bir ota-tugun o'z bolalaridan katta: `50 > 30, 40`; `30 > 10, 20`; `40 > 35, 15`. Ammo `30` va `40` o'rtasida (ikkalasi ham 50ning bolasi) tartib talab qilinmaydi.

---

## 3. Heap turlari: Min-Heap va Max-Heap

| Xususiyat | Min-Heap | Max-Heap |
|---|---|---|
| Ildiz (root) qiymati | Eng kichik element | Eng katta element |
| Ota-bola munosabati | ota ≤ bola | ota ≥ bola |
| Qo'llanilishi | Dijkstra algoritmi, eng kichikni topish | Priority Queue, eng kattani topish |

Ikkala turi ham bir xil mantiqqa asoslangan — faqat solishtirish shartining yo'nalishi teskari.

---

## 4. Heap qanday ifodalanadi (Array asosida)

Heapning eng go'zal jihati shundaki, uni pointerlarsiz, oddiy **bitta massiv** yordamida to'liq ifodalash mumkin. Buning siri — to'liq ikkilik daraxt xususiyatida.

Massivda `i` indeksdagi tugun uchun (0-indeksdan boshlab hisoblaganda):

```
Chap bola indeksi   = 2*i + 1
O'ng bola indeksi   = 2*i + 2
Ota-tugun indeksi   = (i - 1) / 2   (butun songa yaxlitlab)
```

### Misol

Max-Heap: `[50, 30, 40, 10, 20, 35, 15]`

| Indeks | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|---|
| Qiymat | 50 | 30 | 40 | 10 | 20 | 35 | 15 |

Daraxt ko'rinishida bu xuddi yuqoridagi rasmga to'g'ri keladi. Masalan, indeks `1` (qiymati 30) uchun:
- Chap bola: `2*1+1 = 3` → qiymati 10
- O'ng bola: `2*1+2 = 4` → qiymati 20
- Ota: `(1-1)/2 = 0` → qiymati 50

Array asosidagi bu ifodalash tufayli heap juda kam xotira sarflaydi va **cache-friendly** (protsessor keshi bilan yaxshi ishlaydi), chunki barcha elementlar xotirada ketma-ket joylashgan.

---

## 5. Asosiy operatsiyalar — nazariy tushuntirish

Heap ustida bajariladigan asosiy amallar quyidagilar:

1. **Peek / Find-Max (yoki Find-Min)** — ildizdagi elementni ko'rish. `O(1)`
2. **Insert** — yangi element qo'shish. `O(log n)`
3. **Extract-Max (yoki Extract-Min)** — ildizni o'chirish va qaytarish. `O(log n)`
4. **Heapify (Sift-Down / Bubble-Down)** — tartibni tiklash. `O(log n)`
5. **Build-Heap** — tartiblanmagan massivdan heap qurish. `O(n)`

### 5.1. Insert (Qo'shish) — "Sift-Up" jarayoni

Yangi elementni heapga qo'shish uchun quyidagi qadamlar bajariladi:

1. Yangi elementni massivning **oxiriga** qo'shamiz (bu to'liq daraxt xususiyatini saqlaydi).
2. Endi bu element heap shartini buzgan bo'lishi mumkin — uni ota-tugun bilan solishtiramiz.
3. Agar yangi element ota-tugundan katta bo'lsa (max-heap uchun), ular joyini almashtiramiz.
4. Bu jarayonni element to'g'ri joyga kelguncha yoki ildizga yetguncha davom ettiramiz.

Bu jarayon **"sift-up"** yoki **"bubble-up"** deb ataladi, chunki element daraxt bo'ylab yuqoriga ko'tariladi.

**Qadamma-qadam misol** (Max-Heap): `[50, 30, 40, 10, 20]` ga `45` qo'shamiz.

```
Qadam 1: [50, 30, 40, 10, 20, 45]   — 45 ni oxiriga qo'shdik
Qadam 2: 45 ning otasi = (5-1)/2 = 2 -> qiymati 40. 45 > 40, almashtiramiz
Qadam 3: [50, 30, 45, 10, 20, 40]
Qadam 4: 45 ning yangi otasi = (2-1)/2 = 0 -> qiymati 50. 45 < 50, to'xtaymiz
```

Natija: `[50, 30, 45, 10, 20, 40]` — heap sharti qayta tiklandi.

### 5.2. Extract-Max/Min (O'chirish) — "Sift-Down" jarayoni

Ildizdagi (eng katta yoki eng kichik) elementni olib tashlash quyidagicha bajariladi:

1. Ildiz qiymatini eslab qolamiz (bu qaytariladigan natija).
2. Massivning **oxirgi** elementini ildiz o'rniga qo'yamiz.
3. Massiv o'lchamini bittaga kamaytiramiz (oxirgi joy bo'shadi).
4. Endi yangi ildiz heap shartini buzgan bo'lishi mumkin — uni bolalari bilan solishtiramiz.
5. Ikki boladan kattarog'i (max-heap uchun) bilan joy almashtiramiz.
6. Bu jarayonni tugun o'z o'rnini topguncha, ya'ni ikkala bolasidan ham katta bo'lguncha yoki bargga (leaf) yetguncha davom ettiramiz.

Bu jarayon **"sift-down"** yoki **"heapify"** deb ataladi.

**Qadamma-qadam misol** (Max-Heap): `[50, 30, 45, 10, 20, 40]` dan max ni chiqaramiz.

```
Qadam 1: Natija = 50 (eslab qolamiz)
Qadam 2: Oxirgi element (40) ni ildizga qo'yamiz: [40, 30, 45, 10, 20]
Qadam 3: 40 ning bolalari: chap=30 (idx1), o'ng=45 (idx2). Kattasi = 45
Qadam 4: 40 < 45, joy almashtiramiz: [45, 30, 40, 10, 20]
Qadam 5: 40 (endi idx2) bolalarini tekshiramiz -> bolalari yo'q (barg), to'xtaymiz
```

Natija: `50` qaytarildi, qolgan heap: `[45, 30, 40, 10, 20]`

---

## 6. Heap qurish (Build Heap)

Agar bizda tartiblanmagan `n` ta elementdan iborat massiv bo'lsa va undan heap qurmoqchi bo'lsak, buni ikki xil usulda qilish mumkin:

### Usul 1: Har birini birma-bir qo'shish
Har bir elementni bo'sh heapga `insert` qilib boramiz. Bu usul `O(n log n)` vaqt talab qiladi.

### Usul 2: Samarali Build-Heap (tavsiya etiladi)
Bu usul massivning **oxirgi ota-tugunidan** boshlab, orqaga qarab har bir tugunga `sift-down` (heapify) amalini qo'llaydi:

```
oxirgi_ota_indeks = n/2 - 1
i = oxirgi_ota_indeks dan 0 gacha kamaytirib:
    sift_down(massiv, i)
```

**Nima uchun oxirgi ota-tugundan boshlanadi?** Chunki barglar (leaf node)lar allaqachon heap shartini qanoatlantiradi (ularning bolalari yo'q), shuning uchun ularni tekshirishning hojati yo'q.

Bu usul, ajablanarli tarzda, `O(n)` vaqtda ishlaydi (nazariy jihatdan `O(n log n)` ga o'xshab ko'rinsada, matematik tahlil buni `O(n)` ekanini isbotlaydi, chunki daraxtning pastki qatorlaridagi ko'p sonli tugunlar juda kam sift-down qadamini talab qiladi).

---

## 7. Heap Sort algoritmi

Heap ma'lumot tuzilmasining eng mashhur qo'llanilishlaridan biri — **Heap Sort** saralash algoritmi. U `O(n log n)` vaqtda ishlaydi va **qo'shimcha xotira talab qilmaydi** (in-place sort).

### Algoritm qadamlari:

1. Berilgan massivdan Max-Heap quramiz (`Build-Heap`, `O(n)`).
2. Ildizdagi (eng katta) elementni massivning oxiridagi element bilan almashtiramiz.
3. Heap o'lchamini bittaga kamaytiramiz (endi oxirgi element "saralangan" hisoblanadi).
4. Yangi ildizga `sift-down` qo'llab, heap shartini tiklaymiz.
5. 2-4 qadamlarni heap bo'sh qolguncha takrorlaymiz.

Natijada massiv o'sish tartibida saralanadi (Max-Heap ishlatilganda).

### Vizual tushuntirish

```
Boshlang'ich:     [50, 30, 45, 10, 20, 40]
1-qadam:  50 <-> 40 almashtiramiz -> [40, 30, 45, 10, 20, 50], heap_size=5
          sift-down(40) -> [45, 30, 40, 10, 20 | 50]
2-qadam:  45 <-> 20 almashtiramiz -> [20, 30, 40, 10, 45 | 50], heap_size=4
          sift-down(20) -> [40, 30, 20, 10 | 45, 50]
... va hokazo, oxir-oqibat: [10, 20, 30, 40, 45, 50]
```

Heap Sort ustunligi: eng yomon holatda ham `O(n log n)` kafolatlanadi (Quick Sort'dan farqli o'laroq), lekin amalda konstant koeffitsienti kattaroq bo'lgani uchun ko'pincha Quick Sort'dan sekinroq ishlaydi.

---

## 8. Vaqt va xotira murakkabligi

| Operatsiya | Vaqt murakkabligi | Izoh |
|---|---|---|
| Peek (Find-Max/Min) | O(1) | Ildiz doim indeks 0'da |
| Insert | O(log n) | Daraxt balandligi bo'ylab sift-up |
| Extract-Max/Min | O(log n) | Daraxt balandligi bo'ylab sift-down |
| Build-Heap (n elementdan) | O(n) | Matematik jihatdan isbotlangan |
| Heap Sort | O(n log n) | Build-Heap + n marta extract |
| Berilgan elementni qidirish | O(n) | Heap qidiruv uchun optimallashtirilmagan! |
| Xotira | O(n) | Faqat massiv, qo'shimcha pointer kerak emas |

**Muhim eslatma:** Heap tasodifiy elementni qidirish uchun mo'ljallanmagan — bu uning kuchli tomoni emas. Agar sizga tez-tez ixtiyoriy elementni qidirish kerak bo'lsa, boshqa tuzilmalar (masalan, Balanced BST yoki Hash Table) ko'proq mos keladi.

---

## 9. Heapning qo'llanilish sohalari

1. **Priority Queue (ustuvorlik navbati)** — heap priority queue'ning eng samarali implementatsiyasi hisoblanadi. Operatsion tizimlarda vazifalarni ustuvorlik bo'yicha bajarish uchun ishlatiladi.
2. **Dijkstra algoritmi** — grafda eng qisqa yo'lni topishda Min-Heap yordamida keyingi eng yaqin tugunni tezda tanlash uchun.
3. **Prim algoritmi** — minimal yoyilgan daraxt (MST) qurishda.
4. **Heap Sort** — yuqorida ko'rib chiqilgan saralash algoritmi.
5. **K ta eng katta/kichik elementni topish** — masalan, `n` ta elementdan eng katta `k` tasini `O(n log k)` vaqtda topish.
6. **Median topish** (Median Finder) — ikkita heap (Min-Heap va Max-Heap) yordamida oqim ma'lumotlarida medianani real vaqtda hisoblash.
7. **Huffman kodlash** — ma'lumotlarni siqishda (data compression) belgilarning chastotasiga qarab daraxt qurishda.
8. **Event-driven simulyatsiyalar** — voqealarni vaqt bo'yicha tartiblab bajarish uchun.

---

## 10. Python'da amalga oshirish

Python'da heap bilan ishlashning ikki yo'li bor: standart `heapq` modulidan foydalanish (tavsiya etiladi) yoki heapni noldan yozish (o'rganish uchun foydali).

### 10.1. Standart `heapq` moduli

Python'ning o'rnatilgan `heapq` moduli faqat **Min-Heap**ni qo'llab-quvvatlaydi.

```python
import heapq

# Bo'sh ro'yxatdan heap yaratamiz
heap = []

# Elementlarni qo'shish -> O(log n)
heapq.heappush(heap, 50)
heapq.heappush(heap, 30)
heapq.heappush(heap, 40)
heapq.heappush(heap, 10)
heapq.heappush(heap, 20)

print(heap)  # [10, 20, 40, 50, 30] -- Min-Heap tartibida (ildiz = eng kichik)

# Eng kichik elementni ko'rish -> O(1)
print(heap[0])  # 10

# Eng kichik elementni chiqarib olish -> O(log n)
smallest = heapq.heappop(heap)
print(smallest)  # 10
print(heap)      # [20, 30, 40, 50]

# Tayyor ro'yxatdan tezda heap qurish -> O(n)
data = [50, 30, 40, 10, 20, 35, 15]
heapq.heapify(data)
print(data)  # Min-Heap tartibiga keltirilgan massiv
```

### 10.2. Max-Heap hosil qilish (heapq orqali)

`heapq` faqat Min-Heap qo'llab-quvvatlagani uchun, Max-Heap kerak bo'lsa, elementlarning **manfiysini** saqlash odatiy trik hisoblanadi:

```python
import heapq

max_heap = []
values = [50, 30, 40, 10, 20]

for v in values:
    heapq.heappush(max_heap, -v)   # manfiy qiymat sifatida qo'shamiz

# Eng katta elementni olish uchun yana manfiylaymiz
largest = -heapq.heappop(max_heap)
print(largest)  # 50
```

### 10.3. `heapq`ning foydali funksiyalari

```python
import heapq

data = [50, 30, 40, 10, 20, 35, 15]

# n ta eng katta elementni topish
print(heapq.nlargest(3, data))   # [50, 40, 35]

# n ta eng kichik elementni topish
print(heapq.nsmallest(3, data))  # [10, 15, 20]
```

### 10.4. Heapni noldan (from scratch) yozish

Heap ichki mexanizmini chuqur tushunish uchun uni o'zimiz yozib ko'ramiz. Quyida to'liq **Max-Heap** klassi keltirilgan:

```python
class MaxHeap:
    def __init__(self):
        self.heap = []

    def _parent(self, i):
        return (i - 1) // 2

    def _left_child(self, i):
        return 2 * i + 1

    def _right_child(self, i):
        return 2 * i + 2

    def peek(self):
        """Eng katta elementni ko'rish, O(1)"""
        if not self.heap:
            raise IndexError("Heap bo'sh")
        return self.heap[0]

    def insert(self, value):
        """Yangi element qo'shish, O(log n)"""
        self.heap.append(value)
        self._sift_up(len(self.heap) - 1)

    def _sift_up(self, i):
        while i > 0 and self.heap[i] > self.heap[self._parent(i)]:
            parent_idx = self._parent(i)
            self.heap[i], self.heap[parent_idx] = self.heap[parent_idx], self.heap[i]
            i = parent_idx

    def extract_max(self):
        """Eng katta elementni chiqarib olish, O(log n)"""
        if not self.heap:
            raise IndexError("Heap bo'sh")

        max_value = self.heap[0]
        last_value = self.heap.pop()  # oxirgi elementni olib tashlaymiz

        if self.heap:
            self.heap[0] = last_value
            self._sift_down(0)

        return max_value

    def _sift_down(self, i):
        size = len(self.heap)
        while True:
            left = self._left_child(i)
            right = self._right_child(i)
            largest = i

            if left < size and self.heap[left] > self.heap[largest]:
                largest = left
            if right < size and self.heap[right] > self.heap[largest]:
                largest = right

            if largest == i:
                break  # heap sharti bajarildi, to'xtaymiz

            self.heap[i], self.heap[largest] = self.heap[largest], self.heap[i]
            i = largest

    def build_heap(self, array):
        """Tartiblanmagan massivdan heap qurish, O(n)"""
        self.heap = array[:]
        n = len(self.heap)
        for i in range(n // 2 - 1, -1, -1):
            self._sift_down(i)

    def size(self):
        return len(self.heap)

    def is_empty(self):
        return len(self.heap) == 0


# --- Sinov ---
if __name__ == "__main__":
    mh = MaxHeap()
    for val in [50, 30, 40, 10, 20, 45]:
        mh.insert(val)

    print("Heap ichki massiv:", mh.heap)
    print("Eng katta element:", mh.peek())

    print("\nElementlarni kamayish tartibida chiqarish:")
    while not mh.is_empty():
        print(mh.extract_max(), end=" ")
```

**Kutilgan natija:**
```
Heap ichki massiv: [50, 30, 45, 10, 20, 40]
Eng katta element: 50

Elementlarni kamayish tartibida chiqarish:
50 45 40 30 20 10
```

### 10.5. Python'da Heap Sort implementatsiyasi

```python
def heap_sort(array):
    heap = MaxHeap()
    heap.build_heap(array)

    result = []
    while not heap.is_empty():
        result.append(heap.extract_max())

    result.reverse()  # o'sish tartibida qaytarish uchun
    return result


print(heap_sort([50, 30, 40, 10, 20, 35, 15]))
# [10, 15, 20, 30, 35, 40, 50]
```

---

## 11. C tilida amalga oshirish

C tilida heapni odatda struct va massiv (dinamik xotira bilan) yordamida amalga oshiramiz. Quyida to'liq **Max-Heap** implementatsiyasi keltirilgan.

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int *data;      // heap elementlarini saqlovchi massiv
    int size;       // hozirgi elementlar soni
    int capacity;   // massivning maksimal sig'imi
} MaxHeap;

/* Heapni yaratish */
MaxHeap* create_heap(int capacity) {
    MaxHeap *heap = (MaxHeap*)malloc(sizeof(MaxHeap));
    heap->data = (int*)malloc(capacity * sizeof(int));
    heap->size = 0;
    heap->capacity = capacity;
    return heap;
}

/* Yordamchi funksiyalar: indekslarni hisoblash */
int parent(int i)      { return (i - 1) / 2; }
int left_child(int i)  { return 2 * i + 1; }
int right_child(int i) { return 2 * i + 2; }

void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

/* Elementni yuqoriga ko'tarish (sift-up) */
void sift_up(MaxHeap *heap, int i) {
    while (i > 0 && heap->data[i] > heap->data[parent(i)]) {
        swap(&heap->data[i], &heap->data[parent(i)]);
        i = parent(i);
    }
}

/* Elementni pastga tushirish (sift-down) */
void sift_down(MaxHeap *heap, int i) {
    int largest = i;
    int left = left_child(i);
    int right = right_child(i);

    if (left < heap->size && heap->data[left] > heap->data[largest])
        largest = left;

    if (right < heap->size && heap->data[right] > heap->data[largest])
        largest = right;

    if (largest != i) {
        swap(&heap->data[i], &heap->data[largest]);
        sift_down(heap, largest);   // rekursiv chaqiruv
    }
}

/* Yangi element qo'shish, O(log n) */
void insert(MaxHeap *heap, int value) {
    if (heap->size == heap->capacity) {
        printf("Heap to'lgan, o'lcham kengaytirilmoqda...\n");
        heap->capacity *= 2;
        heap->data = (int*)realloc(heap->data, heap->capacity * sizeof(int));
    }

    heap->data[heap->size] = value;
    heap->size++;
    sift_up(heap, heap->size - 1);
}

/* Eng katta elementni ko'rish, O(1) */
int peek(MaxHeap *heap) {
    if (heap->size == 0) {
        fprintf(stderr, "Xato: heap bo'sh\n");
        exit(1);
    }
    return heap->data[0];
}

/* Eng katta elementni chiqarib olish, O(log n) */
int extract_max(MaxHeap *heap) {
    if (heap->size == 0) {
        fprintf(stderr, "Xato: heap bo'sh\n");
        exit(1);
    }

    int max_value = heap->data[0];
    heap->data[0] = heap->data[heap->size - 1];
    heap->size--;
    sift_down(heap, 0);

    return max_value;
}

/* Tartiblanmagan massivdan heap qurish, O(n) */
void build_heap(MaxHeap *heap, int *array, int n) {
    for (int i = 0; i < n; i++) {
        heap->data[i] = array[i];
    }
    heap->size = n;

    for (int i = n / 2 - 1; i >= 0; i--) {
        sift_down(heap, i);
    }
}

void free_heap(MaxHeap *heap) {
    free(heap->data);
    free(heap);
}

/* --- Sinov --- */
int main(void) {
    MaxHeap *heap = create_heap(10);

    int values[] = {50, 30, 40, 10, 20, 45};
    int n = sizeof(values) / sizeof(values[0]);

    for (int i = 0; i < n; i++) {
        insert(heap, values[i]);
    }

    printf("Heap ichki massiv: ");
    for (int i = 0; i < heap->size; i++) {
        printf("%d ", heap->data[i]);
    }
    printf("\n");

    printf("Eng katta element: %d\n", peek(heap));

    printf("Elementlarni kamayish tartibida chiqarish: ");
    while (heap->size > 0) {
        printf("%d ", extract_max(heap));
    }
    printf("\n");

    free_heap(heap);
    return 0;
}
```

**Kompilyatsiya va ishga tushirish:**
```bash
gcc -o heap_demo heap_demo.c
./heap_demo
```

**Kutilgan natija:**
```
Heap ichki massiv: 50 30 45 10 20 40
Eng katta element: 50
Elementlarni kamayish tartibida chiqarish: 50 45 40 30 20 10
```

### 11.1. C tilida Heap Sort

```c
#include <stdio.h>

void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

/* n -- joriy heap o'lchami, i -- sift-down boshlanadigan indeks */
void heapify(int arr[], int n, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;

    if (left < n && arr[left] > arr[largest])
        largest = left;

    if (right < n && arr[right] > arr[largest])
        largest = right;

    if (largest != i) {
        swap(&arr[i], &arr[largest]);
        heapify(arr, n, largest);
    }
}

void heap_sort(int arr[], int n) {
    /* 1-qadam: Max-Heap qurish */
    for (int i = n / 2 - 1; i >= 0; i--) {
        heapify(arr, n, i);
    }

    /* 2-qadam: elementlarni birma-bir chiqarib, oxiriga qo'yish */
    for (int i = n - 1; i > 0; i--) {
        swap(&arr[0], &arr[i]);   // ildizni oxiriga olib boramiz
        heapify(arr, i, 0);       // qolgan (kichraygan) heapni tuzatamiz
    }
}

int main(void) {
    int arr[] = {50, 30, 40, 10, 20, 35, 15};
    int n = sizeof(arr) / sizeof(arr[0]);

    heap_sort(arr, n);

    printf("Saralangan massiv: ");
    for (int i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    return 0;
}
```

---

## 12. C++ tilida amalga oshirish

C++ da heap bilan ishlashning ikki yo'li bor: standart kutubxonadagi `std::priority_queue` (tavsiya etiladi) yoki `<algorithm>` dagi heap funksiyalari, yoki heapni noldan klass sifatida yozish.

### 12.1. `std::priority_queue` — tayyor yechim

`std::priority_queue` ichki tomondan aynan heap tuzilmasi asosida ishlaydi va standart holatda **Max-Heap** hisoblanadi.

```cpp
#include <iostream>
#include <queue>
#include <vector>

int main() {
    // Standart holatda Max-Heap
    std::priority_queue<int> max_heap;

    max_heap.push(50);
    max_heap.push(30);
    max_heap.push(40);
    max_heap.push(10);
    max_heap.push(20);

    std::cout << "Eng katta element: " << max_heap.top() << "\n";  // 50

    std::cout << "Kamayish tartibida chiqarish: ";
    while (!max_heap.empty()) {
        std::cout << max_heap.top() << " ";
        max_heap.pop();
    }
    std::cout << "\n";

    return 0;
}
```

### 12.2. Min-Heap qurish (`std::priority_queue` orqali)

Min-Heap yaratish uchun uchinchi shablon parametrini `std::greater<int>` sifatida beramiz:

```cpp
#include <iostream>
#include <queue>
#include <vector>
#include <functional>   // std::greater uchun

int main() {
    std::priority_queue<int, std::vector<int>, std::greater<int>> min_heap;

    min_heap.push(50);
    min_heap.push(30);
    min_heap.push(40);
    min_heap.push(10);

    std::cout << "Eng kichik element: " << min_heap.top() << "\n";  // 10

    return 0;
}
```

### 12.3. `<algorithm>` kutubxonasidagi heap funksiyalari

C++ STL oddiy massiv/vektor ustida heap amallarini bajaruvchi funksiyalarni ham taqdim etadi:

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> data = {50, 30, 40, 10, 20, 35, 15};

    // Vektorni Max-Heap tartibiga keltirish, O(n)
    std::make_heap(data.begin(), data.end());

    std::cout << "Ildiz (eng katta): " << data.front() << "\n";  // 50

    // Yangi element qo'shish
    data.push_back(60);
    std::push_heap(data.begin(), data.end());  // O(log n)

    // Eng katta elementni o'chirish
    std::pop_heap(data.begin(), data.end());   // eng kattani oxiriga olib boradi
    data.pop_back();                            // uni vektordan olib tashlaydi

    // Heap Sort qilish uchun
    std::sort_heap(data.begin(), data.end());  // O(n log n)

    std::cout << "Saralangan: ";
    for (int v : data) std::cout << v << " ";
    std::cout << "\n";

    return 0;
}
```

### 12.4. Heapni noldan klass sifatida yozish (to'liq misol)

Endi heap mexanizmini chuqur tushunish uchun uni o'zimiz C++ klassi ko'rinishida yozamiz:

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>

class MaxHeap {
private:
    std::vector<int> data;

    int parent(int i)     const { return (i - 1) / 2; }
    int leftChild(int i)  const { return 2 * i + 1; }
    int rightChild(int i) const { return 2 * i + 2; }

    void siftUp(int i) {
        while (i > 0 && data[i] > data[parent(i)]) {
            std::swap(data[i], data[parent(i)]);
            i = parent(i);
        }
    }

    void siftDown(int i) {
        int size = data.size();
        while (true) {
            int largest = i;
            int left = leftChild(i);
            int right = rightChild(i);

            if (left < size && data[left] > data[largest])
                largest = left;
            if (right < size && data[right] > data[largest])
                largest = right;

            if (largest == i) break;

            std::swap(data[i], data[largest]);
            i = largest;
        }
    }

public:
    // Yangi element qo'shish, O(log n)
    void insert(int value) {
        data.push_back(value);
        siftUp(data.size() - 1);
    }

    // Eng katta elementni ko'rish, O(1)
    int peek() const {
        if (data.empty()) throw std::runtime_error("Heap bo'sh");
        return data[0];
    }

    // Eng katta elementni chiqarib olish, O(log n)
    int extractMax() {
        if (data.empty()) throw std::runtime_error("Heap bo'sh");

        int maxValue = data[0];
        data[0] = data.back();
        data.pop_back();

        if (!data.empty()) {
            siftDown(0);
        }

        return maxValue;
    }

    // Tartiblanmagan massivdan heap qurish, O(n)
    void buildHeap(const std::vector<int>& array) {
        data = array;
        int n = data.size();
        for (int i = n / 2 - 1; i >= 0; i--) {
            siftDown(i);
        }
    }

    bool isEmpty() const { return data.empty(); }
    size_t size()  const { return data.size(); }
};

int main() {
    MaxHeap heap;

    for (int val : {50, 30, 40, 10, 20, 45}) {
        heap.insert(val);
    }

    std::cout << "Eng katta element: " << heap.peek() << "\n";

    std::cout << "Kamayish tartibida chiqarish: ";
    while (!heap.isEmpty()) {
        std::cout << heap.extractMax() << " ";
    }
    std::cout << "\n";

    return 0;
}
```

**Kompilyatsiya va ishga tushirish:**
```bash
g++ -std=c++17 -o heap_demo heap_demo.cpp
./heap_demo
```

**Kutilgan natija:**
```
Eng katta element: 50
Kamayish tartibida chiqarish: 50 45 40 30 20 10
```

### 12.5. C++ da priority_queue yordamida maxsus solishtirish (custom comparator)

Amaliy loyihalarda ko'pincha oddiy sonlar emas, balki murakkab obyektlarni ustuvorlik bo'yicha saqlash kerak bo'ladi. Bunda maxsus comparator ishlatiladi:

```cpp
#include <iostream>
#include <queue>
#include <vector>
#include <string>

struct Task {
    std::string name;
    int priority;
};

// Comparator: kichik priority raqami -> yuqori ustuvorlik (Min-Heap kabi ishlaydi)
struct CompareTask {
    bool operator()(const Task& a, const Task& b) {
        return a.priority > b.priority;  // teskari solishtirish
    }
};

int main() {
    std::priority_queue<Task, std::vector<Task>, CompareTask> taskQueue;

    taskQueue.push({"Email yozish", 3});
    taskQueue.push({"Xatolikni tuzatish", 1});
    taskQueue.push({"Kod ko'rib chiqish", 2});

    std::cout << "Bajarilish tartibi:\n";
    while (!taskQueue.empty()) {
        Task t = taskQueue.top();
        std::cout << "- " << t.name << " (ustuvorlik: " << t.priority << ")\n";
        taskQueue.pop();
    }

    return 0;
}
```

Bu misol real loyihalarda (masalan, operatsion tizim vazifalar navbati) heap qanday qo'llanilishini ko'rsatadi.

---

## 13. Mashqlar

O'rganganlaringizni mustahkamlash uchun quyidagi mashqlarni bajarib ko'ring:

1. **Min-Heap yozing** — yuqoridagi Max-Heap kodlarini Min-Heap'ga o'zgartiring (Python, C va C++ da).
2. **K ta eng katta elementni topish** — berilgan massivdan `k` ta eng katta elementni `O(n log k)` vaqtda topuvchi funksiya yozing (kichik o'lchamdagi Min-Heap yordamida).
3. **Median Finder** — sonlar oqimidan (stream) real vaqtda medianani hisoblovchi tuzilma yozing (ikkita heap: Max-Heap va Min-Heap yordamida).
4. **Heapni tekshirish** — berilgan massiv to'g'ri Max-Heap ekanligini tekshiruvchi funksiya yozing.
5. **`decrease-key` operatsiyasini amalga oshiring** — Dijkstra algoritmi uchun zarur bo'lgan, heapdagi biror elementning qiymatini kamaytirish va heapni qayta tartiblash funksiyasini yozing.
6. **Heap Sort'ni Quick Sort bilan solishtiring** — ikkalasini ham amalga oshirib, katta massivlarda vaqt bo'yicha taqqoslang.

---

## 14. Xulosa

Heap — bu oddiy, ammo juda kuchli ma'lumot tuzilmasi bo'lib, u:

- Array yordamida samarali ifodalanadi (qo'shimcha pointer kerak emas);
- Eng katta/kichik elementni `O(1)` da topish imkonini beradi;
- Qo'shish va o'chirish amallarini `O(log n)` da bajaradi;
- Priority Queue, Dijkstra, Prim, Heap Sort kabi ko'plab algoritm va tuzilmalarning asosini tashkil etadi.

Heapni chuqur tushunish — algoritmlar va ma'lumotlar tuzilmalari sohasida mustahkam poydevor yaratadi. Uni noldan yozib ko'rish (bu darslikda qilganingizdek) sift-up va sift-down mexanizmlarini chuqur anglashga yordam beradi, standart kutubxonalardan (`heapq`, `std::priority_queue`) foydalanish esa amaliy loyihalarda vaqtingizni tejaydi.

**Keyingi qadam sifatida** tavsiya etiladi: LeetCode yoki AtCoder'dan heap mavzusidagi masalalarni yechib, nazariyani amaliyot bilan mustahkamlang (masalan: "Kth Largest Element in an Array", "Merge K Sorted Lists", "Top K Frequent Elements").
