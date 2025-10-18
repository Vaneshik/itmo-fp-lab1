# Лабораторная работа №1

**Студент:** Чусовлянов Максим Сергеевич  
**Группа:** P3308  
**Вариант:** [Задача 12](https://projecteuler.net/problem=12), [Задача 18](https://projecteuler.net/problem=18)

---

## Проблема №12

- **Название**: Highly Divisible Triangular Number

- **Описание**: Последовательность треугольных чисел генерируется путем добавления натуральных чисел. Так, $7$-ое треугольное число будет $1 + 2 + 3 + 4 + 5 + 6 + 7 = 28$. Первые десять треугольных чисел:

  ```
  1, 3, 6, 10, 15, 21, 28, 36, 45, 55, ...
  ```

  Можно видеть, что $28$ - это первое треугольное число, которое имеет более пяти делителей.

- **Задание**: Какое значение первого треугольного числа имеет более пятисот делителей?

### Идея решения

Треугольное число вычисляется по формуле: $T_n = \frac{n \cdot (n + 1)}{2}$

Для подсчета количества делителей числа используется оптимизация: достаточно перебрать делители до $\sqrt{n}$. Если $i$ - делитель числа $n$, то $\frac{n}{i}$ тоже делитель. Особый случай - когда $i = \sqrt{n}$, тогда добавляем только один делитель.

Вспомогательные функции реализованы в модуле [`math`](/src/itmo_fp_lab1/util/math.clj):

```clj
(defn triangle-number
  "Вычисляет n-ое треугольное число"
  [n]
  (quot (* n (inc n)) 2))
```

```clj
(defn count-divisors
  "Подсчитывает количество делителей числа n"
  [n]
  (if (<= n 0)
    0
    (let [sqrt-n (int (Math/sqrt n))
          divisors (reduce (fn [acc i]
                             (if (zero? (mod n i))
                               (if (= i (/ n i))
                                 (inc acc)
                                 (+ acc 2))
                               acc))
                           0
                           (range 1 (inc sqrt-n)))]
      divisors)))
```

### Монолитная реализация с использованием хвостовой рекурсии

Алгоритм:
- Начинаем с $n = 1$
- Вычисляем $n$-ое треугольное число
- Подсчитываем количество его делителей
- Если делителей больше заданного порога, возвращаем число
- Иначе переходим к $n + 1$

```clj
(defn highly-divisible-triangular [min-divisors]
  (loop [n 1]
    (let [tri-num (triangle-number n)
          divisors (count-divisors tri-num)]
      (if (> divisors min-divisors)
        tri-num
        (recur (inc n))))))
```

[Файл с решением](/src/itmo_fp_lab1/task12/tail_rec.clj)

### Простое рекурсивное решение

Аналогично хвостовой рекурсии, но используется вложенная функция без оптимизации:

```clj
(defn highly-divisible-triangular [min-divisors]
  (letfn [(find-number [n]
            (let [tri-num (triangle-number n)
                  divisors (count-divisors tri-num)]
              (if (> divisors min-divisors)
                tri-num
                (find-number (inc n)))))]
    (find-number 1)))
```

[Файл с решением](/src/itmo_fp_lab1/task12/simple_rec.clj)

### Модульная реализация

Здесь используется композиция функций высшего порядка:
- `iterate` создает бесконечную последовательность натуральных чисел
- `map` преобразует их в треугольные числа
- `filter` выбирает числа с достаточным количеством делителей
- `first` берет первый подходящий элемент

```clj
(defn highly-divisible-triangular [min-divisors]
  (->> (iterate inc 1)
       (map triangle-number)
       (filter #(> (count-divisors %) min-divisors))
       (first)))
```

[Файл с решением](/src/itmo_fp_lab1/task12/module.clj)

### Генерация последовательности через `map`

Расширенная версия модульной реализации, где `map` используется для создания структурированных данных:

```clj
(defn highly-divisible-triangular [min-divisors]
  (->> (iterate inc 1)
       (map #(vector % (triangle-number %)))
       (map (fn [[idx tri-num]] 
              {:index idx 
               :value tri-num 
               :divisors (count-divisors tri-num)}))
       (filter #(> (:divisors %) min-divisors))
       (first)
       (:value)))
```

[Файл с решением](/src/itmo_fp_lab1/task12/map.clj)

### Работа со спец синтаксисом циклов

Использование императивного стиля с атомами и циклом `while`:

```clj
(defn highly-divisible-triangular [min-divisors]
  (let [result (atom nil)
        n (atom 1)]
    (while (nil? @result)
      (let [tri-num (triangle-number @n)
            divisors (count-divisors tri-num)]
        (if (> divisors min-divisors)
          (reset! result tri-num)
          (swap! n inc))))
    @result))
```

[Файл с решением](/src/itmo_fp_lab1/task12/cycles.clj)

### Работа с бесконечными списками

Использование `drop-while` для пропуска элементов, не удовлетворяющих условию:

```clj
(defn highly-divisible-triangular [min-divisors]
  (->> (iterate inc 1)
       (map triangle-number)
       (drop-while #(<= (count-divisors %) min-divisors))
       (first)))
```

[Файл с решением](/src/itmo_fp_lab1/task12/inf_seq.clj)

### Решение на Python

```python
def count_divisors(n):
    if n <= 0:
        return 0
    count = 0
    sqrt_n = int(math.sqrt(n))
    for i in range(1, sqrt_n + 1):
        if n % i == 0:
            if i == n // i:
                count += 1
            else:
                count += 2
    return count

def triangle_number(n):
    return n * (n + 1) // 2

def highly_divisible_triangular(min_divisors):
    n = 1
    while True:
        tri_num = triangle_number(n)
        divisors = count_divisors(tri_num)
        if divisors > min_divisors:
            return tri_num
        n += 1
```

[Файл с решением](/py/task12.py)

---

