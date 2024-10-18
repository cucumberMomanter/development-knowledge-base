### Введение
Многопоточность - это принцип построения программы, при котором несколько блоков кода могут выполняться одновременно.

**Цели многопоточности :**
- Производительность
- Concurrency (основная цель)

**Concurrency** - выполнение нескольких задач одновременно.
Например, при работе в Word одновременно происходят следующие процессы:
- ввод текста;
- проверка орфографии;
- автосохранение и т.д.

Если бы эти процессы выполнялись в одном потоке, то Word при вводе текста постоянно подвисал.

#### Multithreading в одноядерных процессорах
> Достигается за счет технологии `Context switch`

Задачи делятся на мелкие подзадачи и процессор постоянно между ними переключается.
В процедуру переключения контекста входит так называемое `планирование задачи` — процесс принятия решения, какой задаче передать управление.

##### Example
> Разобьем сложение от `1` до `1000` на 4 потока:

1. 1 .. 250
2. 251 .. 500
3. 501 .. 750
4. 751 .. 1000

В одноядерном процессоре, будет постоянно переключение между подзадачами:
- `1 + 2 + 3 + 4` - выполняется сложение в первом потоке <font style="color:red">с запоминанием результата</font>;
- `251 + 252` - запоминание и переключение
- `751 + 752 + 753` - -//-
- после может продолжится сложение в первом потоке : `.. + 5 + 6 + 7`
- и т.д.

> Из-за того, что переключение процессора происходит очень быстро м/ду подзадачами - **создается видимость того, что потоки выполняются одновременно**.

Потоки постоянно борются за процесорное время.
И в случае одноядерного процессора `1-н` поток справился бы быстрее, т.к. не было бы временных затрат на переключения м/ду потоками.

В многоядерном процессоре не факт, что будет задействовано все `4` ядра, возможно `2` или вообще `1`. Все зависит от загруженности процессора.

### 2. Способы создания нового потока
#### Наследование от класса Thread
> Логика работы потока прописывается в методе `run()`, но запускаем поток методом `start()`.
 
`run()` автоматически вызывается `JVM`, после того, как был вызван метод `start()` .
<font style="color:red">ВАЖНО:</font> не надо самостаятельно запускать метод `run()` у потока.

```java
public class Example1 {  
    public static void main(String[] args) {  
        MyThread1 mt1 = new MyThread1();  
        MyThread2 mt2 = new MyThread2();  
  
        mt1.start();  
        mt2.start();  
    }  
}  
  
class MyThread1 extends Thread {  
    public void run() {  
        for (int i = 1; i <= 1000; i++) {  
            System.out.println(i);  
        }  
    }  
}
  
class MyThread2 extends Thread {  
    public void run() {  
        for (int i = 1000; i > 0; i--) {  
            System.out.println(i);  
        }  
    }  
}
```
- потоки будут работать параллельно, поочередно захватывая консоль.
- при этом второй поток может захватить консоль первым. Эти потоки не синхронизированы м/ду собой и работают независимо друг от друга. И какой поток начнет работать первым или закончит мы знать не можем.

---
В этом примере на самом деле `3` потока, а не `2`.
Т.к. `main` запускается в отдельном потоке, который создается автоматически.
![[Pasted image 20241003210534.png | 250]]
- при этом `main` поток может завершить работу первым, но `app` не останавливается, она ждет завершения работы всех потоков.
---

#### Имплементация интерфейса Runnable
> Не всегда есть возможность наследоваться от класса `Thread`, т.к. в `java` нет множественного наследования.
> По этому мы можем имплементировать интерфейс `Runnable`. И этот вариант **чаще** используется.
> 
> `Thread` так же имплементирует интерфейс `Runnable`.

```java
public class Example2 {  
    public static void main(String[] args) {  
        Thread mt1 = new Thread(new MyThread3());  
        Thread mt2 = new Thread(new MyThread4());  
          
        mt1.start();  
        mt2.start();  
    }  
}  
  
class MyThread3 implements Runnable {  
    @Override  
    public void run() {  
        //...  
    }  
} 
  
class MyThread4 implements Runnable {  
    @Override  
    public void run() {  
        //...  
    }  
}
```

---
`Runnable` - это функциональный интерфейс `->` его можно записать через :
- анонимный класс
- лямбда-выражение

```java
new Thread(new Runnable() {  
    @Override  
    public void run() {  
        // ....  
    }  
}).start();  
  
new Thread(() -> System.out.println("Yooo")).start();  
```

- мы создали поток, не создавая отдельного класса для этого потока.

---

#### Еще примеры
##### Создание нескольких экземпляров одного потока
> Можно создать несколько экземпляров одного потока и они будут выполнять одну и ту же работу параллельно.

```java
public class Example {  
    public static void main(String[] args) {  
        Thread mt1 = new MyThread(); 
        Thread mt2 = new MyThread(); 
          
        mt1.start();  
        mt2.start();  
    }  
}  
  
class MyThread implements Thread {   
    public void run() {  
        //...  
    }  
} 
```

##### Использование потока метода  `main`
> Т.к. метод `main` выполняется в отдельном потоке `->` этот поток так же можно использовать для выполнения необходимой логики.

**Example 1**
```java
public class Example1 extends Thread {
	public void run() {  
        for (int i = 1; i <= 1000; i++) {  
            System.out.println(i);  
        }  
    }

    public static void main(String[] args) {  
        Example1 thread = new Example1();   
  
        thread.start();

		for (int i = 1000; i > 0; i--) {  
            System.out.println(i);  
        }
    }  
}
```

**Example 2**
```java
public class Example3 {
    public static void main(String[] args) {  
        MyThread thread = new MyThread();   
  
        thread.start();

		for (int i = 1000; i > 0; i--) {  
            System.out.println(i);  
        }
    }  
}

class MyThread extends Thread {
	public void run() {  
        for (int i = 1; i <= 1000; i++) {  
            System.out.println(i);  
        }  
    }
}
```

### 3. Методы Thread

#### Name
> Дефолтное имя   `Thread-` +  порядковый номер

- `setName()` - установить имя потока;
- `getName()` - получить имя протока;

#### Priority
> Дефолтный приоритет `5`.
> Приоритетная шкала от `1` до `10` (где `10` - это наивысший приоритет).

<font style="color:red">ВАЖНО</font>:  ни какой гарантии нет, что поток с наивысшим приоритетом запустится быстрее, чем поток с более низким приоритетом.

В классе есть константы для установки приоритета:
- `Thread.MIN_PRIORITY` - *1*
- `Thread.NORM_PRIORITY` - *5*
- `Thread.MAX_PRIORITY` - *10*

Методы:
- `setPriority` - установить приоритет;
- `getPriority` - получить приоритет;

#### currentThread
`Thread.currentThread()` - возвращает текущий поток.

#### sleep
> Усыпляет поток на переданное кол-во миллисекунд.
> Выбрасывает `InterruptedExeption`.

```java
Thread.sleep(1000);
```

Поток может прервать другой поток (попросить его остановиться). И если прерываемый поток находился в это время в спячке - выбрасывается `InterruptedExeption`.

Пока один поток спит, выполняются другие потоки.
```java
public static void main(String[] args) {  
    Thread thread1 = new Thread(() -> counting());  
    Thread thread2 = new Thread(() -> counting());  
  
    thread1.start();  
    thread2.start();  
  
    System.out.println("End");  
}  
  
public static void counting() {  
    try {  
        for (int i = 1; i <= 3; i++) {  
            System.out.println(Thread.currentThread().getName() + " " + i);
            Thread.sleep(1000);  
        }  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
}

Выведется в консоль:
// End (Может и не на первом месте стоять, но где то в начале)
// Thread-0 1
// Thread-1 1
// Thread-0 2
// Thread-1 2
// Thread-0 3
// Thread-1 3
```
- пока потоки спят - `main-поток` завершит свою работу, но приложение будет ждать завершения работы всех потоков.

#### join
> Поток **В КОТОРОМ** вызывается `join`  будет ждать окончания работы потока **НА** котором был вызван `join`. И только после этого продолжит свою работу.

```java
public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new Thread(() -> counting());  
    Thread thread2 = new Thread(() -> counting());  
  
    thread1.start();
    thread2.start();

	// поток main будет ждать окончания работы thread1 и thread2
	// и только после этого продолжит свое выполнение
	thread1.join();
	thread2.join();
  
    System.out.println("End");
}  
  
public static void counting() {  
    try {  
        for (int i = 1; i <= 3; i++) {  
            System.out.println(Thread.currentThread().getName() + " " + i);
            Thread.sleep(1000);  
        }  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
}

Выведется в консоль:
// Thread-0 1
// Thread-1 1
// Thread-0 2
// Thread-1 2
// Thread-0 3
// Thread-1 3
// End
```
- метод `join` выбрасывает `InterruptException`.

> Есть еще один вариант метода `join` с параметром.
> В параметр передается кол-во миллисекунд по истечению которых необходимо продолжить работу внешнему потоку (`timeout`).

```java
public static void main(String[] args) throws InterruptedException {
	Thread thread1 = new Thread();
	thread1.join(1500);
}
```
- поток `main` продолжит свою работу либо после окончания работы потока `thread1`, либо по истечению `1,5 сек`. Зависит от того, что произойдет быстрее.

#### state
> `getState()`

У потока есть `3` состояния:
- `NEW` - когда поток был создан и еще не  вызван метод `start()`;
- `RUNNEBLE` - после вызова `start()`;
  у него есть два состояния:
  - `ready` - после вызова метода `start()` поток ждет своего запуска ОС.
  - `running` - выполение;
- `TERMINATED` - поток завершил свою работу;

![[Pasted image 20241004200741.png | 500]]

```java
Thread thread = new Thread();
System.out.println("NEW");
thread.start();
System.out.println("RUNNABLE");
thread.join();
System.out.println("TERMINATED");
```

- с выводом состояний надо быть аккуратнее, т.к. пока выводится в консоль статус `RUNNABLE` поток `thread` может быстро закончить работу и уже иметь статус `TERMINATED`.

### Почему не стоит самостоятельно запускать `run()`?
> При использовании метода `run()` напрямую - новый поток **НЕ** будет создан.
> А метод `run()` отработает в том же потоке из которого он запускался.

```java
public class Ex4 implements Runnable {   
    public void run() {  
        System.out.println("Method run. Thread name: "
							        + Thread.currentThread().getName());  
    }  
  
    public static void main(String[] args) {  
        Thread thread = new Thread(new Ex4());  
        thread.start();  
        //thread.run();
  
        System.out.println("Method main. Thread name: "
							        + Thread.currentThread().getName());  
    }  
}
```

При вызове метода `start()` вывод в консоль будет следующий:
```
Method main. Thread name: main
Method run. Thread name: Thread-0
```

При вызове метода `run()` напрямую (вместо `start()`):
```
Method main. Thread name: main
Method run. Thread name: main
```

### 5. Понятия связанные с многопоточностью
#### Concurrency и parallelism (согласованность)
> `Concurrency` означает выполнение сразу нескольких задач.
> В зависимости от процессора `concrurrency` может достигаться разными способами.

---
`Concurrency` -  согласованность, совпадение.
`Concurrent` - согласованный, совпадающий, **параллельный**.

---

Примеры `concurrency`:
1) Петь и кушать
   - в один момент времени можно либо петь, либо кушать.
   - это пример достижения `concurrency` за счет `context switch` (одноядерный процессор).
1) Готовить и говорить по телефону
   - готовить и говорить по телефону можно одновременно.
   - это пример достижения `concurrency` за счет `parallelism`.

`Parallelism` - выполнение потоков параллельно (в одно и то же время).
`Parallelism` - это частный случай `Concurrency`. Достигается за счет использования многоядерных процессоров.

#### Asynchronous и synchronous
> В синхронном программировании задачи выполняются **последовательно** друг за другом.

---
**Example. Необходимо написать `2` письма.**

Сначала пишете `1-e` письмо, потом `2-е`.
Письма пишутся последовательно, друг за другом.
Пока `1-e` письмо не напишите, вы не можете приступить ко второму.
- это пример `synchronous` программирования.

```java
System.out.println("Hello");
System.out.println("Yuras");
System.out.println("!");
```
- этот код будет выполняться последовательно (синхронно).

---

> В асинхронном программировании каждая следующая задача <font style="color:red">НЕ</font> ждет окончания выполнения предыдущей.
> Асинхронное программирование помогает достичь `concurrency`.

---
**Example. Делаем завтрак и стираем одежду**

Можно включить стиральную машинку и пока она стирает - делать завтрак.
Эти процессы будут выполняться одновременно (параллельно).
- это пример `asynchronous` программирования.
- в асинхронном программировании пока одна работа выполняется `->` мы можем переключится на выполнение другой работы.

---

##### Асинхронное программирование
```java
public static void main(String[] args) throws InterruptedException {  
    Thread thread1 = new Thread(() -> counting());  
    Thread thread2 = new Thread(() -> counting());  
  
    thread1.start();  
    thread2.start();  
}  
  
public static void counting() {  
    try {  
        for (int i = 1; i <= 10; i++) {  
            System.out.println(Thread.currentThread().getName() + " " + i);  
            Thread.sleep(100);  
        }  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
}
```

```md
Запуск 1             Запуск 2

Thread-1 1           Thread-0 1
Thread-0 1           Thread-1 1
Thread-1 2           Thread-1 2
Thread-0 2           Thread-0 2
Thread-0 3           Thread-1 3
Thread-1 3           Thread-0 3
Thread-0 4           Thread-1 4
Thread-1 4           Thread-0 4
```

При нескольких запусках программы `output` будет разным. Где-то начнет работу первым `thread1`, где-то `thread2`.

**Почему так происходит ?**
Наши потоки работаю `асинхронно`, у них нет никакого порядка в их исполнении.
`Output` - непредсказуем.
Такое поведение программы называется <font style="color:red">не детерминированное</font>.

В программах, где используется `асинхронное` программирование - разработчик не может определить порядок выполнения потоков.

### 6. Ключевое слово volatile
> Ключевое слово `volatile` используется для пометки переменной, хранящейся **только** в основной памяти (`main memory`).
> В кеше `CPU` ---  `volatile` переменные не хранятся `->` рассинхронизации значения переменной в разных потоках **НЕ** будет.

#### Разберем volatile на примере
> Из потока main останавливаем цикл в другом потоке

```java
public class VolatileEx extends Thread {  
    boolean b = true;  
  
    public void run() {  
        long counter = 0;  
  
        while (b) {  
            counter++;  
        }  
        System.out.println("Loop is finished. Counter = " + counter);  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        VolatileEx thread = new VolatileEx();  
        thread.start();
        // Усыпляем поток main на 3 секунды, чтобы дать поработать потоку tread
        Thread.sleep(3000);  
        System.out.println("After 3 seconds");
        // Завершаем цикл в потоке tread
        thread.b = false;
        // Дожидаемся завершения работы потока tread
        thread.join();  
        System.out.println("End of program");  
    }  
}
```

Ожидание:
```md
After 3 seconds
Loop is finished. Counter = ....
End of program
```

Реальность:
```md
After 3 seconds

и программа не останавливается..
```

**Почему так происходит?**

В многопоточных app, где потоки работают с переменными, **КАЖДЫЙ** поток может **скопировать значение переменной из общей памяти** (`main memory`) <font style="color:red">в кеш процессора</font> (`CPU cache`).
Это производится для быстрого доступа, `CPU cache` - это очень быстрая область памяти.

![[Pasted image 20241005200116.png | 550]]
- потоки `main` и `thread` копируют начальное значение `b` в кеш процессора;
- в какой то момент времени в потоке `main` меняется значение `b = false`;
- новое значение `b` хранится в `CPU 2 cache` потока `main`.
  И мы не знаем когда новое значение попадет в `main memory`, а после в `CPU 1 cache` потока `thread`.

<font style="color:red">Для решения этой проблемы</font> используем ключевое слово `volatile` для переменной `b`.

#### Условие использование `volatile`
<font style="color:red">ВАЖНО</font>
Для синхронизации значения переменной м/ду потоками, `volatile` используется тогда, когда только <font style="color:red">один поток может изменять</font> значение этой переменной, а <font style="color:red">остальные потоки</font> могут его <font style="color:red">только читать</font>.

##### Рассмотрим пример
> Каждый поток выполняет операцию `a++`.
> Переменная `a` - `volatile` -> хранится только в основной памяти.

`a++` это по факту `a = a + 1`.
Чтобы выполнить эту операцию, потоку необходимо выполнить следующие действия:
1. Прочесть значение `a` из `main memory`
2. Увеличить полученное значение на `1`;
3. Записать новое значение `a` в `main memory`

![[Pasted image 20241005210329.png | 550]]

Т.к. потоки работают параллельно, может получиться так, что оба потока вычитают одно и тоже значение `a` (например `5`), увеличат его и поочередно запишут `6`.
Хотя в таком случае ожидается значение  `7`.

### 7. Data race. Synchronized methods
> `Data race` - это проблема, которая может возникнуть когда два или более потоков обращаются к одной и той же переменной и как минимум `1` поток ее изменяет.
> 
> `Data race` - гонка данных, потоки как будто бы участвуют в гонке и пытаются побыстрее проделать свои операции.

#### Рассмотри проблему `Data race`на примере
> `3` потока.
> В каждом потоке, в цикле, срабатывает метод `increment()` (`count++` и `sout`).

```java
public class Ex7 {  
    public static void main(String[] args) {  
        Thread th1 = new Thread(new MyRunnableImpl());  
        Thread th2 = new Thread(new MyRunnableImpl());  
        Thread th3 = new Thread(new MyRunnableImpl());  
  
        th1.start();  
        th2.start();  
        th3.start();  
    }  
}  
  
class Counter {  
    public static int count = 0;  
}  
  
class MyRunnableImpl implements Runnable {  
    public void increment() {  
        Counter.count++;  
        System.out.printf(Counter.count + " ");  
    }  
  
    public void run() {  
        for (int i = 0; i < 3; i++) {  
            increment();  
        }  
    }  
}
```

`Output`
```md
2 4 5 2 6 3 8 9 7
```
- в  `output` нет ни какой последовательности вывода чисел, иногда они даже повторяются;
- каждый новый запуск дает другой `output`.

##### Почему так происходит?
> Здесь нет никакой синхронизации м/ду потоками.
> Они одновременно работают с одними и те ме же данными.

<font style="color:red">ПРИМЕЧАНИЕ</font>:
- `volatile` тут не подходит, т.к. больше `1` потока меняют данные.
- потоки могут работать с разной скоростью, т.к. им могут выделяться ядра процессора с разной загруженностью.

> В нашем примере представим, что поток `Th2` самый быстрый, а `Th3` самый медлeнный.

![[Pasted image 20241009212650.png | 500]]

- все потоки прочли значение `3`;
- `Th1` увеличил значение до `4` и записал в память, но еще не вывел в консоль.
- `Th2` успел проделать все три итерации и записал в `MM` значение `6`
  - увеличил до `4` -> записал в `MM` -> вывел на экран `4`;
  - прочитал `4` -> увеличил до `5` -> записал в `MM` -> вывел на экран `5`;
  - прочитал `5` -> увеличил до `6` -> записал в `MM` -> вывел на экран `6`;
- `Th1` вычитал `6` из `MM` для вывода в консоль и вывел `6`(а не `4`);
- `Th3` увеличил до `4` -> записал в `MM` -> вывел на экран `4`;

На этом этапе поток `Th2` работу, остались только `Th1` и `Th3`.
Далее ситуация может развиваться, например, так:
- если `Th1` успел вычитать `6` до того, как `Th3` записал `4` в `MM`, то он увеличит значение до `7` и запишет его;
- если не успел и вычитал `4`, то увеличит и запишет `5`;
- или же поток `Th3` быстрее сработает и отработает еще раз.

---
<font style="color:red">ВАЖНО</font>
- данная проблема проявляется и лучше видна на большем кол-ве итераций в цикле.

В примере выше:
- больше заметна проблема вывода в консоль (числа выводятся не по очереди и повторяются).
- при увеличении переменной `counter` потоки практически не конфликтуют и не перезатирают друг друга (даже на большом кол-ве итераций).
- это происходит из-за того, что в каждой итерации есть операция вывода в консоль (`sout`).
  **Вероятно**, получение доступа к консоли намного медленнее, чем увеличение значения `counter`. И `=>` `counter` успевает быстро изменить свое значение, а тормозят и конфликтуют потоки именно на выводе в консоль.
- **Видимо**, поток быстро вычитывает значение переменной из `MM` и долго ждет доступа к консоли.
  
---
Если из итерации цикла убрать операцию `sout` - картина изменится. Потоки начинают конфликтовать при `counter++` и перезаписывать друг друга.

Это видно в следующем примере:
```java
public class Ex8 {  
    static int counter = 0;  
    static void increment() {counter++;}  
  
    public static void main(String[] args) throws InterruptedException {  
        Thread thread1 = new Thread(new R());  
        Thread thread2 = new Thread(new R());  
  
        thread1.start();  
        thread2.start();  
  
        thread1.join();  
        thread2.join();  
  
        System.out.println(counter);  
    }  
}  
  
class R implements Runnable {  
    public void run() {  
        for (int i = 0; i < 1000; i++) {  
            Ex8.increment();  
        }  
    }  
}
```

Ожидаемый `output`: 2000.
Но он далеко не всегда такой.

##### Решение проблемы синхронизации потоков
> Когда несколько потоков хотят работать с одной переменной, мы должны быть уверенны, что в одно и то же время **чтение** переменной **и запись** в нее нового значения **осуществляется только одним потоком**.

Это достигается за счет блокирования доступа (`lock`) к необходимому участку кода (например, к `методу`).

Блокировка доступа к методу достигается за счет ключевого слова `synchronized`.
`synchronized` нельзя применить к переменной.

Если с `synchronized` методом уже работает поток `Th1` `->` все остальные потоки будут стоять и ждать, пока `Th1` закончит свою работу с этим методом.

```java
public static synchronized void increment() {
	counter++;
}
```
- таким образом в один и тот же момент времени доступ к `counter` есть только у одного потока. В этом и заключается принцип синхронизации.

---
`volatile` - используется когда, только `1` поток хочет изменять данные.
`synchronized` - используется когда `> 1` потока хотят изменять данные.

---

### 8. Понятие "монитор". Synchronized blocks
> `Монитор` - это сущность/механизм, благодаря которому достигается корректная работа при синхронизации.
> В `Java` у каждого класса и объекта есть **привязанный к нему `монитор`**.

Любая **блокировка** с помощью `synchronized` идет (ставится) на <font style="color:red">объекте</font> или на <font style="color:red">классе</font>, а **НЕ** на куске какого-л кода.
Для синхронизации метода используются мониторы объекта или класса.

---
Синхронизация идет по монитору, а монитор это неотъемлемая часть каждого объекта и каждого класса в `Java`.

---

#### Монитор
> `Монитор` есть у каждого объекта и класса.
> `Монитор` может иметь статус `свободен`/`занят`.
> В одно и то же время `монитор` может заниматься только одним потоком.

![[Pasted image 20241007131221.png | 500]]
- когда поток `Thread2` заходит в область кода, помеченную `synchronized`, монитор объекта/класса переходит в состояние `занят`.
- когда монитор становится `занят`, на этот кусок кода ставится `lock` -> мы запираем двери для остальных потоков.
- и пока `Thread2` не освободит монитор `->` все остальные потоки будут ждать.
- после освобождения `Thread2` монитора `->` остальные потоки будут бороться кто из них первый захватит монитор.

#### Синхронизированные блоки
```java
// Синхронизированный блок объекта
// Можно поместить любой объект, в том числе и this
synchronized(this) {
	//...some code...
}

// Синхронизированный блок класса
synchronized(MyClass.class) {
	//...some code...
}
```

> Когда метод, который принадлежит объекту (**НЕ** `static`), помечается `synchronized` -> блокируется монитор объекта (`this`).

Эти методы эквивалентны:
```java
public class Example {
	public static int count = 0;
	
	public synchronised void increment() {
		count++;
		System.out.println(count);
	}
	
	// У метода объекта по умолчанию synchronized(this) {}
	
	public void increment() {
		synchronized(this) {
			count++;
			System.out.println(count);
		}
	}
}
```

> Когда метод, помеченный `synchronized`, принадлежит классу (`static`) -> блокируется монитор класса.

Эти методы эквивалентны:
```java
public class Example {
	public static int count = 0;
	
	public static synchronised void increment() {
		count++;
		System.out.println(count);
	}
	
	public static void increment() {
		synchronized(Example.class) {
			count++;
			System.out.println(count);
		}
	}
}
```

> `synchronized` блоки можно писать внутри методов

```java
public void method() {
	// some code
	
	Example ex = new Example();
	synchronized(ex) {
		ex.increment();
	}
	
	// some code
}
```

#### Синхронизация работы нескольких методов
> Мы можем синхронизировать несколько методов по одному объекту.
> И получается, что на несколько методов будет только один `монитор`.

![[Pasted image 20241007150745.png | 500]]
- пока монитор объекта `object1` будет занят `Thread1`, методы `method1` и `method2` будут недоступны для других потоков.
- когда `Thread1` освободит монитор `object1`, его захватит `Thread2` через `method2`, и метод `method1` станет так же недоступен для остальных потоков.

##### Рассмотрим на примере входящих звонков
> На телефон могут звонить по:
> - мобильной связи
> - скайпу
> - What's app
> 
> Должно быть 3 разных потока, по одному на каждый канал связи.
> Если уже ведется разговор по одному из каналов связи и приходит звонок по другому `->` последний должен дождаться окончания текущего разговора.

```java
public class Example {  
  
    synchronized void mobileCall() {  
        System.out.println("Mobile call start");  
        sleep(3000);  
        System.out.println("Mobile call end");  
    }  
  
    synchronized void skypeCall() {  
        System.out.println("Skype call start");  
        sleep(5000);  
        System.out.println("Skype call end");  
    }  
  
    synchronized void whatsappCall() {  
        System.out.println("Whatsapp call start");  
        sleep(7000);  
        System.out.println("Whatsapp call end");  
    }  
  
    private void sleep(long millis) {  
        try {  
            Thread.sleep(millis);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    }  
  
    public static void main(String[] args) {  
        Thread thread1 = new Thread(new MobileCallRunImpl());  
        Thread thread2 = new Thread(new SkypeCallRunImpl());  
        Thread thread3 = new Thread(new WhatsappCallRunImpl());  
  
        thread1.start();  
        thread2.start();  
        thread3.start();  
    }  
}  
  
  
class MobileCallRunImpl implements Runnable {  
    public void run() {  
        new Example().mobileCall();  
    }  
}  
  
class SkypeCallRunImpl implements Runnable {  
    public void run() {  
        new Example().skypeCall();  
    }  
}  
  
class WhatsappCallRunImpl implements Runnable {  
    public void run() {  
        new Example().whatsappCall();  
    }  
}
```
`Output`
```md
Mobile call start
Whatsapp call start
Skype call start
Mobile call end
Skype call end
Whatsapp call end
```
- `output` не такой, как ожидался. Все звонки начали одновременно, друг за другом. 

<font style="color:red">ОШИБКА</font> :
В данном примере используется синхронизация метода по объекту, но в потоках используются **РАЗНЫЕ** объекты `Example`.
И `=>` потоки между собой не синхронизированы, т.к. у каждого объекта свой монитор.

Для решения этой проблемы необходимо **синхронизировать все потоки по одному объекту**.

> Изменения коснутся только класса `Example`:

```java
public class Example {
  
    public static final Object lock = new Object();
  
    void mobileCall() {  
        synchronized (lock) {  
            System.out.println("Mobile call start");  
            sleep(3000);  
            System.out.println("Mobile call end");  
        }  
    }  
  
    void skypeCall() {  
        synchronized (lock) {  
            System.out.println("Skype call start");  
            sleep(5000);  
            System.out.println("Skype call end");  
        }  
    }  
  
    void whatsappCall() {  
        synchronized (lock) {  
            System.out.println("Whatsapp call start");  
            sleep(7000);  
            System.out.println("Whatsapp call end");  
        }  
    }  
							.
							.
							.
}
```
- мы просто добавили `static` объект для использования его монитора при синхронизации методов.
- Обычно именно такой формат блокировочного объекта и используется:
  ```java
	static final Object lock = new Object();
  ```

---
<font style="color:red">ВАЖНО</font> : при синхронизации потоков по объекту необходимо следить, что в этих потоках используется один и тот же объект.

Так же невозможно синхронизировать конструктор, `JVM` гарантирует, что конструктор может обрабатываться, в одно и то же время, только одним потоком.

---

### 9. Методы wait и notify
> Для извещения потоком других потоков о своих действиях часто используются следующие методы:

`wait` - освобождает монитор и переводит вызывающий поток в состояние ожидания до тех пор, пока другой поток не вызовет метод `notify()`.
 
 `notify` - **НЕ** освобождает монитор и будит поток, у которого ранее был вызван метод `wait()`.
 
 `notifyAll` - **НЕ** освобождает монитор и будит все потоки, у которых ранее был вызван метод `wait()`.

#### Рассмотрим работу методов на примере 
> Есть магазин булочек (`max = 5` булочек в один и тот же момент времени).
> 
> **Пекарня** в день может печь `max = 10` булочек.
> Если магазин заполнен, то пекарня ждет пока покупатель купит хотя бы `1` булочку.
> 
> **Покупатель** покупает до тех пор, пока в магазине есть булочки в наличии.
> Если магазин пуст, то покупатель ждет, пока пекарня завезет хотя бы `1` булочку.
> 
> Пекарня и Покупатель работают каждый в своем потоке.

![[Pasted image 20241009214222.png]]
- Если в `Market` `0` булочек `->` покупатель вызывает `wait` и ждет пока пекарня доставит хотя бы одну булочку и разбудит его (т.е. вызовет `notify`).
- Если пекарня доставляет булочки в `Market` она вызывает `notify` и будит поток покупателя.
- Если `Market` полон и пекарня не может доставить ни одной булочки `->` она вызывает `wait` и ждет пока покупатель заберет из `Market` хотя бы одну булочку и разбудит ее `notify`.
- Если покупатель забирает  `1` булочку `->` он будит поток пекарни, уведомляя, что в `Market` освободилось место.

```java
public class WaitNotifyEx {  
    public static void main(String[] args) {  
        Market market = new Market();  
        Producer producer = new Producer(market);  
        Consumer consumer = new Consumer(market);  
  
        Thread thread1 = new Thread(producer);  
        Thread thread2 = new Thread(consumer);  
  
        thread1.start();  
        thread2.start();  
    }  
}  
  
class Market {  
    private int breadCount = 0;  
  
    public synchronized void getBread() {  
        while(breadCount < 1) {  
            try {  
                wait();  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
  
        breadCount--;  
        System.out.println("Потребитель купил 1 булочку");  
        System.out.println("Булочек в магазине: " + breadCount);  
        notify();  
    }  
  
    public synchronized void putBread() {  
        while(breadCount >= 5) {  
            try {  
                wait();  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
  
        breadCount++;  
        System.out.println("Пекарня испекла 1 булочку");  
        System.out.println("Булочек в магазине: " + breadCount);  
        notify();  
    }  
}  
  
class Producer implements Runnable {  
  
    private final Market market;  
  
    public Producer(Market market) {  
        this.market = market;  
    }  
  
    public void run() {  
        for (int i = 0; i < 10; i++) {  
            market.putBread();  
        }  
    }  
}  
  
class Consumer implements Runnable {  
    private final Market market;  
  
    public Consumer(Market market) {  
        this.market = market;  
    }  
  
    public void run() {  
        for (int i = 0; i < 10; i++) {  
            market.getBread();  
        }  
    }  
}
```
- вызов `notify` может быть и холостым, ведь другой поток в этот момент может быть так же активен (не в состоянии ожидания).

---
<font style="color:red">ВАЖНО ПОНЯТЬ</font>
- методы `wait` и `notify` вызываются только из синхронизированного контекста. Т.е. из `synchronized` блока или метода.
- `synchronized` блоки/методы вызываются на объекте, который используется для создания `lock` (в примере выше это `this`). `wait` и `notify` вызываются на **ЭТОМ ЖЕ** объекте.
- Методы `wait` и `notify` работают для **потоков**, которые **синхронизируются** по **монитору объекта**, на котором эти **методы вызываются**.

Пример с другим объектом:
```java
class Market {  
    private final Object lock = new Object();  
    private int breadCount = 0;  
  
    public void getBread() {  
        synchronized (lock) {  
            while (breadCount < 1) {  
                try {  
                    lock.wait();  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }  
  
            breadCount--;  
            System.out.println("Потребитель купил 1 булочку");  
            System.out.println("Булочек в магазине: " + breadCount);  
            lock.notify();  
        }  
    }  
  
    public void putBread() {  
        synchronized (lock) {  
            while (breadCount >= 5) {  
                try {  
                    lock.wait();  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }  
  
            breadCount++;  
            System.out.println("Пекарня испекла 1 булочку");  
            System.out.println("Булочек в магазине: " + breadCount);  
            lock.notify();  
        }  
    }  
}
```
- поток в котором был вызван `lock.wait()` уходит в ожидание на объекте `lock`.
- следующий поток который вызовет `lock.notify()` выведет из ожидания предыдущий поток, который **НА ЭТОМ ЖЕ** объекте ушел в ожидание.

---

**ТЕЗИСЫ**:
- `wait()` -  освобождает монитор;
- `notify()` -  не освобождает монитор;
- метод `wait(long millis)` ждет либо вызова `notify` другим потоком, либо истечения переданного `millis`.
- в **javadoc** рекомендуется `wait()` вызывать не в `if`, а в `while`, т.к. в очень редких случаях поток может проснуться без `notify` и тогда мы должны быть уверены, что условие `while` перепроверится еще раз.


### 10. Возможные ситуации в многопоточном программировании
#### DeadLock
> Ситуация, когда 2 или более потоков залочены навсегда, ожидают друг друга и ничего не делают.

---
Это возникает, когда <font style="color:red">несколько потоков</font> используют <font style="color:red">синхронизацию на нескольких объектах</font> **НЕ** в одинаковом порядке.

**Решение**: необходимо чтобы потоки <font style="color:red">осуществляли синхронизацию</font> по нескольким объектам <font style="color:red">в одинаковом порядке</font>.

---

Рассмотрим пример в котором:
- `Thread10` пытается сначала захватить монитор объекта `lock1`, а внутри этого `synchronyzed` блока пытается захватить монитор объекта `lock2`.
- А в `Thread20` наоборот, сначала пытается захватить `lock2`, а потом `lock1`.

```java
public class DeadLockExample {  
    public static final Object lock1 = new Object();  
    public static final Object lock2 = new Object();  
  
    public static void main(String[] args) {  
        Thread thread10 = new Thread10();  
        Thread thread20 = new Thread20();  
  
        thread10.start();  
        thread20.start();  
    }  
}  
  
class Thread10 extends Thread {  
    public void run() {  
        System.out.println("Thread10: попытка захвата lock1");  
        synchronized (DeadLockExample.lock1) {  
            System.out.println("Thread10: захват lock1");  
            System.out.println("Thread10: попытка захвата lock2");  
            synchronized (DeadLockExample.lock2) {  
                System.out.println("Thread10: захват lock1 и lock2");  
            }  
        }  
    }  
}  
  
class Thread20 extends Thread {  
    public void run() {  
        System.out.println("Thread20: попытка захвата lock2");  
        synchronized (DeadLockExample.lock2) {  
            System.out.println("Thread20: захват lock2");  
            System.out.println("Thread20: попытка захвата lock1");  
            synchronized (DeadLockExample.lock1) {  
                System.out.println("Thread20: захват lock1 и lock2");  
            }  
        }  
    }  
}
```

- Данный пример может отработать нормально, когда один из потоков первым успеет захватить оба объекта и `lock1` и `lock2`.
- **`Deadlock`** возникает когда `Thread10` успевает захватить `lock1`, но не успевает `lock2`. При этом `Thread20` захватывает `lock2`, но не успевает `lock1`.
- И эти потоки подвисают в бесконечном ожидании друг друга.

`Output`
```md
Thread10: попытка захвата lock1
Thread10: захват lock1
Thread20: попытка захвата lock2
Thread20: захват lock2
Thread20: попытка захвата lock1
Thread10: попытка захвата lock2

... И программа навсегда подвисает
```


#### Livelock
> ситуация, когда 2 или более потоков залочены навсегда, ожидают друг друга, проделывают какую-то работу, но без какого-л прогресса.

**Пример 1**
Есть узкий мост, с одной полосой, по которому может ездить два типа машин:
- обычные машины
- машины на автопилоте

У машин с автопилотом заложено, что если они встречают встречку на мосту, они сдают назад и пропускают машину, потом снова заезжают на мост.
Но если на мосту встретятся две машины с автопилотом, то они бесконечно будут сдавать назад, а потом снова встречаться на мосту.

**Пример 2**
Есть два потока, работающих с одним ресурсом.
`Thread1` что-то пишет в этот ресурс, а `Thread2` постоянно это стирает.
`Thread1` видит, что ресурс пуст и снова пишет, а `Thread2` видит, что ресурс не пустой и стирает написанное.

#### Lock starvation
> ситуация, когда менее приоритетные потоки все время находятся в ожидании своего запуска.

<br>

### 11. Lock и ReentrantLock
> `Lock` - интерфейс, который имплементируется классом `ReentrantLock`.
> 
> Так же как и ключевое слово `synchronized`, `Lock` нужен для достижения синхронизации м/ду потоками.

#### Методы Lock
`lock()` -  активируется блокировка, после которой кодом (который идет после метода `lock()`) может воспользоваться только текущий поток.

`unlock()` - снимается блокировка.

`tryLock() -> boolean` - позволяет делать `lock` когда ресурс свободен или не останавливаться и делать что-то другое, когда ресурс занят.

#### Пример использования lock() и unlock()
> Эти два класса, с использованием `synchronized` и с использованием `Lock` полностью идентичны.

**`synchronized`** :
```java
class Call {  
    private final Object lock = new Object();  
  
    public void callMobile() {
	    synchronized(lock) {
	        try {   
	            System.out.println("Mobile call start");  
	            Thread.sleep(3000);  
	            System.out.println("Mobile call end");  
	        } catch (InterruptedException e) {  
	            e.printStackTrace();  
	        }
        }
    }  
  
    public void callSkype() {
	    synchronized(lock) {
	        try {
	            System.out.println("Skype call start");  
	            Thread.sleep(5000);  
	            System.out.println("Skype call end");  
	        } catch (InterruptedException e) {  
	            e.printStackTrace();  
	        }
	    }
    }
```
- в `synchronized` конструкции не нужно заботится о том, чтобы снять лок (`lock.unlock()`), это происходит автоматически.

**`Lock`** :
```java
class Call {  
    private Lock lock = new ReentrantLock();  
  
    public void callMobile() {  
        try {  
            lock.lock();  
            System.out.println("Mobile call start");  
            Thread.sleep(3000);  
            System.out.println("Mobile call end");  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        } finally {  
            lock.unlock();  
        }  
    }  
  
    public void callSkype() {  
        try {  
            lock.lock();  
            System.out.println("Skype call start");  
            Thread.sleep(5000);  
            System.out.println("Skype call end");  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        } finally {  
            lock.unlock();  
        }  
    }
}
```
- `unlock()` - там где возможен выброс исключения, необходимо вызывать в `finally` блоке, чтобы он снимался в любом случае.

#### Пример использования tryLock()
> Если монитор объекта свободен `tryLock()` ставит на него блокировку и возвращает `true`. Т.е. срабатывает как обычный `lock()`.
> 
> Если занят `->` возвращает `false`. И поток продолжает выполнять остальной код, после блока с синхронизацией.

Данная логика реализуется за счет использования `if`-а.

```java
class Employee extends Thread {  
    private String name;  
    private Lock lock;  
  
    public Employee(String name, Lock lock) {  
        this.name = name;  
        this.lock = lock;  
        // поток сразу запускается при создании объекта  
        this.start();  
    }  
  
    public void run() {  
        if (lock.tryLock()) {  
            try {  
//            System.out.println(name + " ждет..");  
//            lock.lock();  
                System.out.println(name + " пользуется банкоматом");  
                Thread.sleep(2000);  
                System.out.println(name + " закончил(-а) с банкоматом");  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            } finally {  
                lock.unlock();  
            }  
        } else {  
            System.out.println(name + "не хочет стоять в ожереди");  
        }  
    }  
}
```

#### Разница `lock()` и `tryLock()`
- `lock()` -  если блокировка активирована уже другим потоком, то все остальные потоки останавливаются и ждут пока она будет снята. Т.е. ничего не делают.
- `tryLock()` - при активной блокировке не останавливается, а продолжает выполнять другую работу.

### 12. Daemon потоки
> `Daemon` потоки предназначены для выполнения фоновых задач и оказания различных сервисов `User` потокам.
> 
> При завершении работы последнего `User` потока программа завершает свое выполнение, не дожидаясь окончания работы `Daemon` потоков.
> 
> Исходя из логики, если `User` потоков нет, то никакие сервисы им оказывать не нужно. По этому можно завершать программу, не дожидаясь окончания работы `Daemon` потока.

#### Тезисы
- `Daemon` потоки полезны для поддержания `back ground` заданий.
- Например, большинство потоков `JVM` - это `Daemon` потоки. Они занимаются сборкой мусора, освобождают память и т.д.

#### User Thread
> это обычный поток.
> 
> Программа ждет завершения работы всех `user thread` и только после этого завершает работу сама.

#### Создание Daemon потока
После создания потока, **НО** перед его запуском необходимо вызвать на потоке метод `setDaemon()`.

```java
Thread thread = new Thread();
thread.setDaemon(true);
thread.start();

thread.isDaemon();
```

- если вызвать на потоке `setDaemon()` после его запуска выбросится `IllegalThreadStateException`.
- проверить является ли поток `Daemon` можно методом `isDaemon()`;

#### Example
> `User` поток завершит работу первым и программа завершится не дожидаясь окончания работы `Daemon` потока.

```java
public class DaemonExample {  
    public static void main(String[] args) {  
        System.out.println("Main thread start");  
  
        Thread userThread = new UserThread();  
        userThread.setName("user_thread");  
  
        Thread daemonThread = new DaemonThread();  
        daemonThread.setDaemon(true);  
        daemonThread.setName("daemon_thread");  
  
        userThread.start();  
        daemonThread.start();  
  
        System.out.println("Main thread end");  
    }  
}  
  
class UserThread extends Thread {  
    public void run() {  
        System.out.println(Thread.currentThread().getName()  
                                    + " is daemon: " + isDaemon());  

        for (char i = 'A'; i < 'J'; i++) {  
            try {  
                Thread.sleep(300);  
                System.out.println(i);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
}  
  
class DaemonThread extends Thread {  
    public void run() {  
        System.out.println(Thread.currentThread().getName()  
                                    + " is daemon: " + isDaemon());  
  
        for (int i = 1; i < 1000; i++) {  
            try {  
                Thread.sleep(100);  
                System.out.println(i);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
}
```

### 13.  Прерывание потоков
> Мы можем послать сигнал потоку, что мы хотим его прервать.
> Так же у мы можем в самом потоке проверить, хотят ли его прервать.
> 
> Программист сам решает, что делать, если проверка показала, что поток хотят прервать.

#### Методы
##### `stop()` (Deprecated)
> В старых версиях `Java` прерывали с помощью метода `stop`.
> Этот метод **грубо** прерывал поток и некоторые процессы оставались в непонятном и не оконченном состоянии.

```java
Thread thread = new Thread();
thread.start();
		.
thread.stop();
```

##### `interrupt()`
> Метод не прерывает поток, а делает запрос на прерывание.

Поток в котором вызывается это метод, посылает сигнал потоку **НА** котором вызывается метод `interrupt`, что он хочет его прервать.

##### `isInterrupted() -> boolean`
> Поток проверяет хотят ли его прервать.

А прерываться ему или нет уже решаем сами.

#### Examples
##### Пример использования методов `interrupt()` и `isInterrupted()`
```java
public class InterruptionEx {  
    public static void main(String[] args) throws InterruptedException {  
        System.out.println("Main thread starts");  
        Thread thread1 = new InterruptedThread();  
        thread1.start();  
        Thread.sleep(2000);  
        // поток main посылает сигнал, что хочет прервать поток thread1  
        thread1.interrupt();  
  
        thread1.join();  
        System.out.println("Main thread ends");  
    }  
}  
  
class InterruptedThread extends Thread {  
    private double sqrtSum = 0;  
  
    public void run() {  
        for (int i = 1; i < 1_000_000_000; i++) {  
            // проверяем хочет ли кто-то прервать поток  
            if (isInterrupted()) {  
                System.out.println("Поток хотят прервать");  
                // убеждаемся, что у всех объектов нормальное состояние
                // и завершаем поток  
                // т.е. подготавливаем поток к нормальному завершению
                return;  
            }  
  
            sqrtSum += Math.sqrt(i);  
        }  
  
        System.out.println(sqrtSum);  
    }  
}
```
- при использовании `isInterrupted` мы сами решаем прерывать поток или нет.

##### Пример с выбрасыванием `InterruptedException`
> В таких методах, как `sleep` или `join` выбрасывается `InterruptedException` для того, чтобы мы сразу могли узнать, что поток пытаются прервать.

Например, поток `thread1` засыпает на час, а его попытался прервать поток `thread2` через `3 минуты` после того, как заснул `thread1`.
- чтобы не ждать пока пройдет час и `thread1` сам проснется и узнает, что его хотят прервать `->` выбрасывается `InterruptedException`.
- мы можем сразу же обработать `InterruptedException` внутри потока и сделать необходимые нам действия (прервать поток, продолжить сон и т.д.)

```java
class InterruptedThread extends Thread {  
    private double sqrtSum = 0;  
  
    public void run() {  
        for (int i = 1; i < 1_000_000_000; i++) {  
            // проверяем хочет ли кто-то прервать поток  
            if (isInterrupted()) {  
                System.out.println("Поток хотят прервать");  
                // убеждаемся, что у всех объектов нормальное состояние
                // и завершаем поток  
                // т.е. подготавливаем поток к нормальному завершению
                return;  
            }  
  
            try {  
                sleep(100);  
            } catch (InterruptedException e) {  
                System.out.println("Поток хотят прервать во время сна");
                // принимает решение прервать поток в этом случае
                return;  
            }  
  
            sqrtSum += Math.sqrt(i);  
        }  
  
        System.out.println(sqrtSum);  
    }  
}
```
- по сравнению со сном в `100` миллисекунд, остальные операции выполняются очень быстро и большее время наш поток будет спать `->` с большей вероятностью вызов метода `interrupt()` из `main` потока придется на фазу сна.

### 14. Thread pool и ExecutorService (`part 1`)
На практике часто приходится создавать большое кол-во потоков.
А создание потока очень затратная операция. На нее уходит не мало времени и при создании потока происходит множество процессов в `JVM`  и `ОС`. Да и управлять отдельно созданными потоками неудобно.
На помощь приходит механизм `Thread pool-ов`.

> `Thread pool - это множество потоков, каждый из которых предназначен для выполнения той или иной задачи.`

`Thread pool` - удобен в использовании и более эффективен с точки зрения различных процессов, которые происходят под капотом (по сравнению с множеством одиночных потоков).

#### Что такое `Thread pool`
> это множество потоков.

И этому множеству потоков программа поставляет разные задания:
`task 1`, `task 2`, ... `task n`

![[Pasted image 20241012123845.png]]

Под капотом `Thread pool` сам определяет какой поток будет выполнять какое задание.

Когда поток из `Thread pool` заканчивает выполнять свое текущее задание. Он дает понять `Thread pool-у`, что он свободен и готов брать новое задание.

> В java с `Thread pool-ами`удобнее всего работать посредством `ExecutorService`.

#### ExecutorService
Напрямую `Thread pool` практически никогда не создают.
```java
ExecutorService threadPoolExecutor = new ThreadPoolExecutor();
```


`Thread pool` удобнее всего создавать, используя factory методы класса `Executors`.
<font style="color:red">Предпочтительный</font> способ создания `Thread pool`:
```java
Executors.newFixedThreadPool(int count) - создаст pool с "count" потоками;
Executors.newSingleThreadPool(int count) - создаст pool с одним потоком;
```

##### Методы
- `execute()` - передает задание (`task`) в `Thread pool`, где оно выполняется одним из потоков.
- `shutdown()` - после выполнения этого метода `ExecutorService` понимает, что новых заданий больше не будет и, выполнив поступившие до этого задания, завершает работу.
- `awaitTermination()` (<font style="color:red">работает как</font> `join()`) - принуждает поток, в котором он вызвался, подождать до тех пор, пока не выполнится одно из двух событий:
  - `ExecutorService` завершит свою работу;
  - **истечет время**, переданное в параметр метода.

##### Наглядный пример использования `ExecutorService`
> Создаем `Thread pool` из `5` потоков.
> И передаем в него `10` заданий.

```java
public class ThreadPoolEx {  
    public static void main(String[] args) {  
        ExecutorService executorService = Executors.newFixedThreadPool(5);  
  
        for (int i = 0; i < 10; i++) {  
            executorService.execute(new MyRunnable100());  
        }  
  
        System.out.println("Main ends");  
    }  
}  
  
class MyRunnable100 implements Runnable {  
    public void run() {  
        System.out.println(Thread.currentThread().getName());  
        SleepUtil.sleep(500);  
    }  
}  
  
class SleepUtil {  
    public static void sleep(int millis) {  
        try {  
            Thread.sleep(millis);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    }  
}
```

- в потоке `main`, в цикле передаем `10` заданий в `Thread pool`, после этого поток `main` должен сразу же завершиться.

`Outout`:
```md
pool-1-thread-2
pool-1-thread-1
Main ends
pool-1-thread-5
pool-1-thread-3
pool-1-thread-4
pool-1-thread-3
pool-1-thread-2
pool-1-thread-1
pool-1-thread-5
pool-1-thread-4
```
- `Thread pool` начал работу быстрее, чем завершился поток `main` и успел дать задания двум своим потокам.
- Такое равномерное распределение работы м/ду потоками получилось за счет долгого выполнения каждого задания (`500` миллисекунд). Если убрать временную задержку `->` картина может изменится и какой-то один поток может выполнить почти все задания (т.к. задания будут выполняться практически моментально).

Разберем по шагам `output`:
1. `Thread pool` получил `1-е` задание и начал выполнять его в потоке `pool-1-thread-2`;
2. Поступило второе задание `->` идет проверка `есть ли свободный поток ?` ->  задание передается на выполнение потоку `pool-1-thread-1`;
3. Поток `main` завершает свою работу `->` значит все задания переданы в `Thread pool`;
4. `3-e` задание -> `pool-1-thread-5`;
5. `4-e` задание -> `pool-1-thread-3`;
6. `5-e` задание -> `pool-1-thread-4`;
---
В данной точке все потоки `Thread pool` заняты и параллельно выполняют `5` заданий.
Каждое задание выполняется `500` миллисекунд.
Но к этому моменту явно не прошло еще `0.5` секунды.
Остальные задания ждут пока освободится какой-л поток и возьмет их в работу.

---
7. Первым освобождается поток `pool-1-thread-3`;
8. Потом поток `pool-1-thread-2` и т.д.

<font style="color:red">ВАЖНО</font>:
В этом примере программа не завершает свою работу сама после выполнения этих `10` заданий.
Все потому, что `ExecutorService` ждет новые задания и по этому программа не завершается.
Если мы не намерены больше давать заданий `ExecutorService-у`, то обязательно нужно завершать его работу методом **`shutdown()`**.

##### Более полный пример использования `ExecutorService`
> Каждое задание выполняется `5` секунд.
> Поток `main` либо ждет завершения работы `ExecutorService`, либо **ТАКЖЕ** `5` секунд и продолжает свою работу.

```java
public class ThreadPoolEx {  
    public static void main(String[] args) throws InterruptedException {  
        ExecutorService executorService = Executors.newFixedThreadPool(5);  
  
        for (int i = 0; i < 10; i++) {  
            executorService.execute(new MyRunnable100());  
        }  

		// Завершаем работу Thread pool
        executorService.shutdown();
        // Поток main либо ждет завершения работы Thread pool, либо 5 секунд
        // и продолжает свою работу
        executorService.awaitTermination(5, TimeUnit.SECONDS);  
  
        System.out.println("Main ends");  
    }  
}  
  
class MyRunnable100 implements Runnable {  
    public void run() {  
        System.out.println(Thread.currentThread().getName());  
        SleepUtil.sleep(5000);  
    }  
}  
  
class SleepUtil {  
    public static void sleep(int millis) {  
        try {  
            Thread.sleep(millis);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    }  
}
```

`Output`:
```md
pool-1-thread-1
pool-1-thread-2
pool-1-thread-3
pool-1-thread-4
pool-1-thread-5
// Прошло ничтожно мало времени

// Прошло 5 секунд
pool-1-thread-2
Main ends
pool-1-thread-4
pool-1-thread-3
pool-1-thread-1
pool-1-thread-5
```
- первые `5` заданий взялись в работу моментально;
- после истечения `5` секунд поток  `pool-1-thread-2` оказался быстрее потока `main` и взял новое задание в работу (`пример не детерменированного поведения потоков`);
- поток `main` продолжил свою работу по истечению `5` секунд, не дожидаясь завершения работы `Thread pool`;

### 15. Thread pool и ExecutorService (`part 1`)


###