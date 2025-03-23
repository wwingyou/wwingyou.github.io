---
layout: post
title: 이터레이터 패턴
tags: DesignPattern
---

## 이터레이터 패턴

자료형의 구조와는 상관 없이 데이터를 순회할 수 있도록 해주는 '이터레이터'
인터페이스를 이용하는 패턴.

모든 자료구조를 일관적으로 사용할 수 있다는 점에 장점이 있다.

Java 에서는 `Interable<T>` 인터페이스와 `Iterator<T>` 인터페이스를 사용한다.

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

class IteratorTest {

    private static class MyList<T> implements Iterable<T> {

        List<T> store = new ArrayList<>();

        public void add(T item) {
            store.add(item);
        }

        @Override
        public Iterator<T> iterator() {
            return new Iterator<T>() {

                int index = 0;

                @Override
                public boolean hasNext() {
                    return store.size() > index;
                }

                @Override
                public T next() {
                    return store.get(index++);
                }

            };
        }

    }


    public static void main(String[] args) {
        MyList<String> myList = new MyList<>();
        myList.add("Hello");
        myList.add("World");
        myList.add("Boys!");

        for (String text : myList) {
            System.out.println(text);
        }
    }
}
```

코드에서 볼 수 있듯 `Iterable<T>` 인터페이스는 `Iterator<T>` 인터페이스의
구현체를 반환하는 `iterator()` 메소드를 가지고 있다. 자신이 가지고 있는 아이템을
순회할 수 있는 `Iterator<T>` 인스턴스를 여기서 반환하면 `for` 문 등에서 이를
활용해 리스트의 아이템을 순회할 수 있다.

### 장점

- 여러 타입의 자료구조를 일관적으로 순회할 수 있다.

### 단점

- `for`문 등을 활용해 직접 순회하는 것보다는 느리다.
