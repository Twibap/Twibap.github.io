---
layout: post
title: "Kotlin에는 Stream Api가 없다."
author: Twibap
date: '2022-10-15'
category: kotlin, java, stream, collections
---

# Kotlin과 stream

Kotlin에는 stream package가 없다. Kotlin에서 사용하는 stream 문법은 `kotlin.collections` 에서 구현 되어있다.
그래서인지 작동 방식 또한 완전히 다르다. Java로 개발하다가 Kotlin으로 넘어가기에 앞서 함수형 프로그래밍에 익숙해지기 위해 stream을 적극적으로 활용하고 있었다.
그리고 Kotlin을 사용하기 시작하면서 관성적으로 stream 문법을 사용했는데, 그런데 웬걸 같은 로직을 돌렸을 때 Java 보다 약 2.5배 느린 결과가 나왔다.
Kotlin과 Java의 stream 문법이 어떻게 다른지 알아보자.

## Stream 문법. 어떻게 다른가?

### Java에서 stream

1부터 10까지 자연수가 담긴 배열 중 짝수만 출력하는 로직을 준비했다.

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        int[] numbers = {
                1, 2, 3, 4, 5, 6, 7, 8, 9, 10
        };
        Arrays.stream(numbers)
                .filter(it -> {
                    System.out.println(it+" is checking to it is Even or odd");
                    return it % 2 == 0;
                })
                .forEach(it -> System.out.println(it+" is Even"));
    }
}
```

위 Java 코드를 실행하면 아래와 같이 출력된다.

```shell
> Task :Main.main()
1 is checking to it is Even or odd
2 is checking to it is Even or odd
2 is Even
3 is checking to it is Even or odd
4 is checking to it is Even or odd
4 is Even
5 is checking to it is Even or odd
6 is checking to it is Even or odd
6 is Even
7 is checking to it is Even or odd
8 is checking to it is Even or odd
8 is Even
9 is checking to it is Even or odd
10 is checking to it is Even or odd
10 is Even
```

이처럼 Java의 Stream은 `수직적`으로 동작한다.

### Kotlin에서 stream

Java에서 사용한 완전히 같은 로직이다. 훨씬 코드가 짧고 군더더기가 없어 보기 좋다. 하지만...

```kotlin
fun main(args: Array<String>) {
    (1..10)
        .filter {
            println("$it is checking to it is Even or odd")
            it % 2 == 0
        }
        .forEach {
            println("$it is Even")
        }
}
```

위 Kotlin 코드를 실행하면 아래와 같은 결과가 출력된다.

```shell
1 is checking to it is Even or odd
2 is checking to it is Even or odd
3 is checking to it is Even or odd
4 is checking to it is Even or odd
5 is checking to it is Even or odd
6 is checking to it is Even or odd
7 is checking to it is Even or odd
8 is checking to it is Even or odd
9 is checking to it is Even or odd
10 is checking to it is Even or odd
2 is Even
4 is Even
6 is Even
8 is Even
10 is Even
```

Kotlin에서 Stream은 `수평적`으로 동작한다. 즉, 매 함수 호출마다 새 Collection을 생성하게된다.
반복문이 호출된 횟수를 놓고 보면 Kotlin의 경우가 반복문 호출 횟수가 더 많다는 것을 알 수 있다.

## Kotlin이 Java보다 느리다?

위 예제는 로직이 같을 때 Kotlin이 Java보다 느리다는 결과를 보여주고 있다. Kotlin은 Java를 개량해서 나온 물건이다.
`stream` 문법을 사용하지 않고 순수하게 `for`문의 성능은 어떨까?

### for문 성능 비교

순수한 for문의 성능을 비교하기 위해 주어진 수보다 작은 소수의 갯수를 구하는 로직을 구성해 보았다.(사실 이거 하다가 발견한거다.)

```java
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        int number = Integer.parseInt(args[0]);

        long start = System.currentTimeMillis();

        System.out.println("Hello Prime!");
        System.out.println("Find Primes under "+ number);

        ArrayList<Integer> primes = new ArrayList<>();
        for (int i = 2; i <= number; i++) {
            if (isPrime(i))
                primes.add(i);
        }

        System.out.println("The Prime number Count is " + primes.size());

        long end = System.currentTimeMillis();
        System.out.println((end - start)+"ms");
    }

    static boolean isPrime(int number) {
        if (number < 2)
            return false;
        if (number == 2)
            return true;
        if (number % 2 == 0)
            return false;

        int rootOfNumber = (int) Math.sqrt(number);
        for (int i = 3; i <= rootOfNumber; i += 2) {
            if (number % i == 0)
                return number == i;
        }
        return true;
    }
}
```

```kotlin
import kotlin.time.ExperimentalTime
import kotlin.time.TimeSource

@OptIn(ExperimentalTime::class)
fun main(args: Array<String>) {
    val number = args[0].toInt()

    val mark = TimeSource.Monotonic.markNow()   // for Time check

    println()
    println("Find Primes under $number")

    val primeNumbers = ArrayList<Int>()
    for (i in (2..number)) {
        if (isPrime(i))
            primeNumbers.add(i)
    }

    println("The Prime number Count is ${primeNumbers.size}")
    println()
    println(mark.elapsedNow())  // for Time check
}

fun isPrime(number: Int): Boolean {
    if (number < 2)
        return false
    if (number == 2)
        return true
    if (number % 2 == 0)
        return false

    val rootOfNumber = sqrt(number.toDouble()).toInt()
    for (i in 3..rootOfNumber step (2)) {
        if (number % i == 0)
            return false
    }

    return true
}
```

두 언어의 실행 결과는 아래와 같다.

```shell
> Task :Main.main()
Hello Prime!
Find Primes under 10000000
The Prime number Count is 664579
1059ms
```

```shell
Find Primes under 10000000
The Prime number Count is 664579

919.656750ms
```

같은 JVM에서 실행한 결과인데도 근소하게 Kotlin이 더 나은 성능을 보여주었다.

## 결론

Kotlin에서는 Java에 비해 코드가 훨씬 간결하고 가독성에 있어서 훨씬 이점이 있었다.
반면, 내부 동작방식 차이로 인해 성능면에서 더 뒤쳐지는 결과를 낳았다.
하지만 이런 성능 차이는 stream 문법의 구현방식 차이로 인해 발생한 현상이다.
stream 문법을 사용하지 않고 단순 for문을 사용한 경우에는 Kotlin이 Java보다 더 나은 성능을 보여주었다.

## 하지만!

Java의 stream 동작 특성을 활용하면 얘기가 또 달라진다.

```java
    int[] numbers = new int[number];
    for (int i = 1; i <= number; i++) {
        numbers[i -1] = i;
    }

    long countOfPrimes = Arrays.stream(numbers)
            .parallel()
            .filter(Main::isPrime)
            .count();
```

Java stream의 `수직적` 특성을 활용하면 병렬 처리가 가능해진다. 이 경우에는 속도가 약 2배 빠른 결과를 보여주었다.

```shell
> Task :Main.main()
Hello Prime!
Find Primes under 10000000
The Prime number Count is 664579
537ms
```
