## Id посылки: 349265284
## GIT:https://github.com/vputt/set3/tree/main/A2

# Анализ алгоритмов сортировки слиянием

- **Стандартный Merge Sort** - классическая реализация алгоритма
- **Гибридный Merge+Insertion Sort** - оптимизированная версия с переключением на insertion sort для малых подмассивов


```
├── merge_random.txt           # Результаты Merge Sort на случайных массивах
├── merge_reversed.txt         # Результаты Merge Sort на обратно отсортированных массивах
├── merge_nearly.txt           # Результаты Merge Sort на почти отсортированных массивах
├── hybrid_random.txt          # Результаты Hybrid Sort на случайных массивах
├── hybrid_reversed.txt        # Результаты Hybrid Sort на обратно отсортированных массивах
├── hybrid_nearly.txt          # Результаты Hybrid Sort на почти отсортированных массивах
└── README.md                  # Документация проекта
```

## Параметры тестирования

### Тестовые данные
- **Размеры массивов**: от 500 до 100000 элементов с шагом 100
- **Диапазон значений**: от 0 до 6000
- **Типы массивов**:
  1. **Random** - массивы со случайными элементами
  2. **Reversed** - массивы, отсортированные в обратном порядке (наихудший случай)
  3. **Nearly Sorted** - почти отсортированные массивы (5% элементов переставлены)

### Параметры измерений
- **Количество замеров**: 15 повторений для каждого размера массива
- **Метод усреднения**: среднее арифметическое
- **Единицы измерения**: микросекунды
- **Threshold значения** для гибридного алгоритма: 5, 10, 20, 30, 50

## Визуализация результатов

Для построения графиков можно использовать Python с библиотекой matplotlib:

```python
import numpy as np
import matplotlib.pyplot as plt

# --------------------- Функции для загрузки данных ---------------------
def load_data(filename):
    data = np.loadtxt(filename)
    size = data[:, 0]
    time = data[:, 1]
    return size, time

def load_hybrid_data(filename):
    data = np.loadtxt(filename)
    size = data[:, 0]
    times = data[:, 1:]
    return size, times

# --------------------- Функции для построения графиков ---------------------
def plot_merge_all(files_dict):
    plt.figure(figsize=(10,6))
    for key, filename in files_dict.items():
        size, time = load_data(filename)
        plt.plot(size, time, label=f"{key} Array")
    plt.title("Merge Sort", fontsize=16)
    plt.xlabel("Размер массива", fontsize=14)
    plt.ylabel("Время выполнения  (мкс)", fontsize=14)
    plt.legend(fontsize=12)
    plt.grid(True)
    plt.show()

def plot_hybrid(files_dict):
    thresholds = [5, 10, 20, 30, 50]
    for key, filename in files_dict.items():
        size, times = load_hybrid_data(filename)
        plt.figure(figsize=(10,6))
        for i, th in enumerate(thresholds):
            plt.plot(size, times[:, i], label=f"Threshold {th}")
        plt.title(f"Hybrid Merge+Insertion Sort\n{key} Array", fontsize=16)
        plt.xlabel("Размер массива", fontsize=14)
        plt.ylabel("Время выполнения  (мкс)", fontsize=14)
        plt.legend(fontsize=12)
        plt.grid(True)
        plt.show()

merge_files = {
    'Random': 'merge_random.txt',
    'Reversed': 'merge_reversed.txt',
    'Nearly Sorted': 'merge_nearly.txt'
}

hybrid_files = {
    'Random': 'hybrid_random.txt',
    'Reversed': 'hybrid_reversed.txt',
    'Nearly Sorted': 'hybrid_nearly.txt'
}
plot_merge_all(merge_files)
plot_hybrid(hybrid_files)

```

## Реализация алгоритмов

### Merge Sort
Классическая реализация алгоритма сортировки слиянием с временной сложностью O(n log n) для всех случаев.

```cpp
void mergeSort(vector<int>& array, int left, int right) {
  if (left < right) {
    int mid = left + (right - left) / 2;
    mergeSort(array, left, mid);
    mergeSort(array, mid + 1, right);
    merge(array, left, mid, right);
  }
}
```

### Hybrid Merge+Insertion Sort
Оптимизированная версия, использующая insertion sort для малых подмассивов (размер ≤ threshold).

```cpp
void hybridMergeSort(vector<int>& array, int left, int right, int threshold) {
  if (right - left + 1 <= threshold) {
    insertionSort(array, left, right);
    return;
  }
  
  if (left < right) {
    int mid = left + (right - left) / 2;
    hybridMergeSort(array, left, mid, threshold);
    hybridMergeSort(array, mid + 1, right, threshold);
    merge(array, left, mid, right);
  }
}
```

**Обоснование**: Insertion sort имеет меньшие константы и работает эффективнее на малых массивах, несмотря на сложность O(n²).

## Класс ArrayGenerator

Генератор создает один базовый массив максимальной длины, из которого извлекаются подмассивы нужного размера:
- getRandomArray(size) - первые size элементов базового массива
- getReversedArray(size) - отсортированный в обратном порядке
- getNearlySortedArray(size, swaps) - почти отсортированный с swaps перестановками

```cpp

class ArrayGenerator {
private:
  vector<int> baseArray;
  int maxLength;
  int minRange;
  int maxRange;

  void generateBaseArray() {
    std::mt19937 gen(static_cast<unsigned>(std::time(nullptr)));
    std::uniform_int_distribution<int> dist(minRange, maxRange);

    for (int i = 0; i <= maxLength; ++i) {
      baseArray.push_back(dist(gen));
    }
  }

public:
  ArrayGenerator() : maxLength(100000), minRange(0), maxRange(6000) {
    generateBaseArray();
  }

  vector<int> getRandomArray(int size) {
    if (size > maxLength) {
      size = maxLength;
    }
    return vector<int>(baseArray.begin(), baseArray.begin() + size);
  }

  vector<int> getReversedArray(int size) {
    if (size > maxLength) {
      size = maxLength;
    }
    vector<int> reversedArray(baseArray.begin(), baseArray.begin() + size);
    std::sort(reversedArray.rbegin(), reversedArray.rend());
    return reversedArray;
  }

  vector<int> getNearlySortedArray(int size, int swaps) {
    if (size > maxLength) {
      size = maxLength;
    }
    vector<int> nearlySortedArray(baseArray.begin(), baseArray.begin() + size);
    std::sort(nearlySortedArray.begin(), nearlySortedArray.end());

    std::mt19937 gen(static_cast<unsigned>(std::time(nullptr)) + size);
    std::uniform_int_distribution<int> dist(0, size - 1);

    for (int i = 0; i < swaps; ++i) {
      std::swap(nearlySortedArray[dist(gen)], nearlySortedArray[dist(gen)]);
    }

    return nearlySortedArray;
  }
};
```

## Класс SortTester

Класс для проведения замеров времени выполнения алгоритмов:

```cpp
class SortTester {
public:
  double measureMergeSortTime(vector<int>& array, int size);
  double measureHybridMergeSortTime(vector<int>& array, int size, int threshold);
};
```

- Выполняет 15 повторных измерений
- Вычисляет среднее арифметическое
- Возвращает результат в микросекундах

## Выводы

### 1. Эффективность гибридного подхода

Гибридная реализация `Merge + Insertion Sort` демонстрирует лучшую производительность по сравнению со стандартным `Merge Sort` на всех категориях тестовых данных. Основное преимущество достигается за счет использования сортировки вставками на небольших подмассивах, где её низкие константные затраты компенсируют квадратичную сложность.
![alt text](https://github.com/vputt/set3/blob/f900a492437606277c4f574acc1d8bae0b2670e5/A2/merge_sort.png)
![alt text](https://github.com/vputt/set3/blob/a1c74779e0880630945e90d054783d8c0269df87/A2/hybrid.png)
### 2. Критическая роль порогового значения

Выбор параметра `threshold` существенно влияет на производительность гибридного алгоритма:

- **Малые значения (threshold = 5)**: Недостаточная оптимизация, так как переключение на insertion sort происходит слишком редко. Алгоритм работает почти как обычный merge sort.

- **Большие значения (threshold = 50)**: Избыточное использование insertion sort приводит к деградации производительности из-за квадратичной сложности O(n²) на больших подмассивах.

- **Оптимальное значение (threshold = 20-30)**: Достигается баланс между преимуществами обоих алгоритмов, обеспечивая стабильную производительность.
![alt text](https://github.com/vputt/set3/blob/f900a492437606277c4f574acc1d8bae0b2670e5/A2/mrgSort%2BInsSort_reversed.png)

### 3. Различия в поведении на разных типах данных

Анализ показывает заметную зависимость времени выполнения от структуры входных данных:

- **Reversed Arrays** (обратно отсортированные): Показывают наилучшую производительность для обоих алгоритмов. Причина в том, что при слиянии подмассивы уже упорядочены относительно друг друга, что минимизирует количество сравнений.

- **Random Arrays** (случайные): Демонстрируют худшее время выполнения из-за отсутствия какой-либо предсказуемой структуры данных.

- **Nearly Sorted Arrays** (почти отсортированные): Гибридный алгоритм показывает максимальное преимущество на этом типе данных благодаря эффективности insertion sort на частично упорядоченных последовательностях.
![alt text](https://github.com/vputt/set3/blob/f900a492437606277c4f574acc1d8bae0b2670e5/A2/type.png)

### 4. Наблюдаемые аномалии на графиках

На графиках присутствуют локальные скачки времени выполнения, особенно заметные на массивах размером около 60000-80000 элементов. Эти аномалии объясняются влиянием внешних факторов:

- **Вмешательство операционной системы**
- **Управление памятью**
- **Аппаратные прерывания**
  
Несмотря на это, общая тенденция роста времени выполнения соответствует теоретической оценке O(n log n).

### 5. На основе эмпирического анализа можно заключить:

- Гибридный алгоритм обеспечивает прирост производительности **15-25%** в среднем
- Оптимальное значение threshold находится в диапазоне **20-30** элементов
- Наибольший эффект достигается на массивах, отсортированных в обратном порядке(меньше пересечений при слиянии -> меньше операций)
- На случайных массивах алгоритм медленнее всего(слияние треубет большего кол-ва операций)
- Для критичных по производительности приложений рекомендуется использовать гибридную реализацию с threshold = 20

![alt text](https://github.com/vputt/set3/blob/f900a492437606277c4f574acc1d8bae0b2670e5/A2/mrgSortVShybrid.png)  

![alt text](https://github.com/vputt/set3/blob/a1c74779e0880630945e90d054783d8c0269df87/A2/hybrid.png)


