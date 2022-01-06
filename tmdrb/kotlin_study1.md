# 코틀린 스터디

## SAM(Single Abstract Method)

-> 인터페이스에 추상 메소드가 단 하나만 존재하는 것을 함수형 인터페이스 또는 SAM 이라고 부른다.

또한 SAM conversions 을 사용하면 람다식을 사용해서 코드를 더 간편하게 사용 할 수 있다.

## 람다식이란 ??

-> 익명 메소드를 전달해서 익명 클래스를 인스턴스화 시키는 것이다.

람다식을 설명하기 전에  **익명클래스**를 먼저 간략하게 설명

  ### 익명클래스란 ? 

  이름이 없는 클래스를 의미한다. 

  딱 한번만 쓰고 말 인터페이스나 자식클래스를 굳이 구현 할 필요가 있을까? 

  그래서 일회성으로 한번만 쓰기 위해서 나온것이 익명클래스이다.
  

람다식 사용 조건 

1. 람다식이 구현할 인터페이스는 단 1개의 추상 메소드를 가지고 있어야 한다.

2. 익명 메소드를 전달하면, 단 1개의 추상 메소드를 구현한 익명 인스턴스를 생성해낸다.

### 람다식을 사용하지 않았을 경우
```
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}


val isEven = object : IntPredicate {
   override fun accept(i: Int): Boolean {
       return i % 2 == 0
   }
}

```
### 람다식을 사용했을 경우

`
val isEven = IntPredicate{ i%2 == 0}
`

## 함수 인터페이스 vs 타입 별명

타입 별명은 단순히 하나의 멤버에 이미 존재하는 타입을 연결시켜준다. 
함수 인터페이스는 여러개의 추상적이지 않은 멤버와 한개의 추상적인 멤버를 가진다.
```
typealias Predicate<T> = (T) -> Boolean

fun foo(p: Predicate<Int>) = p(42)

fun main() {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f)) // prints "true"

    val p: Predicate<Int> = { it > 0 }
    println(listOf(1, -2).filter(p)) // prints "[1]"
}
```
공부가 더 필요
--------------


## Visibility modifiers

getter는 항상 같은 접근지정자를 가지며 나머지들은 다른 접근지정자를 가질수 있다.

private,public,iternal,protected 의 지정자가 있으며 기본값은 public이다.

functions, properties, classes, objects 들은 package 안에 top-level에 직접적으로 선언되어 질 수 있다.

이때 선언을 아무것도 안했으면 public이며 어디서든 다 접근가능하다.

앞에 private 선언 했다면 이 파일에서만 접근가능

앞에 iternal 선언 했다면 같은 모듈에서는 어디든지 사용가능하다.

top-level 에서는 protected 접근 지정자 사용 할 수 없다.

classes 에서 접근 지정자 사용

public member은 누구나 접근 가능

private member은 이 class 에서만 접근 가능

protected member은 private과 같지만 subclass에서도 접근 가능

iternal member는 class 가 선언된 같은 모듈안에서 접근 가능

```
open class Outer {
    private val a = 1
    protected open val b = 2
    internal open val c = 3
    val d = 4  // public by default

    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a is not visible
    // b, c and d are visible
    // Nested and e are visible

    override val b = 5   // 'b' is protected
    override val c = 7   // 'c' is internal
}

class Unrelated(o: Outer) {
    // o.a, o.b are not visible
    // o.c and o.d are visible (same module)
    // Outer.Nested is not visible, and Nested::e is not visible either
}
```

## Extension

### Extension function?

기존 클래스에 상속이나 디자인 패턴을 사용하지 않고 기능을 추가 하는것

```
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

reciver 타입을 앞에 쓰고 블록 안에 this 는 reciver 타입과 반응한다.

```
open class Shape
class Rectangle: Shape()

fun Shape.getName() = "Shape"
fun Rectangle.getName() = "Rectangle"

fun printClassName(s: Shape) {
    println(s.getName())
}

printClassName(Rectangle())

```

"shape" 이 프린트 됨

receiver type에 의해서 가상화가 되지 않는다는 것은 receiver로 Shape를 사용하여 Rectangle를 만들어내는 작업이 되지 않는다. (확장함수가 정적으로 전달 된다.)

properties 에서도 확장이 가능하다

val <T> List<T>.lastIndex: Int
  get() = size -1
  
하지만 초기화는 안된다. ex) val A.number = 1 -> error
  
### Companion Object?

java의 static 과 비슷

  
  

