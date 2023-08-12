# Channels

`Channel` 은 서로 다른 코루틴 간의 통신(데이터 교환)을 할 수 있는 메커니즘임.

### Channels 기초

- `Channel` 은 개념적으로 `BlockingQueued`와 유사함.
    - 블로킹 연산인 `put` 대신 일시 중단 연산인 `send` 를 가지고
    - 블로킹 연산인 `take` 대신 일시 중단 연산인 `receive` 를 가짐.

```kotlin
BlockingQueue란?
Java의 동시성 패키지에 포함된 인터페이스로, 스레드 간에 안전하게 항목을 추가하거나 제거할 수 있는 대기열(queue)을 제공함.
```

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

fun main(args: Array<String>) = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch { for(x in 1..5) channel.send(x) }
    repeat(5) { println(channel.receive()) }
    println("Done!")
}

// 실행결과 
1
2
3
4
5
Done!
```

- `send` : 채널에 데이터를 전송함.
    - 전송하려고 할 때, 해당 채널에 데이터를 수신할 준비가 되지 않은 `receive` 가 없다면, 일시 중단됨.
    - 이후 다른 코루틴에서 `receive` 로 채널로부터 데이터 수신할 준비가 되면, `send` 를 호출한 코루틴이 재개됨.
- `receive` : 채널로 부터 데이터를 수신함.
    - 데이터를 수신하려고 시도할 때, 채널에 데이터가 없다면 `receive` 를 호출한 코루틴이 일시중지됨.
    - 다른 코루틴에서 `send` 를 호출하여 데이터를 전송하면, `receive` 를 호출한 코루틴이 다시 재개됨.

```kotlin
interface Channel<E> : SendChannel<E> , ReceiveChannel<E>
```

- `Channel` 은 `SendChannel` , `ReceiveChannel` 을 구현하고 있음.
    - `SendChannel` : 데이터를 채널로 전송하는 연산들을 정의함.
    - `ReceiveChannel` : 데이터를 채널에서 수신하는 연산들을 정의함.

### Channel 닫기와 반복적으로 수신하기

- `Channel` 은 원소들이 오지 않는 다는 것을 알리기 위해 닫힐 수 있음.
    - `close` 함수 사용
        - 이는 `Channel` 에 특별한 close token(닫기 토큰)을 보냄.
        - `Channel` 은 for loop를 사용해서 원소를 편하게 받을 수 있는데, `close` 함수를 사용해서 채널을 닫으면 반복이 멈춤.
        - 즉 `close` 는 데이터의 전송이 끝났음을 명시적으로 알려주는 방법임.
    
    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.Channel
    
    fun main(args: Array<String>) = runBlocking<Unit> {
        val channel = Channel<Int>()
        launch {
            for (x in 1..5) channel.send(x * x)
            channel.close() // 데이터 전송 끝!
        }
    		// for loop로 편하게 받기. close 호출 시 반복이 멈춤
        // 채널에 데이터가 없으면 데이터가 도착할 때 까지 일시중단함.
        for (y in channel) println(y) 
        println("Done!")
    }
    ```
    
    ### Producer로 Channel 만들기
    
    - 코루틴이 원소들의 시퀀스를 생성하는 패턴은 일반적임.
        - 이는 동시성 코드에서 자주 발견되는 `생산자 - 소비` 패턴의 일부임.
    - 생산자는 `produce` 라는 코루틴 빌더를 통해 간단하게 데이터를 생산할 수 있음.
    - 소비자는 for loop를 `consumeEach` 확장 함수로 대체하여 데이터의 소비를 대체함.
    
    ```kotlin
    생산자-소비자 패턴이란?
    생산자가 데이터를 생성하고, 소비자가 생산한 데이터를 소비하는 구조의 패턴임.
    
    send와 receive 만을 사용하여도 데이터를 전송하고 수신하는 기본적인 작업은 가능하지만
    데이터의 생산과 소비를 분리하고 명확하게 관리하기 위해서 사용됨.
    
    코드의 구조가 더 명확해지며, 필요에 따라 다른 생산자나 소비자를 쉽게 추가, 변경이 가능.
    ```
    
    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.*
    
    fun CoroutineScope.produceSquares(): ReceiveChannel<Int> = produce {
        for (x in 1..5) send(x * x)
    }
    
    fun main() = runBlocking {
        val squares = produceSquares()
        squares.consumeEach { println(it) }
        println("Done!")
    }
    ```
    
    - 생산자의 반환 값을 `ReceiveChannel` 로 지정해주면서 `consumeEach` 사용시 소비가 가능하도록 구현함.
        - `produce` 의 반환타입은 `ReceiveChannel` 임.
    
    ```kotlin
    fun <E> CoroutineScope.produce(
        context: CoroutineContext = EmptyCoroutineContext, 
        capacity: Int = 0, 
        block: suspend ProducerScope<E>.() -> Unit
    ): ReceiveChannel<E>
    ```
    
    ### 파이프 라인
    
    - 하나의 코루틴이 값의 스트림을 생성하는 것을 뜻함.
        - 값의 스트림은 무한할 수도 있음.
    
    ```kotlin
    fun CoroutineScope.produceNumbers() = produce<Int> {
        var x = 1
        while (true) send(x++)
    }
    ```
    
    - 그리고 다른 코루틴들이 그 스트림(파이프 라인)을 소비하고, 작업을 수행하고 새로운 결과를 생성할 수도 있음.
    
    ```kotlin
    fun CoroutineScope.square(numbers: ReceiveChannel<Int>): ReceiveChannel<Int> = produce {
        for (x in numbers) send(x * x)
    }
    ```
    
    - 파이프 라인 연결
    
    ```kotlin
    fun main() = runBlocking {
        val numbers = produceNumbers()
        val squares = square(numbers)
        repeat(5) {
            println(squares.receive()) 
        }
        println("Done!")
        coroutineContext.cancelChildren() // 자식 코루틴 취소
    }
    ```
    
    ### Fan-out
    
    - 복수의 코루틴을 같은 채널로 수신(`receive`)하면서, 작업을 분산할 수 있음.
    
    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.*
    
    fun CoroutineScope.produceNumbers() = produce<Int> {
        var x = 1 // 
        while (true) {
            send(x++) 
            delay(100) 
        }
    }
    
    fun CoroutineScope.launchProcessor(id: Int, channel: ReceiveChannel<Int>) = launch {
        for (msg in channel) {
            println("Processor #$id received $msg")
        }
    }
    
    fun main() = runBlocking {
        val producer = produceNumbers()
        repeat(5) { launchProcessor(it, producer) } // 5개의 코루틴이 만들어짐.
    		delay(950)
        producer.cancel()
    }
    ```
    
    - 생산자 코루틴을 취소하면 코루틴의 채널은 닫힘.
        - 소비자 측의 채널의 반복이 종료됨.
    - `consumerEach` 는 정상작동, 비정상적으로 완료될 때, 언제나 해당 채널을 소비함.
        - 위 예제에선 for loop를 사용해 채널에 대한 반복을 수행했는데, 복수의 코루틴에 사용하기에 안전함.
            - launchProcessor에서 코루틴이 실패하면 다른 코루틴이 채널에 대한 처리를 할 것임.
    
    ### Fan-in
    
    - 복수의 코루틴이 같은 채널에 값을 보낼(`send`) 수 있음.
    
    ```kotlin
    suspend fun sendString(channel: SendChannel<String>, s: String, time: Long) {
        while (true) {
            delay(time)
            channel.send(s)
        }
    }
    
    fun main() = runBlocking {
        val channel = Channel<String>()
        launch { sendString(channel, "foo", 200L) }
        launch { sendString(channel, "BAR!", 500L) }
        repeat(6) { // receive first six
            println(channel.receive())
        }
        coroutineContext.cancelChildren()
    }
    ```
    
    ### ****Buffered channels****
    
    - 버퍼가 되지 않은 채널은 발신자와 수신자가 서로 만날 때 값을 전송함.
        - 이를 랑데뷰(`Rendezvous`)라고도 불림.
    - `send` 가 먼저 실행되면, `receive` 가 실행될 때까지 일시 중단됨.
    - `receive` 가 먼저 실행되면, `send` 가 실행될 때까지 일시 중단됨.
    - `Channel`과 `produce` 빌더 모두 버퍼의 크기를 정하기 위해 `capacity` 파라미터를 줄 수 있음.
    - 버퍼는 지정된 `capacity` 만큼 용량을 두고, 발신자가 일시 중단 전에 복수의 원소들을 보낼 수 있도록 함.
        - 이는 버퍼가 꽉차면 중단된다.
    
    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.*
    
    fun main() = runBlocking {
        val channel = Channel<Int>(4)
        val sender = launch {
            repeat(10) {
                println("Sending $it")
                channel.send(it)
            }
        }
        repeat(10) {
            println(channel.receive())
        }
    
        delay(1000)
        sender.cancel() // cancel sender coroutine
    }
    
    // 실행결과
    Sending 0
    Sending 1
    Sending 2
    Sending 3
    Sending 4
    Sending 5
    0
    1
    2
    3
    4
    5
    Sending 6
    Sending 7
    Sending 8
    Sending 9
    6
    7
    8
    9
    ```
    
    - 버퍼의 `capacity` 만큼 `send`를 하고, 버퍼가 꽉 찼을 경우 일시중단 되면서 `receive` 를 통해 버퍼에 저장된 값을 받을 수 있음.
    
    ### Channel은 평등하다
    
    - 채널로 보내고 받는 작업은 복수의 코루틴을 호출하는 순서에 대해 공정함.
    - 채널은 FIFO 구조로 제공되며, `receive` 를 호출하는 코루틴이 원소를 갖게됨.
    
    ```kotlin
    data class Ball(var hits: Int)
    
    suspend fun player(name: String, table: Channel<Ball>) {
        for (ball in table) { // receive the ball in a loop
            ball.hits++
            println("$name $ball")
            delay(300) // wait a bit
            table.send(ball) // send the ball back
        }
    }
    
    fun main() = runBlocking {
        val table = Channel<Ball>() // a shared table
        launch { player("ping", table) }
        launch { player("pong", table) }
        table.send(Ball(0)) // serve the ball
        delay(1000) // delay 1 second
        coroutineContext.cancelChildren() // game over, cancel them
    }
    ```
    
    - 비동기로 실행되는 ping, pong 두 코루틴에 대해서 누가 먼저 실행 되는진 알 수 없음.
    - 하지만 먼저 들어오는 코루틴에 대해서 `receive` 를 하고 그 다음 코루틴이 실행됨.
    
    ### ****Ticker channels****
    
    - 이는 채널에서 마지막으로 소비가 일어나고 일정 시간이 지난 후에 `Unit`을 생성하는 특별한 랑데뷰 채널임.
    - 시간에 의존적인 처리를 하는데 유용함.
        - 주기적으로 이벤트를 생성할 때 사용.
    
    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.*
    
    fun main() = runBlocking {
        val tickerChannel = ticker(delayMillis = 1000, initialDelayMillis = 0) // 1초마다 이벤트 발생
        var tickCount = 0
    
        val tick = launch {
            for (event in tickerChannel) { // 이벤트가 발생할 때마다 루프 실행
                println("Tick ${++tickCount}")
                if (tickCount >= 5) {
                    tickerChannel.cancel() // 5번 틱 후 채널 종료
                }
            }
        }
        tick.join()
    }
    ```
    
    ```kotlin
    import kotlinx.coroutines.*
    import kotlinx.coroutines.channels.*
    
    fun main() = runBlocking<Unit> {
        val tickerChannel = ticker(delayMillis = 100, initialDelayMillis = 0) 
        var nextElement = withTimeoutOrNull(1) { tickerChannel.receive() }
        println("Initial element is available immediately: $nextElement") 
    
        nextElement = withTimeoutOrNull(50) { tickerChannel.receive() } 
        println("Next element is not ready in 50 ms: $nextElement")
    
        nextElement = withTimeoutOrNull(60) { tickerChannel.receive() }
        println("Next element is ready in 100 ms: $nextElement")
        
        println("Consumer pauses for 150ms")
        delay(150)
      
        nextElement = withTimeoutOrNull(1) { tickerChannel.receive() }
        println("Next element is available immediately after large consumer delay: $nextElement")
        
        nextElement = withTimeoutOrNull(60) { tickerChannel.receive() }
        println("Next element is ready in 50ms after consumer pause in 150ms: $nextElement")
    
        tickerChannel.cancel()
    }
    ```
    
    ```kotlin
    Initial element is available immediately: kotlin.Unit
    Next element is not ready in 50 ms: null
    Next element is ready in 100 ms: kotlin.Unit
    Consumer pauses for 150ms
    Next element is available immediately after large consumer delay: kotlin.Unit
    Next element is ready in 50ms after consumer pause in 150ms: kotlin.Unit
    ```
    
    - `withTimeoutOrNull` 은 주어진 시간 내에 작업을 완료할 수 없으면 null을 반환함.
        - tickerChannel은 100ms 마다 `Unit`을 발생.
