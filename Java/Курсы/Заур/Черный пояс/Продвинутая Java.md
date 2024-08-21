## Comparable & Comparator
> Используются для сравнения объектов.
> А именно помогают узнавать какой объект `>`, `<` или они `==`.
> Чаще всего это нужно для сортировки.

<font style="color:red">!!!</font> 
В идеале
```java
// если
e1.compareTo(e2) == 0; // или Comparator.compare(e1, e2) == 0
// то и
e1.equals(e2) == true

```
Но это в идеале, а не обязательно.

### Урок 1 Comparable
> Интерфейс `Comparable` используется для сравнения объектов, используя естественный порядок.

Под естественным порядком понимается порядок сортировки, логичный ( `естественный`) для сортируемого типа объектов:
- числовой порядок для сортировки целых чисел;
- алфавитный порядок для строк и т. д.

<font style="color:red">ВАЖНО</font> если для класса не видно логики, по которой можно его сортировать, то реализовывать `Comparable` **не нужно**.
А там где нужна будет сортировка по определенным правилам - использовать `Comparator`.

Чтобы имплиментировать `Comparable`, необходимо реализовать метод:
`int compareTo(Element e)`.

#### Collections.sort()
> Данный метод принимает в параметр коллекции объектов, которые реализуют интерфейс `Comparable<T>`.
> Иначе `Java` не поймет как нужно сравнивать м/ду собой объекты.

```java
List<Employee> list = new ArrayList<>();

Collections.sort(list);
```

- если в `Collections.sort()` передать коллекцию объектов не реализующих `Comparable<T>` -> будет ошибка компиляции.
- такие классы как `String`, `Integer` и т.д. реализуют `Comparable<T>`.

```java
class Employee implements Comparable<Employee> {
	private int id;
	private String name;
	private String surname;

	// Реализация 1
	public int compareTo(Employee anotherEmp) {
		if (this.id == anotherEmp.getId()) {
			return 0;
			// если id одинаковые, можно сравнить по имени:
			// return this.name.compareTo(anotherEmp.getName());
		} else if (this.id < anotherEmp.getId()) {
			return -1;
		} else {
			return 1;
		}
	}

	// Реализация 2
	public int compareTo(Employee anotherEmp) {
		return this.id - anotherEmp.getId();
	}

	// Реализация 3
	// Если id будет Integer, можно написать так:
	public int compareTo(Employee anotherEmp) {
		return this.compareTo(anotherEmp);
	}
}
```

##### compareTo
- если текущий объект (`this`) > переданного в параметр объекта: `return` **+ число**
- `this` < `paramObject` : return **- число**
- `this` == `paramObject` : return **0**

#### Arrays.sort()
> Сортирует массив объектов.

Если класс объектов массива не реализует `Comparable<T>` - ошибки компиляции не будет, но при выполнении выбросится `ClassCastException`. Т.к. `Java` не понимает как нужно сравнивать объекты массива.

```java
Arrays.sort(new Employee[] {emp1, emp2, emp3});
```

### Comparator
> Сравнитель.
>  Реализует кастомное (не естественное) сравнение.

Необходимо реализовать метод:
`int compare(Element e1, Element e2)`

```java
class IdComparator implements Comparator<Employee> {
	@Override
	public int compare(Employee emp1, Employee emp2) {
		return emp1.getId() - emp2.getId();
	}
}

class NameComparator implements Comparator<Employee> {
	@Override
	public int compare(Employee emp1, Employee emp2) {
		return emp1.getName().compareTo(emp2.getName());
	}
}
```

#### Collection.sort()
> В параметр метода  `sort()` можно передать компаратор.
> Компаратор можно передать как для коллекции объектов реализующих `Comparable`, так и не реализующих.

В любом случае сравнение будет осуществляться по переданному компаратору.
```java
List<Employee> list = new ArrayList<>();

Collections.sort(list, new IdComparator());
Collections.sort(list, new NameComparator());

// сортировка по имени в обратном порядке
Collections.sort(list, (e1, e2) -> e2.getName().compareTo(e1.getName()));
```

#### Совместное использование Comparable и Comparator
В `Comparable.compareTo()` реализуется естественная сортировка. Например, мы решили что объекты класса `Employee` естественно сортировать по `id`.

А в `Comparator.compare()` реализуются не естественные сортировки. При этом реализаций компараторов может быть несколько.
Например нам нужно сортировать `Employee` по имени (`NameComparator`)  или по имени и фамилии (`NameSurnameComparator`).

По этому, когда нам нужна естественная сортировка мы используем реализованный в классе `Comparable.compareTo()`, а когда любая другая используем `Comparator`.

`Comparator` можно не реализовывать в отдельном классе.
Достаточно написать анонимный класс компаратора, либо использовать `lambda`-выражение.


## Collections
### Урок 1 Введение в Collection и List
![[Pasted image 20240817121155.png]]

- Все коллекции работают с дженериками, т.е. содержат в себе эл-ты определенного типа.
- Благодаря интерфейсу `Itarable` мы можем пробегаться по элементам коллекций в `forEach()`.

#### List
> `List` -  это упорядоченная последовательность эл-тов, позволяющая хранить дубликаты и `null`. Каждый эл-нт имеет индекс.

![[Pasted image 20240817122040.png]]

### Урок 2 ArrayList
> `ArrayList` - это массив, который может изменять свою длину.

- Обычный массив не может изменять свою длину, длина задается при его создании;
- В основе `ArrayList` лежит массив `Object`
- Дефолтный размер внутреннего массива `ArrayList` = 10
- При заполнении внутреннего массива, создается новый массив с длиной
  `newLength = 1.5 * oldLength`
- Операция создания нового массива, при заполнении старого, является дорогостоящей. И если понятно, что массив будет содержать определенное минимальное кол-во эл-тов - можно сразу задать размер массива при создании `ArrayList`.

Создание `ArrayList`:
```java
// Создается дефолтный ArrayList
ArrayList<DataType> list1 = new ArrayList<>();

// Создается ArrayList с длиной внутреннего массива = 100
ArrayList<DataType> list2 = new ArrayList<>(100);

// Создается ArrayList на основе другого ArrayList-а
ArrayList<DataType> list3 = new ArrayList<>(list);
```

### Урок 3-4 методы ArrayList
#### Тезисы
- Методы, которые принимают в параметр `index` выбрасывают `IndexOutOfBoundsException` в случае, если:
  - `index >= size` (`get`, `remove`, `set`)
  - `index > size` (`add`)

#### Методы
- `add(DataType element) -> boolean`
- `add(int index, DataType element) -> boolean`
- `set(int index, DataType element) -> DataType`
- `remove(Object element) -> boolean`
  - сравнение объектов по `equals`
- `remove(int index) -> DataType`
  - Если удалить эл-нт `ArrayList` в середине списка **->** все эл-ты справа от удаляемого - сдвинутся влево.
- `addAll(ArrayList list) -> boolean`
- `addAll(int index, ArrayList list) -> boolean`
- `clear() -> void`
- `indexOf(Object element) -> int`
  - сравнение объектов по `equals`
  - если эл-нт не найден возвращается **`-1`**
- `lastIndexOf(Object element) -> int`
- `size() -> int`
- `isEmpty() -> boolean`
- `contains(Object element) -> boolean`
  - сравнение объектов по `equals`
- `toString() -> String`

### Урок 5 методы ArrayList и связанные с ArrayList
1. `Arrays.asList(DataType[]) -> List<DataType>`
   - созданный `List` жестко связан с массивом, на основе которого создается,
     и `->` имеет тот же размер.
2. `removeAll(Collection <?> c) -> boolean`
3. `retainAll(Collection <?> c -> boolean`
   - антоним метода `removeAll` -> в `ArrayList` останутся только те эл-ты, которые содержатся в переданной коллекции.
4. `containsAll(Collection <?> c) -> boolean`
   - проверяет, содержит ли `ArrayList`, все эл-ты из `ArrayList`, переданного в параметр
5. `subList(int fromIndex, int toIndex) -> List<E>`
   - `[fromIndex, toIndex)`
   - `subList` жестко связан с исходным листом и является его представлением (`view`)
   - если мы сделали `view` **ArrayList**, то изменения необходимо делать именно через этот `view`. Иначе если сделать изменения в исходном **ArrayList** -> `subList` уже не получится использовать, т.к. при первом его вызове будет выброшено `ConcurrentModificationExeption`.
1. `toArray() -> Object []`
2. `toArray(T[] a) - > T[]`
   - В параметр можно передавать массив нулевой длины, java автоматические увеличит его до необходимого размера: `list.toArray(new String[0])`
3. `List.of(E ... elements) -> List<E>`
   - return **unmodifiable* list
   -  не может содержать `null`
1. `List.copyOf(Collection<E> c) -> List<E>`
   - return **unmodifiable** list
   - не может содержать `null`

### Урок 6 Интерфейс Iterator
> Итерация - повторение.
> `Iterator` - повторитель.

```java

List<String> list = new ArrayList<>();
Iterator<String> iterator = list.iterator();

while(iterator.hasNext()) {
	iterator.next();
	iterator.remove();
}
```
- С помощью `Iterator` мы можем удалять эл-ты из коллекции, в отличии от  `forEach loop`.
- Для удаления сначала нужно обязательно получить эл-нт (`iterator.next()`),  а только после удалить (`iterator.remove()`).

### Урок 7 LinkedList
> Эл-ты `LinkedList` - это звенья одной цепочки.
> Эти эл-ты хранят определенные данные, а так же ссылки на предыдущий и следующий эл-ты.

#### Тезисы
- `LinkedList` реализует интерфейс `List` и имеет все его методы
- Каждый эл-нт `LinkedList` знает своих соседей и только их.
- `LinkedList` имеет быстрый доступ к первому и последнему эл-там.

<font style="color:red">Как правило</font>, `LinkedList` следует использовать когда:
1. Невелико кол-во операций получения эл-тов;
2. Велико кол-во операций добавления и удаления эл-тов. Особенно если речь идет об эл-тах в начале коллекции.

<font style="color:red">НО на самом деле</font>, практически во всех ситуациях, лучше использовать `ArrayList`, т.к. он будет работать быстрее и лучше подходит практически под все задачи.

#### Типы связанных списков
> В Java `LinkedList` реализован как **Doubly linked list**.

**Doubly linked list**
![[Pasted image 20240819193017.png]]

**Singly linked list**
![[Pasted image 20240819193058.png]]

### Урок 8 ListIterator
> Палиндром - зеркальное слово, которое читается одинаково с разных концов.

`ListIterator` может идти по эл-там не только вперед, но и назад.

```java
List<String> list = new LinkedList<>();
ListIterator<String> listIterator = list.listIterator();

while(listIterator.hasNext()) {
	listIterator.next();
}

ListIterator<String> reverseIterator = list.listIterator(list.size());

while(reverseListIterator.hasPrevious()) {
	reverseListIterator.previous();
}
```
- В параметр метода `listIterator(int index)` - передается индекс, с которого нужно начинать итерацию:
  - если итерируемся вперед, то итерация начнется с переданного `index`;
  - если назад - то с `index - 1` (по этому в примере выше передается `list.size()`, чтобы начать итерацию с последнего элемента листа);
- `listIterator()` без параметра, начинает с 0 индекса.

### Урок 9 Binary search
> `Binary search` осуществляет поиск за **O(log n)**.
> Но, массив/коллекция должны быть обязательно отсортированы.
> 
> `Binary search` - это просто деление пополам массива на каждом шаге.

В бинарном поиске используется двоичный логарифм для подсчета степени числа `2`.
Чтобы понять **количество операций**, необходимых для бинарного поиска, достаточно ответить на простой вопрос:
**В какую степень надо возвести число 2, чтобы полученный результат был `>=` числу эл-тов коллекции?**

Реализация `binarySearch` в листе/массиве:
```java
int binarySearch(List<? extends Comparable<? super T>> list, T valueToFind) {
	int low = 0;
	int high = list.size() - 1;

	while(low <= high) {
		int mid = low + (high - low) / 2;
		
		Comparable<? super T> value = list.get(mid);
		int compare = valueToFind.compareTo(value);

		if (compare > 0) {
			low = mid + 1;
		} else if (compare < 0) {
			high = mid - 1;
		} else {
			return mid;
		}
	}

	// +1 на случай, если low == 0
	return - (low + 1);
}
```

#### Collections.binarySearch()
> От реализации `Comparable.compareTo` или `Comparator.compare` зависит логика сравнения эл-тов при поиске.
> 
> Если `Collections.binarySearch` возвращает отрицательное значение, значит эл-нт не найден.

<font style="color:red">Важно понимать</font>:
при равенстве двух эл-тов через компаратор (`e1.compareTo(e2) == 0`)
может не быть равенства через **equals** (`e1.equals(e2) == false`).

Перегруженные формы `binarySearch`:
- `binarySearch(List<? extends Comparable<? super T>> list, T key) -> int`
  - `list` - коллекция, которая реализует `Comparable`
  - `key` - искомый эл-нт
- `binarySearch(List<? extends T> list, T key, Comparator<? super T> c) -> int`
  - `list` - коллекция, не обязательно реализующая `Comparable`
  - `key` - искомый эл-нт
  - `c` - компаратор, с помощью которого будет осуществляться поиск `key` в листе (в нем передается логика сравнение эл-тов коллекции с искомым).

```java
List<Employee> list = new ArrayList<>();
Collections.sort(list);
Collections.binarySearch(list, emp1);
```

#### Arrays.binarySearch
```java
int[] array = {5, -8, 17, 0, 150, 53, -94, 35, -3};

Arrays.sort(array);
System.out.println(Arrays.toString(array));

int index = Arrays.binarySearch(array, 35);
```

#### Методы Collections
```java
List<String> list = new ArrayList<>();

// Разворачивает коллекцию
Сollections.reverse(list);

// Перемешивает коллекцию
Collections.shuffle(list);
```

### Урок 10 Big O notation
> Сложность алгоритма описывается через **кол-во операций для достижения результата**.
> 
> **Big O** - описывает эффективность алгоритма.
> А именно, зависимость кол-ва выполняемых операций от кол-ва входных данных.

#### Алгоритмические сложности
> При оценке алгоритма всегда рассматривается наихудший кейс.
> Например, если это поиск эл-та в массиве (**O(n)**), то рассматривается кейс, когда этот эл-нт последний в массиве.
> По этому сложность поиска в массиве всегда **O(n)**, а не **O(n/2)**, например.

**O(1)** - *константная сложность* - алгоритм выполняется за постоянное (константное) время. Не зависит от кол-ва входных данных.
**O(n)** - *линейная сложность* - сложность алгоритма растет линейно (прямо пропорционально) с увеличением входных данных.
**O(log n)** - *логарифмическая сложность* - сложность алгоритма растёт логарифмически с увеличением входных данных. (Бинарный поиск - на каждой итерации берется половина элементов массива).

#### Как хранится массив в памяти
> Почему получение по индексу из массива выполняется за **O(1)**?

<font style="color:red">ВАЖНО</font>: В памяти, под массив, выделяется место так, что все его эл-ты находятся рядом. Такая неделимая область памяти выделилась.

**Java** всегда знает первый байт(`firstArrayByte`), с которого начинается массив, а положение любого эл-та массива можно вычислить по формуле:
```java
startByteOfIndex = firstArrayByte + index * sizeOf(arrayType);
```
- `firstArrayByte` - первый байт, с которого начинается массив;
- `index` - индекс искомого эл-та;
- `sizeOf(arrayType)` - размер типа данных массива;
  - `int` = 4 байта
  - `reference data type` = 8 байт
- `startByteOfIndex` - байт, с которого начинается эл-нт массива с индексом = `index`

##### Рассмотрим на примере массива int
> Создадим массив `int` (`1 int = 4 байта`).

```java
int [] array = new int [4];
array[0] = 3;
array[1] = 12;
array[2] = 5;
array[3] = -7;
```

![[Pasted image 20240822005615.png]]
- в памяти нам выделилась память под массив, начиная со `103 байта`
- под массив выделилось **16 байт**, т.к. `length = 4`, а тип данных `int`.

Найдем эл-нт массива с индексом `2`:
`103 + 2 * 4 = 111`
- эл-нт с `index = 2` начинается со **111** байта

##### Reference data type
> Для массива **String** (`reference data type`) все аналогично.

За исключением:
- **1** `ячейка` массива = **8 байт**
- `ячейка` хранит значение ссылки на объект **String**

#### ArrayList
- `get()` - **O(1)**
- `add()` - **O(1)**
  - но если внутренний массив заполнен полностью, то при добалении нового эл-та будет пересоздаваться новый массив большего размера - это **O(n)**.
- `add(index)` - **O(n)**
- `remove()` - **O(1)** (удаление последнего эл-та)
- `remove(index)` - **O(n)**

#### LinkedList
- `get()` - **O(n)**
- `add()` - **O(1)** (добавление в начало или конец)
- `add(index)` - **O(n)**
- `remove(index)` - **O(n)**

### Урок 11 Vector
> `Vector` - устаревший **synchronized** класс. В своей основе содержит массив эл-тов `Object`.

<font style="color:red">НЕ РЕКОМЕНДОВАН</font> для использования.
**synchronized** - работа с `Vector` синхронизирована м/ду потоками. Пока один поток работает с `Vector`, он заблокирован для изменений другими потоками.

```java
Vector<String> vector = new Vector<>();
vector.add("a");
vector.add("b");
vector.add("c");

vector.get(1);
vector.remove(2);
vector.firstElement();
vector.lastElement();
```

### Урок 12 Stack
> `Stack` - устаревший **synchronized** класс. Использует принцип **LIFO**

<font style="color:red">НЕ РЕКОМЕНДОВАН</font> для использования.

`Stack` -  стопка. Как стопка тарелок, последняя положенная сверху тарелка - снимется первой.

В **Java** вложенные методы выполняются в стеке. Метод, который был вызван последним - завершит работу первым.

```java
Stack<String> stack = new Stack<>();
stack.push("method1()");
stack.push("method2()");
stack.push("method3()");

stack.pop();
stack.peek();

while(!stack.isEmpty()) {
	stack.pop();
}
```

#### Методы Stack
- `push` - добавляет эл-нт в  `Stack` на самый верх "стопки"; 
- `pop` - возвращает эл-нт с верхушки "стопки" и сразу его удаляет из `Stack`;
- `peek` - возвращает эл-нт с верхушки "стопки", но **НЕ** удаляет его.
- `isEmpty` - проверяет пустой ли `Stack`.
  Т.к. если вызвать `pop()` на пустом стеке - выбросится `EmptyStackException`. 

### Урок 13 Введение в Map. HashMap
> Эл-тами всех `Map` являются `key - value`.

![[Pasted image 20240821172523.png]]

#### HashMap
> `HashMap`не запоминает порядок добавления эл-тов.

- `key` эл-тов должны быть уникальными
- Если добавляется эл-нт с `key`, который уже есть в `HashMap`, то у этого ключа перезаписывается `value`;
-  `key` может быть **null**;
- `value` эл-тов могут повторяться;
- `value` может быть **null**;

```java
Map<Integer, String> map = new HashMap<>():
map.put(1000, "Zaur Tregulov");
map.put(2000, "Viktoria Seck");
map.put(3000, "Oleg Petrov");

map.putIfAbsent(1000, "Ivan Ivanov");
map.get(1000);
map.remove(3000);
map.containsValue("Viktoria Seck");
map.containsKey(500);

Set<Integer> keys = map.keySet();
Collection<String> values = map.values();
Set<Map.Entry<Integer, String>> entries = map.entrySet();

```

##### Методы HashMap
- `put(key, value)`
- `putIfAbsent(key, value)` - добавляет эл-нт в `HashMap`, если такого эл-та (с таким `key`) **нет**.
- `get(key) -> value`
- `remove(key)`
- `containsValue(value) -> boolean`
- `containsKey(key) -> boolean`
- `keySet() -> Set<K> keys`
- `values() -> Collection<V> values`
- `entrySet() -> Set<Map.Entry<K, V>> entries`

### Урок 14 Методы equals и hashcode
> переопределение `equals` и `hashcode` **корректным** путем очень важно, когда мы работаем с коллекциями. Особенно с коллекциями, которые начинаются на `Hash..`.

В **Java** есть <font style="color:red">контракт</font> : если переопределил `equals`, переопредели и `hashcode`.
- дефолтные реализации `equals` и `hashcode` (из `Object`) работают с адресом объекта в памяти. То есть они работают на основе одних и тех же данных.
- И если переопределяется один из этих методов - второй так же должен переопределяться.

В **Java** `хеширование` - это преобразование любого объекта в `int`.

Коллекции `HashMap` и `HashSet` при поиске и сравнении объектов сначала используют `hashcode`. И только в случае **равенства хешкодов** у объектов, сравнивают эти объекты через `equals`.
Сначала происходит сравнение по `hashcode`, т.к. он работает намного быстрее, чем `equals`.

**Коллизия** - когда разные объекты возвращают одинаковый `hashcode`.
Коллизии возможны, т.к.  `hashcode` возвращает **int**, а он имеет ограниченный размер.
В теории разных объектов чего-либо (например, людей на планете) может быть больше, чем вмещает в себя **int**.

**Простое число** - это число которое делится только на себя и на 1.

Результат нескольких выполнений метода `hashcode` для одного и того же объекта доложен быть одинаковым.

Если, согласно `equals`, два объекта равны, то и `hashcode` этих объектов обязательно **должен быть одинаковым**.

Если, согласно `equals`, два объекта **НЕ** равны, то допускается, что `hashcode` этих объектов может быть одинаковым.

### Урок 15 HashMap (часть 1)
> В основе `HashMap` лежит массив.
> Каждый эл-нт данного массива является структурой `LinkedList`.
> Данные структуры `LinkedList` и заполняются эл-тами, которые мы добавляем в `HashMap`.

Эл-нт внутреннего массива `HashMap` называется **Bucket**. И этот эл-нт является экземпляром класса `Node`.
```java
static class Node<K,V> implements Map.Entry<K,V> {  
    final int hash;  
    final K key;  
    V value;  
    Node<K,V> next;

		...
}
```
- `next` - ссылка на следующий эл-нт в `LinkedList`

#### Внутренняя структура HashMap
> дефолтная длина внутреннего массива = **16**

```java
Map<Student, Double> map = new HashMap<>();
// Student(name, surname, course)
Student st1 = new Student("Zaur", "Tregulov", 4);
Student st2 = new Student("Yuras", "Yats", 3);
Student st3 = new Student("Aria", "Stark", 2);
Student st4 = new Student("Maria", "Sidorova", 5);
Student st5 = new Student("Zaur", "Tregulov", 4);

map.put(st1, 7.5);
map.put(st2, 6.2);
map.put(st3, 9.3);
map.put(st4, 7.2);
map.put(st5, 3.4); // st5.equals(st1) == true

map.put(null, 2.8);
```

##### Как рассчитывается индекс эл-та внутреннего массива ?
1. На основе `key` вычисляется **hashcode**
2. На основе `hashcode` вычисляется (по формуле) индекс во внутреннем массиве
   - например остаток от деления `hashcode` на длину массива:
     **hashcode % array.length**

|key - value|hashcode|Индекс в массиве|
|-----------|--------|----------------|
|st1 `--` 7.5|  368  | 3    (=368 % 16)|
|st2 `--` 6.2|  192  | 7    (=192 % 16)|
|st3 `--` 9.3|-1154  | 7   (=1154 % 16)|
|st4 `--` 7.2|  192  | 7    (=192 % 16)|
|st5 `--` 3.4|  368  | 3    (=368 % 16)|
|            |       |                |
|null `--` 2.8|  0    |        0       |

- разные `key` могут давать одинаковый **hashcode** (коллизия) -> одинаковый индекс в массиве (<font style="color:red">st2</font> и <font style="color:red">st4</font>)
- так же, разный `hashcode` может давать одинаковый индекс в массиве (<font style="color:red">st2</font> и <font style="color:red">st3</font>)

##### Поэтапно рассмотрим добавление студентов в `HashMap`
![[Pasted image 20240822005835.png]]

> Метод `put(key, value)`:

1. **st1** : `index = 3`
   - ячейка с индексом **3** пустая - просто помещаем в нее объект `Node`(<font style="color:red">st1</font>)
2. **st2** : `index = 7`
   - ячейка с индексом **7** пустая -> помещаем в нее `Node`(<font style="color:red">st2</font>)
3. **st3** : `index = 7`
   - ячейка с индексом **7** <font style="color:red">НЕ</font> пустая -> производим сравнение объектов по `hashcode`
   - `st3.hashcode() == st2.hashcode()` = **false**
     (-1154 не равно 192) => `key` **st3** и **st2** тоже не равны
   - проверям есть ли в `Node`(<font style="color:red">st2</font>) ссылка на следующих эл-нт -> `next == null`
   - добавляем **st3** в поле `next` у `Node`(<font style="color:red">st2</font>)
1. **st4** : `index = 7`
   -  ячейка с индексом **7** <font style="color:red">НЕ</font> пустая -> производим сравнение объектов по `hashcode`
   - `st4.hashcode() == st2.hashcode()` = **true**
     (192 равно 192)
   - у **st4** и **st2** `hashcode` равен -> производим сравнение этих объектов по `equals`
   - `st4.equals(st2)` = **false**
   - проверям есть ли в `Node`(<font style="color:red">st2</font>) ссылка на следующих эл-нт -> есть `next == st3`
   - переходим к `Node`(<font style="color:red">st3</font>) и осуществляем те же действия
1. **st5** : `index = 3`
   - ячейка с индексом **3** <font style="color:red">НЕ</font> пустая -> производим сравнение объектов по `hashcode`
   - `st5.hashcode() == st1.hashcode()` = **true**
   - `st5.equals(st1)` = **true**
   - Т.к. **st5** и **st1** одинаковые объекты -> заменяем `key - value` у `Node`
1. **null** : `index = 0`
   - Все эл-ты с `key` == **null** помещаются в `index = 0`

> Метод `get(key)`:
```java
// st6.equals(st3) = true
map.get(st6);
```
 - вычисляем его `hashcode` (-1154)
 - вычисляем `index` (`1154 % 16` = **7**)
 - находим `LinkedList` на индексе **7**
 - берем первый объект (`st2`) и осуществляем проверку по `hashcode`
 - `st6.hashcode() == st2.hashcode()` = **false**
 - проверяем есть ли следующий эл-нт в `LinkedList` -> есть `next == st3`
 - берем `st3` и проверяем по `hashcode`
 - `st6.hashcode() == st3.hashcode()` = **true**
 - `st6.equals(st3)` = **true**
 - объект найден -> возвращаем `value`

> Метод `entrySet()`:

```java
for(Map.Entry<Student, Double> entry : map.entrySet()) {
	entry.getKey();
	entry.getValue();
}
```

### Урок 16 HashMap (часть 2)

###