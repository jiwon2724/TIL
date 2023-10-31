# CHAPTER 13 : 파일 입출력과 스레드 처리

## 파일 I/O 처리

### 스트림 및 버퍼 처리

- 파일 처리는 `Input`과 `Output`에 대한 데이터 처리임. 이런 데이터가 계속 처리되어 흐르는 것과 같아 스트림(`Stream`)이라고 함.
- 이는 바이트나 텍스트 단위로 처리해서 사람이 인식할 수 있게 만들어야함.
- I/O 처리 성능을 향상하기 위해 중간에 저장공간을 두고 처리하는 방식을 버퍼(`Buffer`) 처리라고 함.
- 기본적으로 파일을 읽고 쓰는 기반은 `InputStream`, `OutputStream`임.
    - 이를 기반으로 바이트 단위로 처리하도록 지원하는 것이 `ByteArrayInputStream`, `ByteArrayOutputStream`임.
- 바이트 스트림은 기본적으로 데이터를 읽고 쓰는 것임.
    - `read` 메서드로 바이트 단위로 읽음.
    - `write` 메서드로 바이트 단위로 사용함.
- 스트림 처리의 기본은 하나 입력하면 하나를 출력함.
    - 중간에 버퍼를 지정하고 처리하면 빠르게 입출력 가능

### 파일 처리 : 읽기

- I/O의 기본은 스트림이므로 이를 상속한 파일도 스트림으로 처리함.
- 파일은 `FileInputStream`의 `read` 메서드로 바이트 단위로 읽음.
- 읽은 파일을 다른 파일에 저장하려면 while을 사용하여 한 바이트씩 읽고, 한 바이트씩 파일에 저장.
    - 이땐 `FileOutputStream`이 `write` 메서드를 사용.
- 파일 처리가 끝나면 항상 `close` 메서드로 닫아야함.

```kotlin
import java.io.FileInputStream
import java.io.FileOutputStream

fun main() {
    val fileInputStream = FileInputStream("/Users/jiwon_dev/Desktop/data.txt")
    val fileOutputStream = FileOutputStream("/Users/jiwon_dev/Desktop/dataout.txt")

    var data = fileInputStream.read()

    while (data != -1) {
        fileOutputStream.write(data)
        data = fileInputStream.read()
    }

    fileInputStream.close()
    fileOutputStream.close()

    val fileInputResultStream = FileInputStream("/Users/jiwon_dev/Desktop/dataout.txt")
    var readData = fileInputResultStream.read()
    while(readData != -1) {
        print(readData.toChar())
        readData = fileInputResultStream.read()
    }
    fileInputResultStream.close()
}

실행결과 : test
```

### 버퍼리더를 사용해 파일 처리

- 입력 스트림으로 객체를 생성하고 버퍼를 사용해 텍스트로 파일을 읽음.
- `File`로 파일을 열고 `InputStream`으로 변경

```kotlin
fun main() {
    val fileInputStream = File("/Users/jiwon_dev/Desktop/data.txt").inputStream()
    val inputString = fileInputStream.bufferedReader().use { it.readLine() }
    println(inputString)
}

실행결과 : test test1 test2
```

### 파일 처리 : 쓰기

- `FileWriter`의 인자에 파일 경로를 전달하여 파일을 생성함.
- `BufferedWriter`에 전달해서 버퍼로 만든 후 파일을 작성하고 작성이 완료되면 파일을 닫음.

```kotlin
fun main() {
    val fw = FileWriter("만들어짐.txt")
    BufferedWriter(fw).apply {
        append("test!!!!!!!!!!!!!!!!!!\ntest@@@@@@@@@@@@@@@@@")
        flush()
        close()
    }
    val lines = File("만들어짐.txt").readLines()
    println(lines)
}

실행결과 : [test!!!!!!!!!!!!!!!!!!, test@@@@@@@@@@@@@@@@@]
파일은 해당 코드를 빌드한 kt파일 폴더의 경로에 만들어짐.
```

### 파일 접근과 NIO 처리

- 파일에 접근할 때 버퍼, 비동기 방식을 지원하는 새로운 패키지임.
    - `Paths` : 경로 등을 확인한 후에 해당 파일의 존재 여부 등 다양한 메서드로 파일 정보를 확인 가능

### 스레드

- JVM은 기본적으로 멀티 스레드 방식으로 실행됨.
- 스레드를 작동하려면 함수나 메서드를 작성하여 스레드에서 실행.
- 스레드를 만들려면 `Thread` 클래스를 상속하거나, `Runnable` 인터페이스와 `Thread` 클래스를 사용하여 작성.
    - 다음은 스레드를 처리할 때 사용되는 주요 메서드임.
        - start : 실제 스레드 환경을 구성한 환경을 실행하는 메서드. 내부에 있는 `run` 메서드가 실행됨.
        - run : 스레드 클래스를 정의할 때, 내부에서 실행될 코드를 작성하는 메서드.
            - 스레드 실행은 `start` 메서드로 처리해야함.
        - join : 스레드가 실행된 다음 종료될 때까지 기다림.
        - sleep : 스레드를 잠시 중단하고 다른 스레드를 처리한 후 다시 자기 스레드를 작동할 수 있게 만듬.

### 스레드 생성

```kotlin
// Thread 클래스 사용
val thread = Thread {
    println("Thread is running.")
}
thread.start()

// Runnable 인터페이스 사용
val runnable = Runnable {
    println("Runnable is running.")
}
val thread = Thread(runnable)
thread.start()

// Thread 클래스를 상속 받아서 사용
class MyThread : Thread() {
    override fun run() {
        println("MyThread is running.")
    }
}
val myThread = MyThread()
myThread.start()

// 코틀린의 thread 함수 사용 (코틀린 표준 라이브러리)
val thread = thread(start = true) {
    println("Kotlin thread is running.")
}
```

### 스레드 풀 사용

- 스레드를 무작정 만들면 컴퓨터의 자원을 너무 많이 사용함.
- 특정 개수의 스레드를 풀(pool)로 만들어서 특정 스레드를 계속 활용.

```kotlin
import java.util.concurrent.Executors

fun main() {
    // 스레드 풀 생성 (여기서는 10개의 스레드를 가진 풀을 생성)
    val threadPool = Executors.newFixedThreadPool(10)

    for (i in 1..10) {
        threadPool.execute {
            // 여기에 각 스레드가 처리할 작업을 작성
            println("Task $i running in thread ${Thread.currentThread().name}")
        }
    }

    // 스레드 풀 종료 (진행 중인 작업은 마친 후 종료)
    threadPool.shutdown()
}
```

### 질문

1. 안드로이드에서 스레드를 개발자가 직접 생성해야하는 경우가 있나요?.?
