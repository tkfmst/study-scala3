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

## XML literals

https://docs.scala-lang.org/scala3/reference/dropped-features/xml.html

XML Literalsが廃止になってXML string interpolationを利用するようになる。  
ただし現在は使えず、今のところは引き続きXML Literalsを使う

## Symbol Literals

https://docs.scala-lang.org/scala3/reference/dropped-features/symlits.html

Symbolリテラルが廃止。  

```scala
// scala2
object Hello extends App {
  println('xyz)
}
```

```scala
// scala3
object Hello:
  def main(args: Array[String]): Unit =
    println('xyz)
```
```sh
# compileするとエラーになる
[error] -- Error: /path/to/Main.scala:3:12
[error] 3 |    println('xyz)
[error]   |            ^
[error]   |symbol literal 'xyz is no longer supported,
```

## Auto application

https://docs.scala-lang.org/scala3/reference/dropped-features/auto-apply.html

2.xでは本来argument list `()` が存在するメソッドを `()` で呼び出す事ができた。  
これはcompilerが `()` を保管していたためだが、scala3では常に定義と一致した呼び出しが必要になる。

```scala
// scala2
object Hello extends App {
  def next(): Int = 
  val n = next
}
```

```scala
// scala3
object Hello:
  def main(args: Array[String]): Unit =
    def next(): Int = 1
    val n = next
```
```sh
# compileするとエラーになる
[error] -- [E100] Syntax Error: /path/to/Main.scala:5:12
[error] 5 |    val n = next
[error]   |            ^^^^
[error]   |            method next must be called with () argument
```

ただしJavaで定義されたメソッドやそれをoverrideしたものは許容される。

```scala
// scala3

object Hello:
  def main(args: Array[String]): Unit =
    1.toString().length()
    1.toString.length
```


## Weak conformance

https://docs.scala-lang.org/scala3/reference/dropped-features/weak-conformance.html

ドキュメントの説明は非常に理解が難しい。  
そもそもscala2だと弱い適合性を意識する事が少ないので非常に難しい。

弱適合性について理解したい場合はscala2の[REFLECTION シンボル、構文木、型](https://docs.scala-lang.org/ja/overviews/reflection/symbols-trees-types.html)#弱適合性関係を読む(削除される機能だが・・・)

単純にscala2 -> scala3へmigrationするときに問題になる点を列挙すると

```scala
// scala2

val d: Double = foo() // OK
def foo() = if (true) 1f else 1d
```

```scala
// scala3

val d: Double = foo() // Error
def foo() = if (true) 1f else 1d
```
```sh
[error] -- [E007] Type Mismatch Error: /path/to/Main.scala:3:23
[error] 3 |    val d: Double = foo()
[error]   |                    ^^^^^
[error]   |                    Found:    AnyVal
[error]   |                    Required: Double
```

のような場合、 `Float` はコンパイル時に自動変換されず `AnyVal` として扱われるためエラーとなる。

しかしscala3のドキュメントに
> Therefore, Scala 3 drops the general notion of weak conformance, and instead keeps one rule: Int literals are adapted to other numeric types if necessary.

とある通り `Int` の場合はDoubleへ自動変換されるためcompile可能である。

```scala
// scala3

val d: Double = foo() // OK 
def foo() = if (true) 1 else 1d // <- from 1f to 1
```

また、関数に返り型が明示されている場合は自動変換され、コンパイルを通過する。  

これは複数の数値型を含む式で、返り型が明示されていれば、例えば `Flaot` -> `Double` のように高精度側への変換は[ `Float` に定義されている `implicit` メソッド](https://dotty.epfl.ch/api/scala/Float$.html#float2double-72d)によって暗黙的に変換される。  
が、明示的になっていない場合は `Int` 以外は `AnyVal` に変換されてしまうため、別の場所から特定の数値型として呼び出されても暗黙的に変換できずに compileエラーとなる。

ただ、基本的にこの問題は、全部 `Any` や `AnyVal` 使っている等がなければ compileでエラーとなるため、実行して初めて気が付くなどの問題としては遭遇しにくい。  
したがって問題が発生しても、その影響はアプリケーションに影響は与えにくいと思われる。


```scala
// scala3

val d: Double = foo() // OK
def foo(): Double = if (true) 1f else 1d // <- Add return type `Double`
```

ちなみにscala3ドキュメントに書いてる例も、返り型をきちんと書いておけば問題なくcompileできる。  
したがって基本返り型がきちんと書いてあるコードであればcompileできると思われる。  
また問題のある書き方の場合もcompile時にエラーになるので、大きな問題にはなりにくいと考えられる。  

NG
```scala
// scala3

def main(args: Array[String]): Unit =
  val ls: List[Double] = foo()

def foo() =  // NG: No return type
  val n: Int = 3
  val c: Char = 'X'
  val d: Double = math.sqrt(3.0)
  List(n, c, d)
```
```
[error] -- [E007] Type Mismatch Error: /path/to/Main.scala:3:30
[error] 3 |    val ls: List[Double] = foo()
[error]   |                           ^^^^^
[error]   |                           Found:    List[AnyVal]
[error]   |                           Required: List[Double]
```

OK
```scala
// scala3

def main(args: Array[String]): Unit =
  val ls: List[Double] = foo()

def foo(): List[Double] = // OK
  val n: Int = 3
  val c: Char = 'X'
  val d: Double = math.sqrt(3.0)
  List(n, c, d)
```
