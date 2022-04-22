scala3の削除機能

## DelayInit = App trait

https://docs.scala-lang.org/scala3/reference/dropped-features/delayed-init.html


`App` をextendsする方法が廃止され、 `def main` を定義するか `@main` アノテーションによりmainメソッドを決定する
```scala
// scala2
object HelloWorld extends App {
  println("Hello, world!")
}
```

```scala
// scala3
object HelloWorld:
  def main(args: Array[String]) =
    println(s"Hello, ${args(0)}")
```
or
```scala
// scala3
object HelloWorld:
  @main def program =
    println(s"Hello, scala3")
```

## Existential Types

https://docs.scala-lang.org/scala3/reference/dropped-features/existential-types.html

そもそもExistential Typesって何だっけ？と思ったが

以下のように書いて、左辺から型パラメータを消してしまう書き方
```scala
// scala2
final case class SomeClass[A]()

type F = SomeClass[A] forSome { type A }

SomeClass(1): F
SomeClass("abc"): F
SomeClass(true): F
```

scala3では `forSome` 自体が無くなった。

ただしワイルドカードで表現できるExistential Typesはサポートされる。
```scala
// scala3
final case class SomeClass[A](a: A)

type F = SomeClass[_]

SomeClass(1) : F
SomeClass("abc"): F
SomeClass(true): F
```

## Procedure Syntax

https://docs.scala-lang.org/scala3/reference/dropped-features/procedure-syntax.html

Scalaメインで書く場合使う事はほぼ無いと思うが以下の書き方
```scala
// scala2
def f() {...}
```

必ず `=` ありのSyntaxが必須になる

```scala
// scala3
def f() = {...}
def f(): Unit = {...}
```

## Class shadowing

https://docs.scala-lang.org/scala3/reference/dropped-features/class-shadowing.html

内部classのoverrideは元々不可だが、あたかもできるかのように書く事ができた。
scala3では内部classはユニークな名前を強制される(同一名称はcompileエラーになる)。

```scala
// scala2
class Base {
  class Ops {
    println("Base.Ops")
  }
  new Ops()
}

class Sub extends Base {
  class Ops {
    println("Sub.Ops")
  }
  new Ops()
}

//ちなみにnew Sub()を呼び出すと
// ---
// Base.Ops
// Sub.Ops
// ---
// のように両方呼び出され、overrideはされない
```

```scala
// scala3
class Base:
  class Ops:
    println("Base.Ops")

  new Ops()

class Sub extends Base:
  class Ops:
    println("Sub.Ops");

  new Ops()
```
```sh
# compileするとエラーになる
[info] compiling 1 Scala source to /path/to/Main.scala
[error] -- [E069] Syntax Error: ... 
[error] 8  |  class Ops:
[error]    |        ^
[error]    |class Ops cannot have the same name as class Ops in class Base -- class definitions cannot be overridden
```

