ScalaStorm provides a Scala DSL for Nathan Marz's [Storm](https://github.com/nathanmarz/storm) real-time computation system. It also provides a framework for Scala and SBT development of Storm topologies.

For example, here is the SplitSentence bolt from the word count topology:

```scala
class SplitSentence extends StormBolt(outputFields = List("word")) {
  def execute(t: Tuple) = t matchSeq {
    case Seq(sentence: String) => sentence split " " foreach
      { word => using anchor t emit (word) }
    t ack
  }
}
```

A couple things to note here:

* The matchSeq DSL enables Scala pattern matching on Storm tuples.  Notice how it gives you a nice way to name and identify the type of each component.  Now imagine the ability to match on different tuple types, like in a join, easily and elegantly!
* The emit DSL reads like English and easily takes multiple args (val1, val2, ...)
* Output fields are easily declared
* It's easy to see exactly when the emits and ack happen

Getting Started
===============

* Download [sbt](https://github.com/harrah/xsbt/wiki) version 0.10.1 or above
* clone this project
* In the root project dir, type `sbt run`.  SBT will automatically download all dependencies, compile the code, and give you a menu of topologies to run.

To help you get started, the ExclamationTopology and WordCountTopology examples from storm starter have been included.

Bolt DSL
========
The Scala DSL for bolts is designed to support many different bolt designs, including all 10 variants of the collector emit() and emitDirect() APIs.  Getting started consists of extending the StormBolt class, passing a list of output fields, and defining the execute method:

```scala
class ExclamationBolt extends StormBolt(outputFields = List("word")) {
  def execute(t: Tuple) = {
    t emit (t.getString(0) + "!!!")
    t ack
  }
}
```

matchSeq
--------
The `matchSeq` method passes the storm tuple as a Scala Seq to the given code block with one or more case statements. The case statements need to have Seq() in order to match the tuple.  If none of the cases match, then by default a handler which throws a RuntimeError will be used.  It is a good idea to include your own default handler.

matchSeq allows easy naming and safe typing of tuple components, and allows easy parsing of different tuple types.  Suppose that a bolt takes in a data stream from one source and a clock or timing-related stream from another source. It can be handled like this:

```scala
def execute(t: Tuple) = t matchSeq {
	case (username: String, followers: List[String]) => // process data
	case (timestamp: Int) =>   // process clock event
	case _ =>                  // default case, optional (but a good idea)
}
```

emit and emitDirect
-------------------
emit takes a variable number of AnyRef arguments which make up the tuple to emit.  emitDirect is the same but the first argument is the Int taskId, followed by a variable number of AnyRefs.

To emit a tuple anchored on one tuple, where t is of type Tuple, do one of the following:

```scala
using anchor t emit (val1, val2, ...)
using anchor t emitDirect (taskId, val1, val2, ...)
anchor(t) emit (val1, val2, ...)
t emit (val1, val2, ...)
```

To emit a tuple to a particular stream:

```scala
using anchor t toStream 5 emit (val1, val2, ...)
using anchor t toStream 5 emitDirect (taskId, val1, val2, ...)
```

To emit anchored on multiple tuples:

```scala
using anchor List(t1, t2) emit (val1, val2, ...)
using anchor List(t1, t2) emitDirect (taskId, val1, val2, ...)
```

To emit unanchored:

```scala
using no anchor emit (val1, val2, ...)
using no anchor emitDirect (taskId, val1, val2, ...)
using no anchor toStream 5 emit (val1, val2, ...)
using no anchor toStream 5 emitDirect (taskId, val1, val2, ...)
```

ack
---
```scala
t ack                // Ack one tuple
List(t1, t2) ack     // Ack multiple tuples, in order of list
```

Spout DSL
=========
The Scala Spout DSL is very similar to the Bolt DSL.  One extends the StormSpout class, declaring the output fields, and defines the nextTuple method:

```scala
class MySpout extends StormSpout(outputFields = List("word", "author")) {
  def nextTuple = {}
}
```

Spout emit DSL
--------------
The spout emit DSL is very similar to the bolt emit DSL.  Again, all variants of the SpoutOutputCollector emit and emitDirect APIs are supported.  The basic forms for emitting tuples are as follows:

```scala
emit (val1, val2, ...)
emitDirect (taskId, val1, val2, ...)
```

To emit a tuple with a specific message ID:

```scala
using msgId 9876 emit (val1, val2, ...)
using msgId 9876 emitDirect (taskId, val1, val2, ...)
```

To emit a tuple to a specific stream:

```scala
toStream 6 emit (val1, val2, ...)
toStream 6 emitDirect (taskId, val1, val2, ...)
using msgId 9876 toStream 6 emit (val1, val2, ...)
using msgId 9876 toStream 6 emitDirect (taskId, val1, val2, ...)
```