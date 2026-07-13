# Sortlash (Sort) Algoritmlari — To'liq Darslik

## Kirish

Sortlash — bu ma'lumotlar to'plamini (massiv, ro'yxat) ma'lum bir tartibda (o'sish yoki kamayish bo'yicha) joylashtirish jarayoni. Bu — algoritmlar va ma'lumot tuzilmalarining eng fundamental mavzularidan biri.

Har bir algoritm uchun quyidagilarni ko'rib chiqamiz:
- **G'oyasi** — algoritm qanday ishlaydi
- **Psevdokod** — tilga bog'liq bo'lmagan umumiy yozuv
- **Big-O tahlili** — Best/Average/Worst case, xotira murakkabligi, barqarorlik (stability)
- **Real kod** — C, C++, Python tillarida

### Asosiy tushunchalar

| Tushuncha | Ma'nosi |
|---|---|
| **Stable (barqaror)** | Teng elementlarning nisbiy tartibi saqlanib qoladi |
| **In-place** | Qo'shimcha massiv talab qilmaydi (O(1) xotira) |
| **Comparison-based** | Elementlarni solishtirish orqali ishlaydi (nazariy pastki chegara: O(n log n)) |

---

## 1. Bubble Sort (Pufakcha usuli)

### G'oyasi
Massivni bir necha marta aylanib chiqamiz. Har bir aylanishda qo'shni ikkita elementni solishtiramiz, agar tartib noto'g'ri bo'lsa — joylarini almashtiramiz. Katta elementlar asta-sekin massiv oxiriga "ko'tariladi" (xuddi suvdagi pufakcha kabi).

### Psevdokod
```
BUBBLE_SORT(A, n):
    for i = 0 to n-1:
        swapped = false
        for j = 0 to n-i-2:
            if A[j] > A[j+1]:
                swap(A[j], A[j+1])
                swapped = true
        if swapped == false:
            break   // massiv allaqachon tartiblangan
```

### Big-O tahlili
| Holat | Vaqt |
|---|---|
| Best (allaqachon tartiblangan) | O(n) |
| Average | O(n²) |
| Worst (teskari tartiblangan) | O(n²) |
| Xotira | O(1) — in-place |
| Stable? | Ha |

### C tilida
```c
#include <stdio.h>

void bubbleSort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        int swapped = 0;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = 1;
            }
        }
        if (!swapped) break;
    }
}

int main() {
    int arr[] = {64, 34, 25, 12, 22, 11, 90};
    int n = sizeof(arr) / sizeof(arr[0]);
    bubbleSort(arr, n);
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    return 0;
}
```

### C++ tilida
```cpp
#include <iostream>
#include <vector>
using namespace std;

void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        bool swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}

int main() {
    vector<int> arr = {64, 34, 25, 12, 22, 11, 90};
    bubbleSort(arr);
    for (int x : arr) cout << x << " ";
}
```

### Python tilida
```python
def bubble_sort(arr):
    n = len(arr)
    for i in range(n - 1):
        swapped = False
        for j in range(n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True
        if not swapped:
            break
    return arr

print(bubble_sort([64, 34, 25, 12, 22, 11, 90]))
```

---

## 2. Selection Sort (Tanlash usuli)

### G'oyasi
Massivning tartiblanmagan qismidan eng kichik elementni topamiz va uni tartiblangan qismning oxiriga qo'yamiz. Bu jarayonni butun massiv tartiblanguncha takrorlaymiz.

### Psevdokod
```
SELECTION_SORT(A, n):
    for i = 0 to n-2:
        minIndex = i
        for j = i+1 to n-1:
            if A[j] < A[minIndex]:
                minIndex = j
        swap(A[i], A[minIndex])
```

### Big-O tahlili
| Holat | Vaqt |
|---|---|
| Best | O(n²) |
| Average | O(n²) |
| Worst | O(n²) |
| Xotira | O(1) — in-place |
| Stable? | Yo'q (standart versiyada) |

Selection Sort har doim O(n²) solishtirish qiladi — massiv qanday holatda bo'lishidan qat'iy nazar. Lekin swap (almashtirish) soni kam — atigi O(n) marta, shuning uchun yozish qimmat bo'lgan holatlarda foydali.

### C tilida
```c
#include <stdio.h>

void selectionSort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) minIdx = j;
        }
        int temp = arr[minIdx];
        arr[minIdx] = arr[i];
        arr[i] = temp;
    }
}

int main() {
    int arr[] = {64, 25, 12, 22, 11};
    int n = sizeof(arr) / sizeof(arr[0]);
    selectionSort(arr, n);
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    return 0;
}
```

### C++ tilida
```cpp
#include <iostream>
#include <vector>
using namespace std;

void selectionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) minIdx = j;
        }
        swap(arr[i], arr[minIdx]);
    }
}

int main() {
    vector<int> arr = {64, 25, 12, 22, 11};
    selectionSort(arr);
    for (int x : arr) cout << x << " ";
}
```

### Python tilida
```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n - 1):
        min_idx = i
        for j in range(i + 1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i]
    return arr

print(selection_sort([64, 25, 12, 22, 11]))
```

---

## 3. Insertion Sort (Kiritish usuli)

### G'oyasi
Karta o'yinidagi kabi — har bir elementni tartiblangan qismning to'g'ri joyiga "kiritamiz". Chapdan boshlab, har bir yangi elementni orqaga qarab siljitib, o'z o'rniga qo'yamiz.

### Psevdokod
```
INSERTION_SORT(A, n):
    for i = 1 to n-1:
        key = A[i]
        j = i - 1
        while j >= 0 and A[j] > key:
            A[j+1] = A[j]
            j = j - 1
        A[j+1] = key
```

### Big-O tahlili
| Holat | Vaqt |
|---|---|
| Best (tartiblangan) | O(n) |
| Average | O(n²) |
| Worst (teskari tartiblangan) | O(n²) |
| Xotira | O(1) — in-place |
| Stable? | Ha |

Kichik yoki deyarli tartiblangan massivlar uchun juda samarali. Shuning uchun Timsort va Introsort kabi hibrid algoritmlarda kichik bo'laklar uchun ishlatiladi.

### C tilida
```c
#include <stdio.h>

void insertionSort(int arr[], int n) {
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}

int main() {
    int arr[] = {12, 11, 13, 5, 6};
    int n = sizeof(arr) / sizeof(arr[0]);
    insertionSort(arr, n);
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    return 0;
}
```

### C++ tilida
```cpp
#include <iostream>
#include <vector>
using namespace std;

void insertionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}

int main() {
    vector<int> arr = {12, 11, 13, 5, 6};
    insertionSort(arr);
    for (int x : arr) cout << x << " ";
}
```

### Python tilida
```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1
        arr[j + 1] = key
    return arr

print(insertion_sort([12, 11, 13, 5, 6]))
```

---

## 4. Shell Sort

### G'oyasi
Insertion Sort'ning takomillashtirilgan versiyasi. Elementlarni katta "gap" (interval) bilan solishtirib, asta-sekin gap'ni kamaytirib boramiz. Bu uzoqdagi elementlarni tezroq to'g'ri joyga yaqinlashtirishga yordam beradi.

### Psevdokod
```
SHELL_SORT(A, n):
    gap = n / 2
    while gap > 0:
        for i = gap to n-1:
            temp = A[i]
            j = i
            while j >= gap and A[j-gap] > temp:
                A[j] = A[j-gap]
                j = j - gap
            A[j] = temp
        gap = gap / 2
```

### Big-O tahlili
| Holat | Vaqt |
|---|---|
| Best | O(n log n) |
| Average | O(n^1.3) taxminan (gap ketma-ketligiga bog'liq) |
| Worst | O(n²) |
| Xotira | O(1) — in-place |
| Stable? | Yo'q |

### C tilida
```c
#include <stdio.h>

void shellSort(int arr[], int n) {
    for (int gap = n / 2; gap > 0; gap /= 2) {
        for (int i = gap; i < n; i++) {
            int temp = arr[i];
            int j = i;
            while (j >= gap && arr[j - gap] > temp) {
                arr[j] = arr[j - gap];
                j -= gap;
            }
            arr[j] = temp;
        }
    }
}

int main() {
    int arr[] = {12, 34, 54, 2, 3};
    int n = sizeof(arr) / sizeof(arr[0]);
    shellSort(arr, n);
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    return 0;
}
```

### C++ tilida
```cpp
#include <iostream>
#include <vector>
using namespace std;

void shellSort(vector<int>& arr) {
    int n = arr.size();
    for (int gap = n / 2; gap > 0; gap /= 2) {
        for (int i = gap; i < n; i++) {
            int temp = arr[i];
            int j = i;
            while (j >= gap && arr[j - gap] > temp) {
                arr[j] = arr[j - gap];
                j -= gap;
            }
            arr[j] = temp;
        }
    }
}

int main() {
    vector<int> arr = {12, 34, 54, 2, 3};
    shellSort(arr);
    for (int x : arr) cout << x << " ";
}
```

### Python tilida
```python
def shell_sort(arr):
    n = len(arr)
    gap = n // 2
    while gap > 0:
        for i in range(gap, n):
            temp = arr[i]
            j = i
            while j >= gap and arr[j - gap] > temp:
                arr[j] = arr[j - gap]
                j -= gap
            arr[j] = temp
        gap //= 2
    return arr

print(shell_sort([12, 34, 54, 2, 3]))
```

---

## 5. Merge Sort (Birlashtirish usuli)

### G'oyasi
**Divide and Conquer** (Böl va hukmronlik qil) strategiyasi. Massivni ikkiga bo'lamiz, har bir yarmini rekursiv tartiblaymiz, so'ngra ikkita tartiblangan yarmini birlashtiramiz (merge qilamiz).

### Psevdokod
```
MERGE_SORT(A, left, right):
    if left < right:
        mid = (left + right) / 2
        MERGE_SORT(A, left, mid)
        MERGE_SORT(A, mid+1, right)
        MERGE(A, left, mid, right)

MERGE(A, left, mid, right):
    L = A[left..mid]
    R = A[mid+1..right]
    i = 0, j = 0, k = left
    while i < len(L) and j < len(R):
        if L[i] <= R[j]:
            A[k] = L[i]; i++
        else:
            A[k] = R[j]; j++
        k++
    // qolgan elementlarni ko'chirish
    copy remaining L and R into A
```

### Big-O tahlili
| Holat | Vaqt |
|---|---|
| Best | O(n log n) |
| Average | O(n log n) |
| Worst | O(n log n) |
| Xotira | O(n) — qo'shimcha massiv kerak |
| Stable? | Ha |

Har doim kafolatlangan O(n log n) — bu uning eng katta afzalligi. Katta ma'lumotlar va **linked list**larni tartiblash uchun juda mos (chunki qo'shimcha xotira random-access talab qilmaydi).

### C tilida
```c
#include <stdio.h>
#include <stdlib.h>

void merge(int arr[], int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    int *L = malloc(n1 * sizeof(int));
    int *R = malloc(n2 * sizeof(int));

    for (int i = 0; i < n1; i++) L[i] = arr[left + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];

    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) arr[k++] = L[i++];
        else arr[k++] = R[j++];
    }
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];

    free(L);
    free(R);
}

void mergeSort(int arr[], int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}

int main() {
    int arr[] = {12, 11, 13, 5, 6, 7};
    int n = sizeof(arr) / sizeof(arr[0]);
    mergeSort(arr, 0, n - 1);
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    return 0;
}
```

### C++ tilida
```cpp
#include <iostream>
#include <vector>
using namespace std;

void merge(vector<int>& arr, int left, int mid, int right) {
    vector<int> L(arr.begin() + left, arr.begin() + mid + 1);
    vector<int> R(arr.begin() + mid + 1, arr.begin() + right + 1);

    int i = 0, j = 0, k = left;
    while (i < (int)L.size() && j < (int)R.size()) {
        if (L[i] <= R[j]) arr[k++] = L[i++];
        else arr[k++] = R[j++];
    }
    while (i < (int)L.size()) arr[k++] = L[i++];
    while (j < (int)R.size()) arr[k++] = R[j++];
}

void mergeSort(vector<int>& arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}

int main() {
    vector<int> arr = {12, 11, 13, 5, 6, 7};
    mergeSort(arr, 0, arr.size() - 1);
    for (int x : arr) cout << x << " ";
}
```

### Python tilida
```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])

    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result

print(merge_sort([12, 11, 13, 5, 6, 7]))
```

---

## 6. Quick Sort (Tez saralash)

### G'oyasi
**Divide and Conquer** strategiyasi. Bitta "pivot" (tayanch) elementni tanlaymiz, massivni pivotdan kichik va pivotdan katta ikki qismga bo'lamiz (partition), so'ngra har bir qismni rekursiv tartiblaymiz.

### Psevdokod
```
QUICK_SORT(A, low, high):
    if low < high:
        pi = PARTITION(A, low, high)
        QUICK_SORT(A, low, pi - 1)
        QUICK_SORT(A, pi + 1, high)

PARTITION(A, low, high):
    pivot = A[high]
    i = low - 1
    for j = low to high - 1:
        if A[j] < pivot:
            i = i + 1
            swap(A[i], A[j])
    swap(A[i+1], A[high])
    return i + 1
```

### Big-O tahlili
| Holat | Vaqt |
|---|---|
| Best | O(n log n) |
| Average | O(n log n) |
| Worst (allaqachon tartiblangan + yomon pivot tanlash) | O(n²) |
| Xotira | O(log n) — rekursiya stack uchun |
| Stable? | Yo'q |

Amaliyotda odatda Merge Sort'dan tezroq ishlaydi, chunki cache-friendly va konstanta koeffitsienti kichik. Worst case'dan qochish uchun pivotni tasodifiy yoki median-of-three usulida tanlash tavsiya etiladi.

### C tilida
```c
#include <stdio.h>

void swap(int *a, int *b) {
    int t = *a; *a = *b; *b = t;
}

int partition(int arr[], int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(&arr[i], &arr[j]);
        }
    }
    swap(&arr[i + 1], &arr[high]);
    return i + 1;
}

void quickSort(int arr[], int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

int main() {
    int arr[] = {10, 7, 8, 9, 1, 5};
    int n = sizeof(arr) / sizeof(arr[0]);
    quickSort(arr, 0, n - 1);
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    return 0;
}
```

### C++ tilida
```cpp
#include <iostream>
#include <vector>
using namespace std;

int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[high]);
    return i + 1;
}

void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

int main() {
    vector<int> arr = {10, 7, 8, 9, 1, 5};
    quickSort(arr, 0, arr.size() - 1);
    for (int x : arr) cout << x << " ";
}
```

### Python tilida
```python
def quick_sort(arr, low=0, high=None):
    if high is None:
        high = len(arr) - 1

    if low < high:
        pi = partition(arr, low, high)
        quick_sort(arr, low, pi - 1)
        quick_sort(arr, pi + 1, high)
    return arr

def partition(arr, low, high):
    pivot = arr[high]
    i = low - 1
    for j in range(low, high):
        if arr[j] < pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1

print(quick_sort([10, 7, 8, 9, 1, 5]))
```

---

## 7. Heap Sort

### G'oyasi
Massivdan **Max-Heap** (yoki Min-Heap) qurib olamiz — bunda har bir ota-element farzand elementlaridan katta bo'ladi. Keyin eng katta elementni (ildizni) massiv oxiriga chiqarib, heap o'lchamini kamaytirib, qayta "heapify" qilamiz.

Zafar, siz Heap mavzusini allaqachon batafsil o'rganganingiz uchun bu yerda o'sha bilim asosida — endi uni sortlashda qo'llaymiz.

### Psevdokod
```
HEAP_SORT(A, n):
    // Max-Heap qurish
    for i = n/2 - 1 downto 0:
        HEAPIFY(A, n, i)

    // Elementlarni birma-bir chiqarish
    for i = n-1 downto 1:
        swap(A[0], A[i])
        HEAPIFY(A, i, 0)

HEAPIFY(A, n, i):
    largest = i
    left = 2*i + 1
    right = 2*i + 2
    if left < n and A[left] > A[largest]:
        largest = left
    if right < n and A[right] > A[largest]:
        largest = right
    if largest != i:
        swap(A[i], A[largest])
        HEAPIFY(A, n, largest)
```

### Big-O tahlili
| Holat | Vaqt |
|---|---|
| Best | O(n log n) |
| Average | O(n log n) |
| Worst | O(n log n) |
| Xotira | O(1) — in-place |
| Stable? | Yo'q |

Har doim kafolatlangan O(n log n) va qo'shimcha xotira talab qilmaydi — bu Quick Sort'dan afzalligi. Lekin amaliyotda cache-locality yomonroq bo'lgani uchun ko'pincha Quick Sort'dan sekinroq ishlaydi.

### C tilida
```c
#include <stdio.h>

void heapify(int arr[], int n, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;

    if (left < n && arr[left] > arr[largest]) largest = left;
    if (right < n && arr[right] > arr[largest]) largest = right;

    if (largest != i) {
        int temp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = temp;
        heapify(arr, n, largest);
    }
}

void heapSort(int arr[], int n) {
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);

    for (int i = n - 1; i > 0; i--) {
        int temp = arr[0];
        arr[0] = arr[i];
        arr[i] = temp;
        heapify(arr, i, 0);
    }
}

int main() {
    int arr[] = {12, 11, 13, 5, 6, 7};
    int n = sizeof(arr) / sizeof(arr[0]);
    heapSort(arr, n);
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    return 0;
}
```

### C++ tilida
```cpp
#include <iostream>
#include <vector>
using namespace std;

void heapify(vector<int>& arr, int n, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;

    if (left < n && arr[left] > arr[largest]) largest = left;
    if (right < n && arr[right] > arr[largest]) largest = right;

    if (largest != i) {
        swap(arr[i], arr[largest]);
        heapify(arr, n, largest);
    }
}

void heapSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);

    for (int i = n - 1; i > 0; i--) {
        swap(arr[0], arr[i]);
        heapify(arr, i, 0);
    }
}

int main() {
    vector<int> arr = {12, 11, 13, 5, 6, 7};
    heapSort(arr);
    for (int x : arr) cout << x << " ";
}
```

### Python tilida
```python
def heapify(arr, n, i):
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2

    if left < n and arr[left] > arr[largest]:
        largest = left
    if right < n and arr[right] > arr[largest]:
        largest = right

    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)

def heap_sort(arr):
    n = len(arr)
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)

    for i in range(n - 1, 0, -1):
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr, i, 0)
    return arr

print(heap_sort([12, 11, 13, 5, 6, 7]))
```

---

## 8. Counting Sort (Sanash usuli)

### G'oyasi
**Comparison-based emas!** Elementlarni solishtirmasdan, ularning nechta marta uchraganini sanaymiz. Faqat ma'lum diapazondagi (0 dan k gacha) butun sonlar uchun ishlaydi.

### Psevdokod
```
COUNTING_SORT(A, n, k):     // k = maksimal qiymat
    count = array of size k+1, filled with 0
    output = array of size n

    for i = 0 to n-1:
        count[A[i]] += 1

    for i = 1 to k:
        count[i] += count[i-1]     // kumulyativ summa

    for i = n-1 downto 0:
        output[count[A[i]] - 1] = A[i]
        count[A[i]] -= 1

    return output
```

### Big-O tahlili
| Holat | Vaqt |
|---|---|
| Best/Average/Worst | O(n + k) |
| Xotira | O(n + k) |
| Stable? | Ha (to'g'ri implementatsiyada) |

`k` — elementlarning maksimal qiymati. Agar `k` `n`ga nisbatan juda katta bo'lsa (masalan, sonlar 1 dan 10^9 gacha), bu algoritm samarasiz bo'lib qoladi.

### C tilida
```c
#include <stdio.h>
#include <string.h>

void countingSort(int arr[], int n) {
    int max = arr[0];
    for (int i = 1; i < n; i++)
        if (arr[i] > max) max = arr[i];

    int *count = calloc(max + 1, sizeof(int));
    int *output = malloc(n * sizeof(int));

    for (int i = 0; i < n; i++) count[arr[i]]++;
    for (int i = 1; i <= max; i++) count[i] += count[i - 1];

    for (int i = n - 1; i >= 0; i--) {
        output[count[arr[i]] - 1] = arr[i];
        count[arr[i]]--;
    }

    memcpy(arr, output, n * sizeof(int));
    free(count);
    free(output);
}

int main() {
    int arr[] = {4, 2, 2, 8, 3, 3, 1};
    int n = sizeof(arr) / sizeof(arr[0]);
    countingSort(arr, n);
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    return 0;
}
```

### C++ tilida
```cpp
#include <iostream>
#include <vector>
using namespace std;

void countingSort(vector<int>& arr) {
    int maxVal = *max_element(arr.begin(), arr.end());
    vector<int> count(maxVal + 1, 0);
    vector<int> output(arr.size());

    for (int x : arr) count[x]++;
    for (int i = 1; i <= maxVal; i++) count[i] += count[i - 1];

    for (int i = arr.size() - 1; i >= 0; i--) {
        output[count[arr[i]] - 1] = arr[i];
        count[arr[i]]--;
    }
    arr = output;
}

int main() {
    vector<int> arr = {4, 2, 2, 8, 3, 3, 1};
    countingSort(arr);
    for (int x : arr) cout << x << " ";
}
```

### Python tilida
```python
def counting_sort(arr):
    if not arr:
        return arr
    max_val = max(arr)
    count = [0] * (max_val + 1)
    output = [0] * len(arr)

    for x in arr:
        count[x] += 1
    for i in range(1, max_val + 1):
        count[i] += count[i - 1]

    for x in reversed(arr):
        output[count[x] - 1] = x
        count[x] -= 1

    return output

print(counting_sort([4, 2, 2, 8, 3, 3, 1]))
```

---

## 9. Radix Sort

### G'oyasi
**Comparison-based emas!** Sonlarni raqam-ma-raqam (eng kichik xonadan boshlab) Counting Sort yordamida tartiblaymiz. Har bir xona (units, tens, hundreds...) bo'yicha alohida sanash orqali saralaymiz.

### Psevdokod
```
RADIX_SORT(A, n):
    max = MAX(A)
    exp = 1
    while max / exp > 0:
        COUNTING_SORT_BY_DIGIT(A, n, exp)
        exp = exp * 10

COUNTING_SORT_BY_DIGIT(A, n, exp):
    output = array of size n
    count = array of size 10, filled with 0

    for i = 0 to n-1:
        digit = (A[i] / exp) % 10
        count[digit] += 1

    for i = 1 to 9:
        count[i] += count[i-1]

    for i = n-1 downto 0:
        digit = (A[i] / exp) % 10
        output[count[digit] - 1] = A[i]
        count[digit] -= 1

    A = output
```

### Big-O tahlili
| Holat | Vaqt |
|---|---|
| Best/Average/Worst | O(d × (n + k)) |
| Xotira | O(n + k) |
| Stable? | Ha |

Bu yerda `d` — eng katta sondagi raqamlar soni, `k` — asos (odatda 10). Sonlar cheklangan uzunlikda bo'lganda deyarli O(n) tezlikda ishlaydi.

### C tilida
```c
#include <stdio.h>
#include <string.h>

int getMax(int arr[], int n) {
    int max = arr[0];
    for (int i = 1; i < n; i++)
        if (arr[i] > max) max = arr[i];
    return max;
}

void countingSortByDigit(int arr[], int n, int exp) {
    int output[n];
    int count[10] = {0};

    for (int i = 0; i < n; i++)
        count[(arr[i] / exp) % 10]++;

    for (int i = 1; i < 10; i++)
        count[i] += count[i - 1];

    for (int i = n - 1; i >= 0; i--) {
        int digit = (arr[i] / exp) % 10;
        output[count[digit] - 1] = arr[i];
        count[digit]--;
    }

    memcpy(arr, output, n * sizeof(int));
}

void radixSort(int arr[], int n) {
    int max = getMax(arr, n);
    for (int exp = 1; max / exp > 0; exp *= 10)
        countingSortByDigit(arr, n, exp);
}

int main() {
    int arr[] = {170, 45, 75, 90, 802, 24, 2, 66};
    int n = sizeof(arr) / sizeof(arr[0]);
    radixSort(arr, n);
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    return 0;
}
```

### C++ tilida
```cpp
#include <iostream>
#include <vector>
using namespace std;

int getMax(vector<int>& arr) {
    return *max_element(arr.begin(), arr.end());
}

void countingSortByDigit(vector<int>& arr, int exp) {
    int n = arr.size();
    vector<int> output(n);
    int count[10] = {0};

    for (int i = 0; i < n; i++)
        count[(arr[i] / exp) % 10]++;

    for (int i = 1; i < 10; i++)
        count[i] += count[i - 1];

    for (int i = n - 1; i >= 0; i--) {
        int digit = (arr[i] / exp) % 10;
        output[count[digit] - 1] = arr[i];
        count[digit]--;
    }
    arr = output;
}

void radixSort(vector<int>& arr) {
    int maxVal = getMax(arr);
    for (int exp = 1; maxVal / exp > 0; exp *= 10)
        countingSortByDigit(arr, exp);
}

int main() {
    vector<int> arr = {170, 45, 75, 90, 802, 24, 2, 66};
    radixSort(arr);
    for (int x : arr) cout << x << " ";
}
```

### Python tilida
```python
def counting_sort_by_digit(arr, exp):
    n = len(arr)
    output = [0] * n
    count = [0] * 10

    for x in arr:
        digit = (x // exp) % 10
        count[digit] += 1

    for i in range(1, 10):
        count[i] += count[i - 1]

    for x in reversed(arr):
        digit = (x // exp) % 10
        output[count[digit] - 1] = x
        count[digit] -= 1

    return output

def radix_sort(arr):
    if not arr:
        return arr
    max_val = max(arr)
    exp = 1
    while max_val // exp > 0:
        arr = counting_sort_by_digit(arr, exp)
        exp *= 10
    return arr

print(radix_sort([170, 45, 75, 90, 802, 24, 2, 66]))
```

---

## 10. Bucket Sort (Chelaklarga bo'lish usuli)

### G'oyasi
Elementlarni bir nechta "chelak"larga (bucket) taqsimlaymiz — har bir chelak ma'lum qiymat oralig'iga mos keladi. Har bir chelakni alohida tartiblaymiz (odatda Insertion Sort bilan), so'ngra hammasini birlashtiramiz. `[0, 1)` oralig'idagi uniform taqsimlangan float sonlar uchun ayniqsa samarali.

### Psevdokod
```
BUCKET_SORT(A, n):
    create n empty buckets
    for i = 0 to n-1:
        idx = n * A[i]              // [0,1) oralig'i uchun
        insert A[i] into bucket[idx]

    for each bucket:
        sort bucket (masalan, insertion sort bilan)

    concatenate all buckets in order
```

### Big-O tahlili
| Holat | Vaqt |
|---|---|
| Best | O(n + k) |
| Average | O(n + k) |
| Worst (barcha elementlar bitta chelakka tushsa) | O(n²) |
| Xotira | O(n + k) |
| Stable? | Ha (agar ichki sort stable bo'lsa) |

### C tilida
```c
#include <stdio.h>
#include <stdlib.h>

#define N 8

void insertionSortFloat(float arr[], int n) {
    for (int i = 1; i < n; i++) {
        float key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}

void bucketSort(float arr[], int n) {
    float **buckets = malloc(n * sizeof(float *));
    int *counts = calloc(n, sizeof(int));
    for (int i = 0; i < n; i++)
        buckets[i] = malloc(n * sizeof(float));

    for (int i = 0; i < n; i++) {
        int idx = (int)(n * arr[i]);
        buckets[idx][counts[idx]++] = arr[i];
    }

    int k = 0;
    for (int i = 0; i < n; i++) {
        insertionSortFloat(buckets[i], counts[i]);
        for (int j = 0; j < counts[i]; j++)
            arr[k++] = buckets[i][j];
        free(buckets[i]);
    }
    free(buckets);
    free(counts);
}

int main() {
    float arr[N] = {0.897, 0.565, 0.656, 0.1234, 0.665, 0.3434, 0.897, 0.5};
    bucketSort(arr, N);
    for (int i = 0; i < N; i++) printf("%.4f ", arr[i]);
    return 0;
}
```

### C++ tilida
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void bucketSort(vector<float>& arr) {
    int n = arr.size();
    vector<vector<float>> buckets(n);

    for (float x : arr) {
        int idx = n * x;
        buckets[idx].push_back(x);
    }

    for (auto& b : buckets)
        sort(b.begin(), b.end());

    int k = 0;
    for (auto& b : buckets)
        for (float x : b)
            arr[k++] = x;
}

int main() {
    vector<float> arr = {0.897, 0.565, 0.656, 0.1234, 0.665, 0.3434, 0.897, 0.5};
    bucketSort(arr);
    for (float x : arr) cout << x << " ";
}
```

### Python tilida
```python
def bucket_sort(arr):
    n = len(arr)
    buckets = [[] for _ in range(n)]

    for x in arr:
        idx = int(n * x)
        buckets[idx].append(x)

    for b in buckets:
        b.sort()  # insertion sort o'rniga Python'ning ichki sort'i

    result = []
    for b in buckets:
        result.extend(b)
    return result

print(bucket_sort([0.897, 0.565, 0.656, 0.1234, 0.665, 0.3434, 0.897, 0.5]))
```

---

## Umumiy Taqqoslash Jadvali

| Algoritm | Best | Average | Worst | Xotira | Stable | In-place |
|---|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Ha | Ha |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | Yo'q | Ha |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Ha | Ha |
| Shell Sort | O(n log n) | O(n^1.3) | O(n²) | O(1) | Yo'q | Ha |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Ha | Yo'q |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | Yo'q | Ha |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | Yo'q | Ha |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(n+k) | Ha | Yo'q |
| Radix Sort | O(d(n+k)) | O(d(n+k)) | O(d(n+k)) | O(n+k) | Ha | Yo'q |
| Bucket Sort | O(n+k) | O(n+k) | O(n²) | O(n+k) | Ha | Yo'q |

### Qaysi algoritmni qachon ishlatish kerak?

- **Kichik massiv (n < ~50)**: Insertion Sort — juda oddiy va tez
- **Umumiy maqsad, kafolatlangan tezlik kerak**: Merge Sort yoki Heap Sort
- **Amaliyotda eng tez (o'rtacha holatda)**: Quick Sort — shu sababli standart kutubxonalarda ko'p ishlatiladi (masalan, C++ `std::sort` — Introsort, ya'ni Quick Sort + Heap Sort + Insertion Sort gibridi)
- **Ma'lum diapazondagi butun sonlar**: Counting Sort yoki Radix Sort
- **Qo'shimcha xotira cheklangan**: Heap Sort (O(1) xotira, kafolatlangan O(n log n))
- **Stability muhim (masalan, bir nechta kalit bo'yicha saralash)**: Merge Sort, Insertion Sort, yoki Counting/Radix Sort
- **Linked list**: Merge Sort — chunki random-access kerak emas

### Python'ning ichki `sort()` va `sorted()` funksiyasi

Python `Timsort` algoritmidan foydalanadi — Merge Sort va Insertion Sort gibridi, real dunyodagi ma'lumotlarda (qisman tartiblangan bloklar) juda samarali, kafolatlangan O(n log n) worst-case va O(n) best-case bilan.

```python
arr = [5, 2, 8, 1, 9]
arr.sort()          # in-place, None qaytaradi
new_arr = sorted(arr)  # yangi ro'yxat qaytaradi
```

C++'ning `std::sort` esa **Introsort** (Quick Sort + Heap Sort fallback + Insertion Sort kichik bo'laklar uchun) ishlatadi.
