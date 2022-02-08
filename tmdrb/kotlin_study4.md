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

## optional모나드

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

## Either모나드

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
## future 모나드

callback 패턴을 사용하게 되면 코드가 복잡해지고 지저분해지는 결과가 발생

그것을 해결하기 위해서 사용

## callback 이란?

callback란 비동기 방식으로 데이터를 주고 받을 때 일반적으로 caller 가 피호출자를 호출하는 형식인데 반대로 피호출자가 caller를 호출하는 패턴이다.

비동기 vs 동기

동기방식은 A가 계속해서 B를 확인해서 처리하는 방식이다.

하지만 비동기 방식은 A,B가 존재 한다면 A가 B가 필요 할 때마다 B한테 요청하면 응답을 받는 방식이다.

callback는 비동기 방식중 하나의 패턴이다.

A 클래스가 B 클래스에 요청을 하게 되면 응답을 하게 되지만 B 클래스에서 A 클래스 내부에 있는 함수에 접근을 하기 위해서는 A의 인스턴스를 사용하게 되면 강한결합 발생

자바에서는 함수를 변수로 사용 할 수 없기 때문에 인스턴스를 만들어서 B로 전달 한뒤 사용

코틀린에서는 future 모나드를 이용해서 callback을 처리

```
class Future<Err, V>(private var scheduler: Scheduler = SchedulerIO) {
    var subscribers: MutableList<Callback<Err, V>> = mutableListOf()
    private var cache: Optional<Either<Err, V>> = Optional.None()
    var semaphore = Semaphore(1)

    private var callback: Callback<Err, V> = { value ->
        semaphore.acquire()
        cache = Optional.Some(value)
        while (subscribers.size > 0) {
            val subscriber = subscribers.last()
            subscribers = subscribers.dropLast(1).toMutableList()
            scheduler.execute {
                subscriber.invoke(value)
            }
        }
        semaphore.release()
    }

    fun create(f: (Callback<Err, V>) -> Unit): Future<Err, V> {
        scheduler.execute {
            f(callback)
        }
        return this
    }
    
    fun subscribe(cb: Callback<Err, V>): Disposable {
        semaphore.acquire()
        when (cache) {
            is Optional.None -> {
                subscribers.add(cb)
                semaphore.release()
            }
            is Optional.Some -> {
                semaphore.release()
                val c = (cache as Optional.Some<Either<Err, V>>)
                cb.invoke(c.value)
            }
        }
        return Disposable()
    }

    fun <P> map(functor: (value: V) -> P): Future<Err, P> {
        return this.flatMap { value ->
            Future<Err, P>().create { callback ->
                callback(Either.Right(functor(value)))
            }
        }
    }

    fun <Q> flatMap(functor: (value: V) -> Future<Err, Q>): Future<Err, Q> {
        return Future<Err, Q>().create { callback ->
            this.subscribe { value ->
                when (value) {
                    is Either.Left -> {
                        callback(Either.Left(value = value.value))
                    }
                    is Either.Right -> {
                        functor(value.value).subscribe(callback)
                    }
                }
            }
        }
    }

    inner class Disposable {
        fun dispose() {
            scheduler.shutdown()
        }
    }
}
```



 

