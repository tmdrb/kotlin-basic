# 함수형 프로그래밍이란?

함수를 사용해서 데이터를 처음부터 끝까지 동일한 결과를 내도록 하는 프로그래밍이다.

## 함수형 프로그래밍 특징 

1. 불변성
2. 참조 투명성 
3. 일급함수
4. 게으른 평가 

## 함수형 프로그래밍의 장점

함수형 프로그래밍을 사용하면 부수효과가 없다. 

또한 코드가 간결해진다.

같은 데이터를 넣으면 항상 같은 결과값이 나온다. 불변성

### 부수효과란??

동일한 함수를 실행 했을 때 지역변수나 전역변수로 인해 결과값이 달라지는 것이다.

ex) 

`
var num3: Int = 100
fun Plus(num1: Int, num2: Int): Int { 
  num2 = num3
  return num1 + num2
}

`

위의 함수를 실행시키게 되면 입력값 num2가 전역변수로 인해서 값이 변한다. 따라서 내가 예상했던 결과값과 다른 결과값이 나오게 된다.

이러한 경우를 부수효과가 있다고 표현한다.

또한 순수한 함수가 아니라고 표현한다.

## 참조 투명성

참조투명성 이란 프로그램의 변경없이 상황에 따라 결과값을 출력 할 수 있어야 하고 컴파일러나 개발자가 어떤 값이 출력될지 예상 할 수 있어야 한다. 

`

var name: String = "이승규"

fun getName(): String { 
  return name
}

fun getName(name: String): String { 
  return name
}

`

첫번째 함수는 다른이름을 출력하기 위해서는 결국 name 이라는 변수를 수정해야 하며 개발자가 어떠한 값이 출력이 될 지 추론하기 힘들다.

하지만 두번째 함수는 다른이름을 출력하기 위해서는 매개변수의 값만을 다르게 넣어주면 되기 때문에 프로그램의 수정이 필요 없다. 또한 추론하기도 쉽다.

두번째와 같은 경우를 참조 투명성이 있다라고 표현한다.

## 일급함수 

1. 매개변수로 함수형을 받을 수 있어야 한다.

2. 함수의 리턴값이 함수형이다.

3. 함수를 변수나 자료구조에 담을 수 있어야 한다.

## 일급객체 

1. 매개변수로 객체를 받을 수 있어야 한다.

2. 함수의 리턴값이 객체이다.

3. 객체를 변수나 자료구조에 담을 수 있어야 한다.

이러한 특징을 활용해서 추상화를 하게 되면 간단하게 코드를 작성 할 수 있게 되고 유지보수가 쉽다.

일반적으로 계산기를 만든다고 가정해보면

`

class Calculator{
    
    public int pluscalculate(int num1,int num2){
        return num1+num2;
    }

    public int minuscalculate(int num1,int num2){
        return num1-num2;
    }
}

`

이런식으로 간단하게 만들 수 있다. 하지만 기능을 추가하기 위해서는 Calculator 객체를 개발자가 알아야 하며 객체의 수정이 필요하다.

하지만 추상화를 이용하면 

`

interface Calculator{
    public int calculate(int num1,int num2)
}

class PlusCalculator implements Calculator{
    public int calculate(int num1,int num2){
        return num1+num2;
    }
}
class MinusCalculator implements Calculator{
    public int calculate(int num1,int num2){
        return num1-num2;
    }
}

public class Main
{
	public static void main(String[] args) {
		Calculator maincalculator = new PlusCalculator();
		System.out.println(maincalculator.calculate(10,1));
		
		Calculator maincalculator1 = new MinusCalculator();
		System.out.println(maincalculator1.calculate(10,1));
	}
}

`

새로운 기능을 추가 한다고 가정 할 때 Calculator 객체를 알 필요도 수정 할 필요도 없다.

kotlin 으로 하면

`

class Calculator{

    fun calculate(operator: Char,num1: Int, num2: Int):Int = when(operator){
            '+'-> num1+num2
            '-'-> num1-num2
            else -> 0
        }
    }


fun main(args: Array<String>) {
    val cal = Calculator()
    
    println(cal.calculate('+',1,2))
    println(cal.calculate('-',1,2))
}

`

간단하게 만들 수 있지만 새로운 기능을 추가하기 위해서는 calculate 함수 내부를 변경해야 한다.

하지만 추상화를 이용해서 함수를 만들게 되면

`

interface Calculator{

    fun calculate(num1: Int, num2: Int) :Int
        
    }

class PlusCalculator : Calculator{
    
    override fun calculate(num1: Int, num2: Int): Int = num1+num2

}

class MinusCalculator : Calculator{
    
    override fun calculate(num1: Int, num2: Int): Int = num1-num2

}

class MainCalculator(val cal: Calculator){
    
    fun calculate(num1: Int, num2: Int): Int{
        return cal.calculate(num1,num2)
    }
}

fun main(args: Array<String>) {
    val cal:MainCalculator = MainCalculator(MinusCalculator())
    
    println(cal.calculate(1,2))
    
}

`

인터페이스를 만들고 MainCalculator 에서 의존성을 주입 받기 때문에 어떤 calculator class 가 있는지 몰라도 된다.

위의 방식에서 함수형 프로그래밍 방식으로 바꾸게 되면

`

class MainCalculator{
    
    fun calculate(cal: (Int,Int) -> Int,num1: Int, num2: Int): Int{
        if(num1 >= num2){
            return cal(num1,num2)
        }
        else{
            throw IllegalArgumentException() 
        }
    }
}

fun main(args: Array<String>) {
    val cal:MainCalculator = MainCalculator()
    
    println(cal.calculate({n1,n2 -> n1+n2},3,2))
    println(cal.calculate({n1,n2 -> n1-n2},3,2))
    
}

`

인터페이스가 없고 새로운 기능이 추가 될 때마다 클래스를 만들 필요가 없어서 간단하다.

## 게으른 평가

게으른 평가란 값이 필요한 시점에 평가되는 것이다.

`
val x:String by lazy {
println("hihi")
"hello"
}

fun main(args: Array<String>) {
    
    println(x)
    println(x)
    
}
`

이런식으로 하게 되면 선언시에 호출 되는것이 아니라 실제로 변수를 호출하게 되면 실행이 된다.

또한 변수를 한번 실행하면 중복해서 실행 되지 않는다.
