---
title: "Entity와 Value Object"
layout: post
date: 2019-05-27 01:20
headerImage: false
tag:
- ddd
- kotlin
- entity
- valueObject
category: ddd
author: Byunghun Lee
star: false
description: Entity and Value Object
---

### Entity

Entity의 가장 큰 특징은 각 Entity 객체마다 고유한 식별자 (identifier)를 갖는다는 것이다. Entity 객체의 멤버(속성)가 변경되더라도 식별자는 바뀌지 않는다. Entity 객체를 생성하고 Entity 객체의 속성을 바꾸고 Entity 객체를 삭제할 때까지 식별자는 유지돼야 한다.

따라서 두 Entity 객체의 식별자가 같다면 두 Entity 객체는 같다고 판단할 수 있다.



```kotlin
import java.util.*

data class Money (val value: Int)

class Wallet (private var _balance: Money) {
    val address: UUID = UUID.randomUUID()

    fun buySomething (price: Money) {
        _balance = Money(_balance.value - price.value)
    }

    val balance: Money get() = _balance
}
```

Wallet은 객체마다 고유한 식별자인 address를 가지고 있으므로 Entity라고 할 수 있다. 또한 id는 상수로 선언되었기 때문에 Wallet 객체가 소멸될 때까지 id는 같은 값을 유지한다.

쉽게 생각하면 지갑에서 얼마를 쓰던 지갑 자체가 바뀌는 것이 아닌 지갑의 잔액만이 바뀌는 것 처럼 Entity 객체의 멤버들의 상태가 어떻게 바뀌든 간에 Entity 객체의 식별자가 바뀌는 일은 없어야 한다.



### Value Object (VO)

VO는 Entity와 다르게 식별자를 갖지 않는다. 따라서 VO는 식별자로 객체의 같고 다름을 판별하지 않는다. VO는 객체에 존재하는 멤버들이 '모두 같은 값'을 갖고 있다면 같은 객체로 판단한다.

```kotlin
class Money (val value: Int, val foo: String)

fun main() {
    val money1 = Money(3000, "hello")
    val money2 = Money(3000, "hello")
    println(money1.hashCode()) // 1872034366
    println(money2.hashCode()) // 1581781576
}
```

Money라는 VO를 정의하고 money1과 money2 모두 같은 값으로 객체를 생성했다. 

위 설명대로라면 객체의 멤버들이 value = 3000,  foo = "hello"로 같으니 money1과 money2는 같은 객체여야 한다. 하지만 hashCode()를 출력해보면 서로 다른 값이 나온다. 이는 money1과 money2가 서로 다른 객체라는 뜻이다.

사실 자바에서 VO를 정의하기 위해서는 Object 객체의 equals()와 hashCode() 메소드를 오버라이딩하여 모든 멤버 값이 같은 경우, equals는 true를, hashCode는 같은 hash code를 반환하도록 재구현해야한다.

하지만 코틀린에서는 VO를 data class로 정의하면 위와 같은 과정이 필요가 없다.

```kotlin
data class Money2 (val value: Int, val foo: String)

fun main() {
    val money1 = Money2(3000, "hello")
    val money2 = Money2(3000, "hello")
    println(money1.hashCode()) // 99255322
    println(money2.hashCode()) // 99255322
}
```

결론은 코틀린을 이용해서 VO를 정의하는 것은 매우 간편하다.



### Value Object의 불변성

Entity와 다르게 일반적으로 VO의 멤버는 ***immutable(불변)*** 하도록 구현한다. 이유는 VO의 멤버가 어디서든 참조하여 변경이 가능하다면 해당 VO는 참조 무결성 및 스레드 안전성 등을 보장할 수 없다.

그렇다면 VO의 멤버를 mutable 하도록 구현하면 어떤 문제점이 생길까?

```kotlin
data class Money (var value: Int)

class Wallet (private var _balance: Money) {
    val address: UUID = UUID.randomUUID()

    fun buySomething (price: Money) {
        _balance.value -= price.value
    }

    val balance: Money get() = _balance
}

fun main() {
    val balance = Money(3000)
    val wallet = Wallet(balance)
}
```

현재 Money의 멤버인 value는 var로 선언되었고 public하다. 따라서 언제 어디서든 값의 변경이 가능하고, Money를 VO로 사용하는 wallet은 데이터 무결성을 보장받을 수 없다.

```kotlin
. . .

fun main() {
    val balance = Money(3000)
    val wallet = Wallet(balance)
    println(wallet.balance.value) // 3000
    balance.value = 3
    println(wallet.balance.value) // 3
}
```

현재 외부에서 balance의 값이 3으로 변경되었다. 그 결과 wallet과 상관 없이 wallet의 balance 값 또한 3으로 변경되었다.

이를 방지하기 위해서 Wallet을 아래와 같이 정의할 수 있다.

```kotlin
class Wallet () {
    val address: UUID = UUID.randomUUID()
    private lateinit var _balance: Money

    constructor (balance: Money): this() {
        // Wallet을 생성함과 동시에 새로운 Money VO를 재생성하여 balance를 초기화한다.
        this._balance = Money(balance.value)
    }

    fun buySomething (price: Money) {
        _balance.value -= price.value
    }

    val balance: Money get() = _balance
}

fun main() {
    val balance = Money(3000)
    val wallet = Wallet(balance)
    println(wallet.balance.value) // 3000
    balance.value = 3
    println(wallet.balance.value) // 3000
}
```

덕분에 외부에서 독립적으로 wallet의 balance를 조정할 수는 없게 되었다. 

하지만 

- ***코드 가독성이 떨어졌고***
- 생성자에서 Money 타입을 받아놓고 Money의 getter로 value를 얻어서 새로운 Money를 생성하여 balance 멤버를 초기화하는... ***코드스멜이 마구 풍기는 코드가 만들어졌다.***

이는 Money VO의 멤버를 val(상수)로 정의하여 불변성만 보장하면 깔끔하게 해결되는 문제이다.

```kotlin
data class Money (val value: Int)

class Wallet (private var _balance: Money) {
    val address: UUID = UUID.randomUUID()

    fun buySomething (price: Money) {
        // _balance.value -= price.value
        _balance = Money(_balance.value - price.value)
    }

    val balance: Money get() = _balance
}

fun main() {
    val basicMoney = Money(3000)
    val wallet = Wallet(basicMoney)
    println(wallet.balance.value) // 3000
    // basicMoney.value = 3 -> 상수(val) 데이터 변경 불가능
    wallet.buySomething(Money(500))
    println(wallet.balance.value) // 2500
}
```

주목할 부분은 

- ***Money VO의 value가 val로 선언되었다는 점***과 
- ***buySomething 메소드***이다.  

현재 value는 val(상수)로 선언되어 초기화 이후 변경이 불가능하다. 따라서 buySomething 메소드에서도 _balance 값을 변경하지 않고 새로운 Money VO를 생성하여 대입하고있다.

Money VO가 불변성을 보장하기 때문에, VO 값의 변경이 필요할 때마다 '값의 변경'이 아닌 '새 값의 대입'이 일어나는 것이다. 다시말해, 지갑 객체는 무엇인가를 구입할 때 잔액에서 구매액이 차감되는 것이 아닌, 낮은 금액의 새 (Money) 객체를 받는 것이다.



결론적으로 VO의 데이터를 불변타입으로 정의하면

- ***데이터 무결성을 보장***받을 수 있고, 
- 글에서 설명은 안했지만 ***다른 스레드에 의해 데이터가 예측불가한 값으로 변경되는 일 등을 방지***하여 
- ***보다 안전하고 안정된 코드 작성이 가능***해진다.