## Extensions 

Kotlin에서는 상속과 decorator 패턴을 사용하지 않고도 새로운 기능을 확장 할 수 있다.

예를 들어 수정 할 수 없는 타사의 라이브러리의 클래스에 대한 새 함수를 작성 할 수 있다.

또한 원래의 클래스의 메소드 처럼 일반적으로 호출 할 수 있다.

```

//Kotlin code

fun MutableList<Int>.swap(index1: Int, index2: Int) {
    
    val tmp = this[index1] // 'this' corresponds to the list
    
    this[index1] = this[index2]
    
    this[index2] = tmp
}

val list = mutableListOf(1, 2, 3)

list.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'list'

---------------------------------------------------------------

//Java code


```

swap 함수를 확장하기 위한 방법으로 함수 이름앞에 리시버타입을 작성해준다.

위와 같이 swap 함수를 list 타입으로 확장 했다. 여기에서 this keyword는 리시버타입인 list와 같다.

list를 선언한뒤 바로 swap이라는 함수를 사용 할 수 있다.

```
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    
    val tmp = this[index1] // 'this' corresponds to the list
    
    this[index1] = this[index2]
    
    this[index2] = tmp
}
```
이런식으로 제너릭으로도 가능하다.

확장을 한다고 해서 확장을 하는 클래스가 수정되는 것이 아닌 "." 으로 클래스에서 새로운 함수만 호출 되게 한다.

### 확장은 리시버 타입을 유추해서 적용되는 것이 아닌 함수를 호출한 타입 그대로 호출된다. 

```
open class Shape

class Rectangle: Shape()

fun Shape.getName() = "Shape"

fun Rectangle.getName() = "Rectangle"

fun printClassName(s: Shape) {
    
    println(s.getName())
}

printClassName(Rectangle())
-----------------------------
결과값: Shape
```
위와 같이 s 는 Shape 타입이다. 하지만 실제로 함수를 호출 할 때 Shape을 상속한 Rectangle 타입으로 값을 넣었지만 Rectangle 타입으로 유추하지 않고 Shape 타입으로 인식한다.

```
class Example {
    
    fun printFunctionType() { println("Class method") }
}

fun Example.printFunctionType() { println("Extension function") }

Example().printFunctionType()
-------------------------------
결과값: Class method

-------------------------------
fun Example.printFunctionType(i: Int) { println("Extension function") 

Example().printFunctionType(1)
-------------------------------
결과값: Extension method
```
원래 class member 함수와 name과 인자가 같은 extension한 함수를 비교해 보면 class member 함수가 적용된다.

하지만 인자가 다른 함수면 인자에 맞게 결정된다.

extension 은 override 개념과는 다르다.

val House.number = 1 와 같이 단순히 값을 initalize 하는것은 적용되지 않는다.

또한 package를 선언하는 외부에서 정의해야 한다.

## DataClass

단순히 data 만을 관리하는 class를 만들기 위해 사용한다.

data class User(val name:String , val age:Int) 이런식으로 data class를 만들게 되면

equals()/hashCode() 

toString() of the form "User(name=John, age=42)"

componentN() 

copy() function

이런 함수들이 컴파일시 자동으로 생성된다. 자동으로 함수를 생성하지 않으려면 생성자를 만들지 말고 멤버로 선언하면 된다.
```

data class Person(val name: String) {
    
    var age: Int = 0

}
```
또한 data class를 작성시 유의사항이 있다.

1. 파라미터에 변수가 하나라도 있어야 한다.
2. val,var로 변수를 생성해야한다.
3. abstract,inner,sealed,open 은 될 수 없다.

## Enum Class

enum class 는 자바의 final과 같이 상수이며 지정해둔 class만 사용하게 할 수 있어 안전하다.

또한 인스턴스화 할 수 없으며 enum class 에서 인스턴스를 만들기 때문에 싱글톤 객체이다.

추상메소드를 사용 할 수 있고 생성자 초기화도 할 수 있다.

```
enum class Material(val type:String){
    PLASTIC("플라스틱"){
        override fun weightTowon(weight: Int): Int {
          return when(weight){
                in 0..10 -> 100
                in 10..29 -> 150
                in 30..100 -> 200
                else -> throw Exception()
            }
 
        }
    },
    GLASS("유리"){
    override fun weightTowon(weight: Int): Int {
          return when(weight){
                in 0..10 -> 150
                in 10..29 -> 250
                in 30..100 -> 300
                else -> throw Exception()
            }
 
        }
    },
    IRON("철"){
    override fun weightTowon(weight: Int): Int {
          return when(weight){
                in 0..10 -> 200
                in 10..29 -> 250
                in 30..30 -> 300
                else -> throw Exception()
            }
 
        }
    };
    
    abstract fun weightTowon(weight: Int): Int

}
```
### 주의사항 -> 생성자 초기화 시 프로퍼티명을 name으로 사용하면 안된다. name 이라는 프로퍼티는 enum class 안에 있는 클래스들의 이름을 출력해주도록 이미 존재하기 때문이다.


## Sealed Class

추상클래스로 상속받는 자식클래스를 제한하기 위한 클래스이다. Sealed class는 상속받는 자식클래스들을 알 수 있게 한다.

좀더 이해를 돕기위한 예:
```
abstract class CarAction

class Go: CarAction()

class Stop: CarAction()

class back: CarAction()

fun main() {
    
    fun getCarAction(ca: CarAction): String{
        
        return when(ca){
            
            is Go -> "GO"
            
            is Stop -> "Stop"
            
            is Back -> "Back"
        }
    }
   
}
// 아래와 같이 추상클래스를 만들고 상속받는 자식클래스들을 만들었다.
// 하지만 이상태로 하면 error가 발생한다.
// 컴파일러는 추상클래스를 상속받는 자식클래스들을 알지 못해서
// when' expression must be exhaustive, add necessary 'else' branch 이라는 error를 발생한다.
// else를 추가하면 error는 사라지지만 Go,Stop,Back 클래스중 하나가 없이 작성되어도 정상작동한다.
// 따라서 실제 상용하될 코드라면 위험 할 수 있다.

```
이러한 문제점을 해결하기 위해서 Sealed class 가 등장했다. enum과 비슷한 역할

sealed class 를 상속받는 자식 클래스들이 멤버가 없는 경우라면 object를 사용한다. (object는 한번만 메모리에 올려서 사용한다.)

같은 패키지 내에서만 상속할 수 있고 sealed class는 본인 스스로 인스턴스화 시킬수 없다.

##Generic

제네릭 타입을 사용하면 컴파일시 잘못된 타입을 체크 할 수 있다. 또한 타입을 외부에서 주입시켜 주기 때문에 타입변환 필요 없다.

java 에서 제네릭 타입을 사용하면 


```
class A<T>{
}
public class Main{
    public static void main(String[] args){
        A<String> a = new A<>();
        
    }

}
```


이런식으로 사용하면 원하는 타입만 외부에서 주입해서 사용하면 되므로 재사용성이 높아진다.

```
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! 여기서 컴파일 에러가 발생
objs.add(1); // 만약 에러가 나지 않아서 1을 집어 넣게 된다면
String s = strs.get(0); // !!! ClassCastException 런타임 에러 발생

```

여기에서 String는 Object를 상속받지만 List<String>는 List<Object>를 상속받지 않는다.

이렇게 형식 인자들끼리는 sub type 관계를 만족하더라도 제네릭을 사용하는 클래스와 인터페이스에서는 sub type 관계가 유지되지 않는 것이 Invariance(불변)이다.
    
그래서 이런 문제를 해결하기 위해서 자바에서는 wildcard가 있다.
    
```
interface Collection<E> ... {
  void addAll(Collection<? extends E> items);
}    
    
```
<? extends E>, <? super E> 이런식으로 작성해서 해결 가능하다.

위에서와 같이 items는 E일수도 E의 Subtype 일수도 있는 item들이 들어있다. 그래서 item을 읽으면 모두 E라는 형식에 담을 수 있다.

그러나 items에 값을 인서트 하지 못한다. 왜냐하면 ?가 어떤 타입인지 모르게 때문에 

읽기만 가능하고 쓰기는 불가능한 ? extends E 는 코틀린에서의 out과 비슷한 의미로 사용되고 이런 것들을 covariance(공변) 라 부릅니다.

반대로 읽기는 불가능하고 쓰기만 가능한 자바에선 ? super E 로 사용되고 코틀린에선 in 으로 사용되는 contravariance(반변)이 있습니다.

contravariance(반변)에서는 E와 같거나 E보다 상위의 타입만 ? 자리에 들어올 수 있습니다. 

items이 Collection<? super E> 라면, items에서 어떤 변수를 꺼내도 E에 담을 수 있을 지 보장할 수 없다. 큰타입을 작은타입으로 담을 수 없기 때문이다.

하지만 반대로 인서트는 가능하다. 
    
    
    
























    
기본적으로 코틀린의 모든 제네릭에서의 형식 인자는 Invariance이다.




    
    
    
    
   
















  

  
 
