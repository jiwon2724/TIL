# SOLID

## 단일 책임 원칙(Single Responsibility Principle)

- 모든 클래스는 하나의 책임만 가지며, 클래스는 그 책임을 완전히 캡슐화해야함.
    - 클래스가 제공하는 모든 기능은 이 책임과 부합해야함.
- 단일 책임은 어떤 클래스나 모듈, 메서드가 단 하나의 기능을 가져야 한다는 뜻.
    - 변경 사항이 발생하더라도 변경 사항에 대한 책임이 있는 부분만 수정하면 됨.

---

특정 데이터를 분석하고 서버에 전송하는 모듈을 예시로 들어보자.

- 데이터를 분석하는 알고리즘
- 서버에 전송하는 형식

이는 위 두가지 이유로 변경될 수 있음.

```kotlin
단일 책임 원칙에 의하면 두 책임을 분리된 클래스나 모듈로 나눠야함.
다른 시기에 다른 이유로 변경되어야 하는 두 가지를 묶는 것은 나쁜 설계일 수 있음.
```

한 클래스를 한 관심사에 집중하도록 유지하는 것은 중요함. 

→ 클래스를 더욱더 튼튼하게 해줌.

## 개방-폐쇄 원칙(Open Closed Principle)

- 소프트웨어가 확장에 대해서는 열려 있어야 하고, 수정에 대해서는 닫혀있어야함.
- 시스템의 구조를 올바르게 구성하여 변경사항이 발생해도, 모듈에 영향이 없도록 하는 것.
- 새로운 기능을 추가하거나 기존 기능을 변경하기 용이함.
- 해당 원칙을 무시한 경우엔 객체지향의 장점인 유연성, 재사용성, 유지 보수성을 얻을 수 없음.

---

기존의 코드를 변경하지 않으면서, 기능을 추가할 수 있도록 설계가 되어야함. 다음 예시를 보자.

```kotlin
fun main(args: Array<String>) {
    val hello = HelloAnimal()

    val cat = Animal("Cat")
    val dog = Animal("Dog")

    hello.hello(cat)
    hello.hello(dog)
}

class Animal(val type: String)
class HelloAnimal {
    fun hello(animal: Animal) {
        when(animal.type) {
            "Cat" -> println("냐옹")
            "Dog" -> println("멍멍")
        }
    }
}
```

- 다음은 Animal 타입을 받고, 해당 동물에 대한 울음소리를 출력하는 예시임.
- 위 코드는 문제 없이 동작하지만, 확장(추가)시 문제가 발생함.
    - 기존의 코드를 변경해야함.
- 만약 여기서 다른 동물을 추가해야한다면? `HelloAnimal` 클래스 when 조건에 동물 타입이 나열됨.
    - 동물이 계속 추가될 때 마다 코드를 계속 추가해야함. → 잘못된 설계

**올바른 OCP 설계는 다음과 같음.**

1. 확장될 것과 변하지 않을 것을 구분
2. 두 모듈이 만나는 지점에 추상화(추상클래스 or 인터페이스)를 정의
3. 구현체에 의존하기 보단 정의한 추상화에 의존하도록 코드를 작성

**OCP를 적용한 코드는 다음과 같음.**

```kotlin
fun main(args: Array<String>) {
    val hello = HelloAnimal()

    val cat = Cat()
    val dog = Dog()

    hello.hello(cat)
    hello.hello(dog)
}

abstract class Animal {
    abstract fun speak()
}

class Cat : Animal() {
    override fun speak() {
        println("냐옹")
    }
}

class Dog : Animal() {
    override fun speak() {
        println("멍멍")
    }
}

class HelloAnimal {
    fun hello(animal: Animal) {
        animal.speak()
    }
}
```

기존 코드 수정없이 확장(새로운 동물 추가)에는 열려있음.

## 리스코프 치환 원칙(Liskov Substitution Principle)

- 치환성은 객체 지향 프로그래밍의 원칙임. → 다형성을 지원하기 위한 원칙
- 클래스 S가 클래스 T의 자식 클래스라면, 별다른 변경 없이 부모 클래스 T를 자식 클래스 S로 치환이 가능해야함. → 다운 캐스팅된 인스턴스가 논리적으로 문제가 없어야함.
    - 변성과 연관되어있음.
- 업 캐스팅 된 상태에서 부모의 메서드를 사용해도 의도대로만 흘러가도록 구성해야함.
    - 부모 클래스를 상속된 상황도 마찬가지임.
- 컬렉션 프레임워크가 LSP를 잘 적용한 예시임.

## 인터페이스 분리 원칙(Interface Segregation Principle)

- 어떤 클래스가 자신이 이용하지 않는 메서드에 의존하지 않아야 한다는 원칙.

```kotlin
abstract class Dog {
    open fun bark() { ... } // 짖기
    open fun sniff() { ... } //냄새 맡기
}

class RobotDog : Dog() {
    override fun sniff() {
        throw Error("냄새맡기 불가능")
    }
}
```

- 위 코드는 Dog 클래스를 상속받는 RobotDog 클래스가 있음. 이는 로봇 강아지라 냄새는 못맡음.
- 위 같은 코드는 RobotDog가 필요없는 메서드를 가지게됨.
    - 인터페이스 분리 원칙에 위반되고, 슈퍼 클래스에서 서브 클래스의 동작도 깨버리므로, 리스코프 치환 원칙에도 위반됨.

## 의존 역전 원칙(Dependency Inversion Principle)

- 모듈들을 분리하는 특정 형식을 지칭함.
- 상위 계층이 하위 계층에 의존하는 의존관계를 역전 시킴.
    - 상위 계층이 하위 계층의 구현으로부터 독립되게 할 수 있음.
- 의존 역전 원칙은 다음과 같은 내용을 내포하고있음.
    - 상위 모듈은 하위 모듈에 의존해서는 안됨.
        - 상위 모듈과 하위 모듈 모두 추상화에 의존해야함.
    - 추상화는 세부 사항에 의존해선 안됨.
        - 세부 사항이 추상화에 의존해야함.

즉, 상위, 하위 객체 모두가 동일한 추상화에 의존해야함.

```kotlin
// 상위 계층
class Zoo {
    val dog: Dog = Dog()
    val cat: Cat = Cat()
}

// 하위 계층
class Cat() { ... }
class Dog() { ... }
```

- 동물원 클래스에 강아지와 고양이의 의존성이 존재함. 상위 계층이 하위 계층이 가지고 있는건 자연스러운 상황임.
- 상위 계층(Zoo)에 하위 계층(추가되는 동물들)의 의존성이 많아질수록 코드의 수정과 관리가 어려워짐.

```kotlin
fun main(args: Array<String>) {
    val zoo: Zoo = Zoo()
    val dog: Dog = Dog()
    val cat: Cat = Cat()
    
    zoo.addAnmial(dog)
}

abstract class Animal { abstract fun speak() }
class Dog: Animal() { override fun speak() { ... } }
class Cat: Animal() { override fun speak() { ... } }

class Zoo {
    val animalList = arrayListOf<Animal>()

    fun addAnimal(animal: Animal) {
        animalList.add(animal)
    }
}
```

- 고양이와 강아지 클래스는 Animal 클래스의 의존성을 가지고 있음.
- 상위 계층인 Zoo클래스도 하위계층의 의존성을 가지고 있지 않음.
- 새로운 동물이 추가되더라도, 상위 계층은 영향을 받지 않음.
- Animal 추상 클래스로 상위, 하위 클래스 모두 추상화에 의존하고있음.

![스크린샷 2023-07-30 오후 10 54 00](https://github.com/jiwon2724/TIL/assets/70135188/8ebc8873-1e46-4015-9044-766f0dcce47f)


DIP 적용시 의존의 방향이 바뀌기 때문에 의존 역전 원칙이라부름.
