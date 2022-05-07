---
title: Exceptions. Создание и обработка исключений.
tags: java exceptions
author: Jack Moloko
key: exceptions
---

## Exceptions. Создание и обработка исключений.

<!--more-->

![Image](/images/Exception.drawio.svg)

```java
public class Test {
    static class MyException extends Exception {
        
    }

    static void test() throws MyException {
        throw new MyException();
    }

    public static void main(String[] args) throws MyException {
        test();
    }
}
```
### Конструирование исключений

_Throwable, Exception, RuntimeException_
* `Throwable()`
* `Throwable(message)`
* `Throwable(cause)`
* `Throwable(message, cause)`
* `protected Throwable(message, cause, suppression, stackTrace)`

### Что есть у исключений
* Сообщение (getMessage())
* Стек (getStackTrace())
* Причина (getCause())
* Подавленные исключения (addSuppressed()/getSuppressed())

### Логирование
* Фреймворки: Log4j, Logback, Slf4j, Java Logging API (java.util.logging)
* Уровни: ERROR/WARN/INFO/DEBUG
* Форматтеры: как логировать? (представление сообщений)
* Аппендеры: куда логировать?

### Перебрасывание исключений

_Пример:_

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Arrays;

static class MyException extends Exception {
    public MyException(String message) {
        super(message);
    }

    public MyException(String message, Throwable cause) {
        super(message, cause);
    }
}

public class Main {
    void readFile() throws MyException {
        try {
            byte[] bytes = Files.readAllBytes(Paths.get("/etc/passwd"));
            System.out.println(Arrays.toString(bytes));
        } catch (IOException e) {
            throw new MyException("Unable to read file", e);
        }
    }
}
```

