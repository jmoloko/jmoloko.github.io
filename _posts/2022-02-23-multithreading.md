---
title: Multithreading in Java
tags: java multithreading
author: Jack Moloko
key: multithreading
---

## Multithreading in Java

<!--more-->


### Доступ к разделяемому ресурсу
* Блокировка (mutex, mutual exclusion)
* Неблокирующий доступ:
  * Lock-free: гарантируется общий прогресс
  * Wait-free: гарантируется прогресс в потоке исполнения
  
### Блокирующий доступ
* Для блокировки используется монитор или лок
* Один и тот же ресурс должен блокироваться одним и тем же монитором
* Блокировать надо как чтение, так и запись

### Synchronized
* `synchronized(obj) {...}`: монитор — объект obj
* `synchronized void method()`: монитор — объект this
* `synchronized static void method()`: монитор — объект .class

_Пример:_
```java
public class Container {
    private final List<String> list = new ArrayList<>();
    synchronized void addEntry(String s) {
        list.add(s);
    }
    int size() {
        return list.size();
    }

    public static void main(String[] args) throws InterruptedException {
        Container container = new Container();
        Runnable foo = () -> {
            for(int i=0; i<100000; i++) {
                container.addEntry("foo");
            }
        };
        List<Threads> threads = new ArrayList<>();
        for(long count=10; count>0; count--) {
            Thread thread = new Thread(foo);
            thread.start();
            threads.add(thread);
        }
        for (Thread thread : threads) {
            thread.join(); // дождаться завершения thread
        }
        System.out.println(list.size());
    }
}
```

### Атомарность (atomicity)
> Операция атомарна, если невозможно наблюдать частичный результат ее выполнения.  
> Любой наблюдатель видит либо состояние системы до атомарной операции, либо после.  

* **В Java:**
  * Запись в поле типа **boolean**, **byte**, **short**, **char**, **int**, **float**, **Object** всегда атомарна
  * Запись в поле **long/double**: атомарна запись старших и младших 32 бит
  * Запись в поле типа **long/double**, объявленное **volatile**, атомарна


**!! arithmetic operations are not atomic !!**  
* Арифметические операции не атомарны!
  * `x++` - не атомарна
  * `x *= 2` - не атомарна

```java
/* НЕ работает!!! */
class Counter {
    volatile int x = 1;
}

public class Main {
    public static void main(String[] args) {
        Counter c = new Counter();
        Runnable r = () -> {
            for (int i = 0; i < 1000000; i++) c.x++;
        };

        List<Thread> threads = Stream.generate(() -> new Thread(r))
                .limit(10).peek(Thread::start)
                .collect(Collectors.toList());

        for(Thread thread: threads)
            thread.join();

        System.out.println(c.x);
    }
}
```

#### Когда завершилась операция?
* **A = 1**
  * ~~Когда приступили к выполнению следующей операции?~~
  * ~~Когда значение 1 оказалось в основной памяти в ячейке соответствующей переменной А?~~
  * ~~Когда результат изменился в точке чтения переменной А?~~

#### Видимость (visibility)
* Результат операции write A, выполненной в потоке 1, виден в операции read A, выполненной в потоке 2
* Видимость определена только для конкретных потоков 1 и 2, нет "глобальной видимости"

#### Порядок (ordering)
* **A happens before B** (A hb B), если все записи, выполненные до точки A (включительно),  
  видны в любой операции чтения после точки B (включительно)
* **A hb B, B hb C -> A hb C**

#### Простые правила happens before
* Для двух операций A и B в одном потоке **A hb B**, если A раньше B в тексте программы (program order)
* Завершение конструктора объекта X **hb** начало finalize X
* Вызов thread.start() **hb** первое действие в потоке thread
* Последнее действие в потоке thread **hb** thread.join()
* Инициализация объекта по умолчанию **hb** любое другое действие

**synchronized**
* Между синхронизациями по одному объекту установлен полный порядок (total order)
* Завершение синхронизации (monitor exit) **hb** начало последующей синхронизации  
  по тому же объекту (monitor enter)
  
### Volatile
* Запись и чтение в поле, объявленное **volatile**, называется volatile write, volatile read
* Речь идет непосредственно о записи, а не о записи членов/элементов массива
  * `volatile int[] x;`
  * `x = new int[10]; // volatile write`
  * `x[0] = 1; // volatile read, plain write`
* volatile write **hb** volatile read, который прочитал это значение

### Singleton (Double checked locking fixed)

```java
public class Container {
    private static volatile Container INSTANCE;
    int x = 1;

    private Container() {
        if (INSTANCE == null) {
            synchronized (Container.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Container();
                }
            }
        }
        return INSTANCE;
    }
}
```

**final**
* Если поток увидел ссылку на объект и она не утекала из конструктора,  
  то он гарантированно увидит все **final** - поля, записанные в конструкторе
* Неизменяемые объекты — друзья многопоточности!

### Singleton (initialization on demand holder idiom)
```java
public class Container {
    int x = 1;
        
    static Container getInstance() {
        return Holder.INSTANCE;
    }
    
    private static class Holder {
        static final Container INSTANCE = new Container();
    }
}
```

### Dead Lock
* Взаимное исключение (неразделяемые ресурсы)
* Минимум два ресурса (один держим, один просим)
* Ресурс освобождается только добровольно тем, кто его держит
* Направленный граф ожидания имеет цикл

_Пример dead lock:_
```java
class Main {
    static void transfer(Queue<String> in, Queue<String> out) {
      synchronized (in) {
        synchronized (out) {
          String res = in.poll();
          if (res != null) {
            out.add(res);
          }
        }
      }
    }

  public static void main(String[] args) {
    Queue<String> in = new ArrayDeque<>(Arrays.asList("foo", "bar", "baz"));
    Queue<String> out = new ArrayDeque<>(Arrays.asList("foo", "bar", "baz"));
    Thread t1 = new Thread(() -> {
      for (int i = 0; i < 100000; i++) {
        System.out.println("Thread1: " + i);
        transfer(in, out);
      }
    });
    Thread t2 = new Thread(() -> {
      for (int i = 0; i < 100000; i++) {
        System.out.println("Thread2: " + i);
        transfer(out, in);
      }
    });
    System.out.println("Started");
    t1.start(); t2.start();
    t1.join(); t2.join();
    System.out.println("Finished");
  }
}
```
![deadlock](/images/java_deadlock.png)

### Live Lock
* Потоки постоянно меняют состояние, но прогресса нет
  
![livelock](/images/java_livelock.png)


#### Ожидание условия

```java
public class Main {
    private boolean content = false;

    public synchronized void waitForContent(){
        while (!content) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("Content has been arrived");
    }

    public synchronized void deliverContent(){
        content = true;
        notifyAll();
    }
}
```

### Стандартная библиотека java.util.concurrent

* *Атомарные переменные*
* *Неблокирующие коллекции*
* *Блокирующие коллекции*
* *Примитивы синхронизации*
* *Пулы потоков*
* *CompletableFuture*

#### Атомарные переменные (j.u.c.atomic)

* **AtomicBoolean**/*Integer*/*Long*/**Reference**
  * `AtomicIntegerReference AtomicBooleanReference ...`
* **AtomicInteger**/*Long*/**ReferenceArray**
  * `AtomicIntegerReferenceArray AtomicLongReferenceArray`
* **AtomicInteger**/*Long*/**ReferenceFieldUpdater**
  * `AtomicIntegerReferenceFieldUpdater ...`
* *Long*/*Double*/**Accumulator**
  * `LongAccumulator DoubleAccumulator`
* *Long*/*Double*/**Adder**
  * `LongAdder DoubleAdder`


#### Compare and Set
* (expect, update) -> result
* Базовый примитив для всех lock-free алгоритмов

```java
public class Main {
    private final AtomicBoolean flag = new AtomicBoolean(false);

    void doOnce(Runnable action) {
        if (flag.compareAndSet(false, true)) {
            action.run();
        }
    }
}
```

**Compare and set через updater**

```java
public class Doer {
    private volatile int flag = 0;

    private static final AtomicIntegerFieldUpdater<Doer> FLAG_UPDATER =
            AtomicIntegerFieldUpdater.newUpdater(Doer.class, "flag");

    void doOnce(Runnable action){
        if (FLAG_UPDATER.compareAndSet(this, 0, 1)){
            action.run();
        }
    }
}
```

**Compare and Set в действии**  
**Реализация атомарного инкремента**

```java
import java.util.concurrent.atomic.AtomicInteger;

public class Main {
    private final AtomicInteger count = new AtomicInteger(0);

    // getAndIncrement есть в стандартной библиотеки
    public int getAndIncrement(AtomicInteger count) {
        int cur;
        do {
            cur = count.get();
        } while (!count.compareAndSet(cur, cur + 1));
        return cur;
    }

    // реализация атомарного метода
    public int getAndDouble(AtomicInteger count) {
        int cur;
        do {
            cur = count.get();
        } while (!count.compareAndSet(cur, cur * 2));
        return cur;
    }

    // getAndDouble с использованием библиотечного метода getAndUpdate
    public int getAndDoubleStandard(AtomicInteger count) {
        return count.getAndUpdate(val -> val * 2);
    }


}
```

**Атомарное обновление нескольких взаимосвязанных переменных**  
**Пример! Изменение координат точки на плоскости**

*Для изменения логически связанных переменных, создаем объект*  
*и меняем сам объект атомарно*

```java
import java.util.concurrent.atomic.AtomicReference;

public class AtomicPoint {
    private static class Point {
        final int x, y;

        private Point(int x, int y) {
            this.x = x;
            this.y = y;
        }

        Point rotateClockwise() {
            return new Point(y, -x);
        }
    }

    private final AtomicReference<Point> pt = new AtomicReference<>(new Point(0, 1));

    public void rotateClockwise() {
        pt.updateAndGet(Point::rotateClockwise);
    }
}
```

### Применение стандартной библиотеки j.u.concurrent  
Методы стандартной библиотеки
* *CountDownLatch* - дождаться обнуления
* *Semaphore* - взять ресурс
* *Exchanger* - обменяться значениями
* *CyclicBarrier* - периодическая работа с синхронизацией
* *Phaser* - работа разделенная на фазы
* *Executors* - делегирование создания потоков 
* *CompletableFuture*
* *ReentrantLock*
* *ReentrantReadWriteLock*

**Тестирование метода doOnce (единоразовое увеличение на 1) см. выше**

```java
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class Main {
    public static void main(String[] args) {
        // количество потоков в зависимости от кол-ва ядер процессора
        final int THREADS = Runtime.getRuntime().availableProcessors();
        Doer doer = new Doer();
        CountDownLatch latch = new CountDownLatch(1);
        AtomicInteger count = new AtomicInteger();

        Runnable r = () -> {
            try {
                latch.await();
            } catch (InterruptedException e) {
            }
            doer.doOnce(count::incrementAndGet);
        };
        List<Thread> threads = Stream.generate(() -> new Thread(r))
                .limit(THREADS).peek(Thread::start)
                .collect(Collectors.toList());
        latch.countDown();

        for (Thread thread: threads) {
            thread.join();
        }

        if (count.get() != 1){
            System.out.println("oops!");
        }
    }

}
```
_Примеры CompletableFuture:_

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicReference;
import java.util.stream.Stream;

class Main {
  public static void main(String[] args) {
    final int THREADS = Runtime.getRuntime().availableProcessors();
    AtomicReference<Doer> doer = new AtomicReference<>();
    AtomicInteger count = new AtomicInteger();
    Runnable r = () -> doer.get().doOnce(count::incrementAndGet);
    for (int i = 0; i < 10000; i++) {
      count.set(0);
      doer.set(new Doer());
      CompletableFuture<?> future = CompletableFuture.allOf(
              Stream.generate(() -> CompletableFuture.runAsync(r))
              .limit(THREADS).toArray(CompletableFuture[]::new));
      future.join();
      if (count.get() != 1){
        System.out.println("oops!");
      }
    }
  }
}
```

```java
import java.math.BigInteger;
import java.util.concurrent.CompletableFuture;

public class Main {
    public static BigInteger factorial(int i) {
        BigInteger res = BigInteger.ONE;
        while (i > 1) {
            res = res.multiply(BigInteger.valueOf(i));
            i--;
        }
        return res;
    }

    public static BigInteger combinations(int n, int k) {
        CompletableFuture<BigInteger> factN = CompletableFuture.supplyAsync(() -> factorial(n));
        CompletableFuture<BigInteger> factK = CompletableFuture.supplyAsync(() -> factorial(k));
        CompletableFuture<BigInteger> factNminusK = CompletableFuture.supplyAsync(() -> factorial(n - k));
        return factN.thenCombine(factk, BigInteger::divide)
                .thenCombine(factNminusK, BigInteger::divide).join();
    }
}

```
### ReentrantLock  
_Пример:_

```java
import java.util.concurrent.locks.ReentrantLock;

class Main {
  private final List<String> list = new ArrayList<>();
  private final ReentrantLock lock = new ReentrantLock(true); // честный, если true
  
  String get(int i) {
      lock.lock();
      try {
          return list.get(i);
      } finally {
          lock.unlock();
      }
  }
  
  void add(String str) {
      lock.lock();
      try {
          list.add(str);
      } finally {
          lock.unlock();
      }
  }
}
```

### ReentrantReadWriteLock  
_Пример:_

```java
import java.util.concurrent.locks.ReentrantReadWriteLock;

class Main {
  private final List<String> list = new ArrayList<>();
  private final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
  
  String get(int i) {
      ReentrantReadWriteLock.ReadLock readLock = lock.readLock();
      readLock.lock();
      try {
          return list.get(i);
      } finally {
          readLock.unlock();
      }
  }
  
  void add(String str) {
      ReentrantReadWriteLock.WriteLock writeLock = lock.writeLock();
      writeLock.lock();
      try {
          list.add(str);
      } finally {
          writeLock.unlock();
      }
  }
  
}
```

### Примитивы синхронизации, конкурентные коллекции, аннотации, рефлекшн

**Не блокирующие коллекции**
* *ConcurrentLinkedQueue (LinkedList)*
* *ConcurrentLinkedDequeue (LinkedList/ArrayDequeue)*
* *CopyOnWriteArrayList (ArrayList)*
* *CopyOnWriteArraySet*
* *ConcurrentHashMap (+ newKeySet()) (HashMap/HashSet)*
* *ConcurrentSkipListMap (TreeMap)*
* *ConcurrentSkipListSet (TreeSet)*

**Принципы не блокирующих коллекций**
* Простые операции атомарны
* Пакетные операции (addAll, removeAll) могут быть не атомарны
* Как правило, длина не хранится (isEmpty()!)
* Не кидает ConcurrentModificationException
* Обычно weakly-consistent (изменения после создания итератора могут быть видны или не видны)

**CopyOnWriteArrayList/Set**
* Содержимое хранится в массиве
* Модифицирующие операции синхронизированы
* Операции на чтение и iterator не синхронизированы
* Любое изменение копирует массив
* Итератор, forEach обходит снимок коллекции н начало итерации (итератор не может модифицировать)
* Сортировать можно Java 8+
* CopyOnWriteArraySet - делегат к CopyOnWriteArrayList
* Методы с множественными модификациями не желательны

**ConcurrentHashMap**
* До Java 8: совокупность сегментов, каждый из которых HashMap
* В Java 8 переписан, больше похож на обычный HashMap
* reduce*/search*/forEach* - создают параллельные задачи
* putIfAbsent, computeIfAbsent, merge и т.д. - атомарны, но могут вызывать функцию под синхронизацией

**ConcurrentSkipListMap**
* Конкурентная замена TreeMap
* Структура данных: skip-list, рандомизированна
* Есть ConcurrentSkipListSet
* computeIfAbsent может вызвать функцию и выкинуть результат
* merge может вызвать функцию несколько раз

![ConcurrentSkipListMap](/images/java_skiplistmap.png)

**Блокирующие коллекции**
* *SynchronousQueue (без памяти, одна вставка = одно удаление)*
* *ArrayBlockingQueue (фиксированный размер)*
* *LinkedBlockingQueue (размер может быть не фиксирован)*
* *PriorityBlockingQueue (PriorityQueue x ArrayBlockingQueue)*

