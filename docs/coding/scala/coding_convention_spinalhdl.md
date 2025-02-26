---
tags:
- coding
- scala
- naming
---
# Coding Convention for SpinalHDL

## Introduction

The coding conventions used in SpinalHDL are the same as the ones documented in the [scala doc](https://docs.scala-lang.org/style/).

Some additional practical details and cases are explained in next chapters.

## class vs case class

When you define a `Bundle` or a `Component`, have a preference to declare them as `case class`.

The reasons are:

- It avoids the use of `new` keywords. Never having to use it is better than sometimes under some conditions.
- A `case class` provides an `clone` function. This is useful in SpinalHDL where there is a need to clone one Bundle. For example, when you define a new `Reg` or a new `Stream` of some kind.
- Construction parameters are directly visible from outside.

## case class

Case class is an alternative way of declaring classes.

``` scala
case class Rectangle(width: Float, height: Float) extends Shape {
  override def getArea() = width * height
}
```

Tere are some differences between `case class` and `class` :

- case classes don't need the `new` keyword to be instantiated
- construction parameters are accessible from outside, you don't need to define them as `val`.

In SpinalHDL, this explains the reasoning behind the coding conventions: it's in general recommended to use `case class` instead of `class` in order to have less typing and more coherency.

### `case` class

All classes names should start with a **upper case** letter

``` scala
class Fifo extends Component {

}

class Counter extends Area {

}

case class Color extends Bundle {

}
```

### companion object

A companion object should start with a **upper case** letter.

``` scala
object Fifo {
  def apply(that: Stream[Bits]): Stream[Bits] = {...}
}

object MajorityVote {
  def apply(that: Bits): UInt = {...}
}
```

An exception to this rule is when the companion object is used as a function (only `apply` inside), and these `apply` functions don't generate hardware:

``` scala
object log2{
  def apply(value: Int): Int = {...}
}
```

### function

A function should always start with a **lower case** letter:

``` scala
def sinTable = (0 until sampleCount).map(sampleIndex => {
  val sinValue = Math.sin(2 * Math.PI * sampleIndex / sampleCount)
  S((sinValue * ((1 << resolutionWidth) / 2 - 1)).toInt, resolutionWidth bits)
})

val rom =  Mem(SInt(resolutionWidth bit),initialContent = sinTable)
```

### instances

Instances of classes should always start with a **lower case** letter:

``` scala
val fifo   = new Fifo()
val buffer = Reg(Bits(8 bits))
```

### if / when

Scala `if` and SpinalHDL `when` should normally be written in the following way:

``` scala
if(cond){
  ...
} else if(cond){

} else {

}

when(cond){

}.elseWhen(cond){

}.otherwise{

}
```

### switch

SpinalHDL switch should normally be written in the following way:

``` scala
switch(value){
  is(key){

  }
  is(key){

  }
  default{

  }
}
```

### Parameters

Grouping parameters of a component/bundle inside a case class is in general welcome:

-   Easier to carry/manipulate to configure the design
-   Better maintainability

``` scala
case class RgbConfig(rWidth: Int, gWidth: Int, bWidth: Int){
  def getWidth = rWidth + gWidth + bWidth
}

case class Rgb(c: RgbConfig) extends Bundle {
  val r = UInt(c.rWidth bit)
  val g = UInt(c.gWidth bit)
  val b = UInt(c.bWidth bit)
}
```

But this should not be applied in all cases. For example: in a Fifo, it doesn't make sense to group the dataType parameter with the depth of the fifo because, in general, the dataType is something related to the design, while the depth is something related to the configuration of the design.

``` scala
class Fifo[T <: Data](dataType: T, depth: Int) extends Component {

}
```
