---
title: Elements of functional programming
tags: java functional lambda optional
author: Jack Moloko
key: functional
---

## Elements of functional programming

<!--more-->

### Функциональный интерфейс
* Интерфейс (не абстрактный класс), содержащий единственный абстрактный метод
* Может быть аннотирован как `@FunctionalInterface`

### Функциональное выражение в Java
* Отображается на функциональный интерфейс
* Не имеет состояния
* Конкретный тип устанавливается по контексту
* Либо лямбда-выражение `((a, b) -> a+b)` либо ссылка на метод `(Integer::sum)`
* Не гарантирует `identity`
* Не гарантирует `getClass() identity`

### Лямбда выражение
* Аргументы:
    * `(Type1 name1, Type2 name2)`
    * `(name1, name2)`
    * `Name`
* Стрелочка `->`
* Тело:
    * Выражение `System.out.println()`
    * Блок `{ System.out.println(); }`
* Void-compatible (void SAM-single abstract method):
    * Блок: каждый return не содержит выражения
    * Выражение: допустимое в утверждении (вызов метода, присваивание, печать и т.д.)
* Value-compatible (non-void SAM-single abstract method):
    * Блок: каждый return содержит выражение и нормальное завершение невозможно
    * Выражение: имеет значение не void
    
#### Примеры лямбда-выражений
```java
// Пример лямбды
public class Main {
    public static void main(String[] args) {
        Runnable r = () -> System.out.println("Hello");
        r.run();
        Runnable rn = (args.length > 0 ?
                () -> System.out.println("Hello " + args[0] + "!") :
                () -> System.out.println("Hello World!"));
    }
}
```
```java
// Пример лямбды
public class Main {
    static void run(Runnable r) {
        r.run();
    }

    public static void main(String[] args) {
        run(() -> System.out.println("Hello world!"));
    }
}
```
```java
// Пример лямбды
public class Main {
    public static void main(String[] args) {
        Object x = (Runnable) (() -> System.out.println("Hello!"));
    }
}
```
```java
class Main {
    public static void main(String[] args) {
        IntSupplier x = () -> 5;
        IntSupplier x2 = () -> System.out.println(5); // Error. Value-compatible
        
        Runnable y1 = () -> 5; // Error. Void-compatible
        Runnable y2 = ()-> System.out.println(5);
    }
}
```

### Захват (Capture) значений
* **Локальная переменная** - должна быть effectively final и инициализирована к месту использования лямбды, берется значение этой переменной.
* **Поле текущего класса** - захватывается this, значение из поля читается при выполнении.
* **Статическое поле** - ничего не захватывается, значение читается при выполнении.
* **Closure (Замыкание)** - лямбда-выражение с захваченными переменными.

#### Примеры захвата значений (замыканий)
```java
class Main {
    int field = 10;
    static int sField = 15;
    
    void test() {
        int var = 5;
        Rnnable r1 = () -> System.out.println(var);
        Rnnable r2 = () -> System.out.println(field);
        Rnnable r3 = () -> System.out.println(sField);
        field = 5;
        sField = 5;
        r1.run(); // 5
        r2.run(); // 5
        r3.run(); // 5
    }
}
```
### Ссылка на метод (method reference)
* Статический метод: 
    * `IntBinaryOperator sum = Integer::sum;`
* Не статический метод (первый параметр SAM-метода превращается в квалификатор): 
    * `Function<String, String> trimmer = String::trim; // res = trimmer.apply("  hello  ");`
* Не статический метод привязанный к экземпляру(instance-bound):
    * `Predicate<String> isFoo = "foo"::equals;`
    * `Consumer<Object> printer = System.out::println;`
* Конструктор:
    * `Supplier<List<String>> listFactory = ArrayList::new;`
* Конструирование массива:
    * `IntFunction<String[]> arrayFactory = String[]::new;`
    
_Пример_
```java
class Main {
    public static void main(String[] args) {
        Predicate<String> pred = "Java"::equals;
        System.out.println(pred.test("Java"));      // true
        System.out.println(pred.test("Kotlin"));    // false
        
        Consumer<Object> methodRefPrinter = System.out::println;
        Consumer<Object> lambdaPrinter = obj -> System.out.println(obj);
        System.setOut(null);
        methodRefPrinter.accept("hello");   // hello
        lambdaPrinter.accept("hello");      // Error. NullPointerException
    }
}
```
### Примитивы функционального программирования

_Примеры_

```java
import java.util.Objects;
import java.util.function.BiFunction;

class Main {
    // Привязка аргумента (bind)
    static <A, B, C> Function<B, C> bind(BiFunction<A, B, C> fn, A a) {
        Objects.requireNonNull(fn);
        return b -> fn.apply(a, b);
    }
    
    // Карирование (curry)
    static <A, B, C> Function<A, Function<B, C>> curry(Bifunction<A, B, C> fn) {
        return a -> b -> fn.apply(a, b);
    }

    public static void main(String[] args) {
        Function<Integer, Integer> inc = bind(Integer::sum, 1);
        System.out.println(inc.apply(10));  // 11

        System.out.println(curry(Integer::sum).apply(5).apply(6));  // 11
    }
}
```

### Optional< T >
* Либо значение (не null), либо его отсутствие
* Optional.of(obj) создает Optional с переданным значением
* Optional.ofNullable(obj) если значение не null - создается Optional со значением,  
  если значение null - создается пустой Optional 
* Optional.empty() создает пустой Optional

_Пример:_
```java
class Main {
    static Optional<Integer> toInteger(String opt ){
        try {
            return Optional.of(Integer.valueOf(opt));
        } catch (NumberFormatException ex) {
            return Optional.empty();
        }
    }
}
```
#### Методы Optional _(не все, часто используемые)_**
* `filter(Predicate T)`
    * `boolean isFooEqualsBar = Optional.of("foo").filter("bar"::equals).isPresent();`
* `<U> map(Function<T, U>)`
    * `String trimmed = Optional.of("  foo  ").map(String::trim).get();`
* `<U> flatMap(Function<T, Optional<U>>)`
    * `Integer num = Optional.of("1234").flatMap(x -> toInteger(x)).orElse(-1);`
* `orElseGet(Supplier<T>)`
    * `Double random = Optional.<Double>empty().orElseGet(Math::random);`
* `<X extends Throwable> orElseThrow(Supplier<T>) throws X`
    * `Double random = Optional.<Double>empty().orElseThrow(Exception::new);`
* `or(Supplier<? extends Optional<? extends T>>)`
    * `getOneOptional().or(() -> getAnotherOptional());`
    
_Пример:_
```java
interface User {
    String name();
    boolean isActive();
    void updateCarma(int delta);
}
interface UserRepository {
    Optional<User> findUser(String name);
}

class Main {
    void increaseUserCarma(UserRepository repository, String name) {
        repository.findUser(name)
                .filter(User::isActive)
                .orElseThrow(() -> new IllegalArgumentException("No such user: " + name))
                .updateCarma(1);
    }
}
```
