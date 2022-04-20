---
title: Collections. Коллекции в Java.
tags: java exceptions
author: Jack Moloko
key: collections
---

## Collections. Коллекции в Java.

<!--more-->

![Image](/images/Java Collection Framework.drawio_1.svg)


### Collections

**`Iterable<E>`**
```java
class Main {
    void printAll(Iterable<?> iterable) {
        Iterator<?> iterator = iterable.iterator();
        while (iterator.hasNext()) {
            Object obj = iterator.next();
            System.out.println(obj);
        }
    }
    
    /* Сокращенная запись. */
    void printAll(Iterable<?> iterable) {
        for (Object obj : iterable) {
            System.out.println(obj);
        }
    }
}
```
### Свой Iterable

```java
import java.util.Iterator;
import java.util.NoSuchElementException;

class Main {
    static <T> Iterable<T> nCopies(T value, int count) {
        if (count < 0) {
            throw new IllegalArgumentException("Negative count: " + count);
        }
        return () -> new Iterator<T>() {
            int rest = count;

            @Override
            public boolean hasNext() {
                return rest > 0;
            }

            public T next() {
                if (rest == 0)
                    throw new NoSuchElementException();
                rest--;
                return value;
            }
        };
    }
}
```
### Collections<E> extends Iterable<E>
* `int size();`
* `boolean isEmpty();`
* `boolean contains(Object o);`
* `boolean containsAll(Collection<?> c);`
* `Object[] toArray(); `
* `<T> T[] toArray(T[] a);`
* `boolean add(E e);`
* `boolean remove(Object o);`
* `boolean addAll(Collection<? extends E> c);`
* `boolean removeAll(Collection<?> c);`
* `boolean retainAll(Collection<?> c);`
* `void clear();`

### List<E> extends Collection<E>
* `boolean addAll(int index, Collection<? extends E> c);`
* `void sort(Comparator<? super E> c);`
* `E get(int index);`
* `E set(int index, E element);`
* `void add(int index, E element);`
* `E remove(int index); / boolean remove(Object o);`
* `int indexOf(Object o);`
* `int lastIndexOf(Object o);`
* `ListIterator<E> listIterator();`
* `ListIterator<E> listIteraor(int index);`
* `List<E> subList(int fromIndex, int toIndex);`

### ListIterator<E> extends Iterator<E>
* `boolean hasPrevious();`
* `E previous();`
* `int nextIndex();`
* `int previousIndex();`
* `void set(E e);`
* `void add(E e);`

### Стандартные списки
* **ArrayList** - изменяемый список общего назначения
* **Arrays.asList** - изменяемая обертка над массивом
* **Collections.emptyList()** - неизменяемый пустой список
* **Collections.singletonList(x)** - неизменяемый список из одного элемента
* **Collections.nCopies(n, x)** - неизменяемый список из n одинаковых элементов
* **List.of(...)** - (java 9) неизменяемый список из указанных элементов(или массива), null не приемлет
* **List.copyOf(...)** - (java 10) неизменяемая копия указанного списка
* **Collections.unmodifiableList(list)** - неизменяемая обертка над списком
* **Collections.synchronizedList(list)** - синхронизированная обертка над списком
* **Collections.checkedList(list, type)** - проверяемая обертка

### Стандартные множества
* **HashSet** - изменяемое неупорядоченное множество общего назначения
* **LinkedHashSet** - изменяемое упорядоченное (ordered) множество общего назначения
* **TreeSet** - изменяемое сортированное (sorted) множество
* **EnumSet** - изменяемое множество элементов Enum
* **Collections.emptySet()** - неизменяемое пустое множество
* **Collections.singletone(x)** - неизменяемое множество из одного элемента
* **Set.of(...)** - (java 9) некпорядоченное неизменяемое множество заданных элементов(без null и повторов)
* **Set.copyOf(...)** - (java 10) неизменяемая копия
* **Collections.unmodifiableSet(set)** - неизменяемая обертка над множеством
* **Collections.synchronizedSet(set)** - синхронизированная обертка над множеством
* **Collections.checkedSet(set, type)** - проверяемая обертка

### HashSet
* Использует hashCode и расширяемую hash-таблицу 
  _(table[obj.hashCode() % table.length])_
* При коллизии формирует связный список
* Если коллизий много, то формирует RB-дерево (красно-черное дерево)

### TreeSet

_TreeSet< E > implements NavigableSet< E >_
* `first(), last()`
* `headSet(), tailSet(), subset()`
* `floor(E), ceiling(E)`
* `higher(E), lower(E)`
* `descendingSet()`
* `descendingIterator()`

### Queue

_Queue< E > extends Collection< E >_
* `offer/add`
* `poll/remove`
* `peek/element`

`offer/poll/peek` - Возвращают null/false, если не вышло

`add/remove/element` - Кидают исключения, если не вышло
 
### Deque

_Deque< E > extends Queue< E >_
* `offerFirst/addFirst`
* `offerLast/addLast`
* `pollFirst/removeFirst`
* `pollLast/removeLast`
* `peekFirst/getFirst`
* `peekLast/getLast`

#### Стандартные очереди
* **ArrayDeque** - изменяемая очередь общего назначения
* **PriorityQueue** - изменяемая очередь с приоритетом (heap-based)

### !!! Не надо использовать !!!
* **Enumeration** -> Iterator
* **Vector** -> ArrayList
* **Stack** -> ArrayDeque
* **Dictionary** -> Map
* **Hashtable** -> HashMap
* **LinkedList** -> ArrayList/ArrayDeque

