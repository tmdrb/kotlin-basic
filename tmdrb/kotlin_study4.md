# Functor

펑터는 값을 컨테이너로 씌워서 전달해주는 것이다.

ex) 10 -> Functor(10) 이런식으로 

펑터를 사용하는 이유?

fmap을 사용해서 어떤 타입이던지 map 적용 가능하다.

if,else 를 사용하지 않아도 Functor 내부에서 값이 있는지 없는지 구별가능

null을 신경쓰지 않아도 저절로 판단해준다.

# applicative functor

# Monad

만약 값을 나누는 함수가 있다고 가정해보자

일반적으로 나누는것이 아닌 0 으로 나눌 경우에 exception이 발생한다.

하지만 순수함수 관점에서 보면 exception이라는 새로운 객체가 생성되었고 결국 사이드 이펙트가 발생한다.

순수함수를 유지하기 위해서 구조체와 같이 exception값과 원시타입을 묶어서 하나의 타입을 만들어 보자고 해서 생성 된 것이 모나드이다.

모나드의 궁극적인 목적은 함수의 합성이 가능하다.

함수내에서 예측하지 않은 값이 나오지 않기 때문에 계속해서 합성이 가능하다.

##optional모나드

값을 선택 할 수 있는 optional 모나드

```
sealed class Optional<T> {
    
    class None<T> : Optional<T>()
    data class Some<T>(val value:T) : Optional<T>()
}

infix fun <T,R> Optional<T>.map(functor:(value: T)->R): Optional<R> {

    return this.flatmap{ value -> Optional.Some(functor(value))}
}

infix fun <T,R> Optional<T>.flatmap(functor:(value: T) -> Optional<R>):Optional<R>{
    
    when(this) {
        is Optional.Some -> return functor(this.value)
        is Optional.None -> return Optional.None()
    } 
}
```

##Either모나드

optional과 비슷하지만 left에도 값을 담을 수 있는 either 모나드
```
sealed class Either<out L, out R> {
    
    data class Left<L>(val value: L) : Either<L, Nothing>()
    data class Right<R>(val value: R) : Either<Nothing,R>()
    
}

infix fun <L,R,P> Either<L,R>.map(functor:(value: R) -> P):Either<L,P>{
    
    return this.flatmap{ value -> Either.Right(functor(value))}
    
} 

infix fun <L,R,P> Either<L,R>.flatmap(functor:(value: R) -> Either<L,P>):Either<L,P>{
    
    when(this){
        is Either.Left -> return Either.Left(this.value)
        is Either.Right -> return functor(this.value)
        
    }
}

```
##future 모나드

