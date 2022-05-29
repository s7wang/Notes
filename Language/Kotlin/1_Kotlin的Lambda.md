# Kotlin的Lambda强大表现

Kotlin中的几种Lambda表达

```kotlin
fun main(args: Array<String>) {

    val method: (String, String) -> Unit = {aStr, bStr -> 
        println("a:$aStr, b:$bStr")}
    method("xxx", "yyy")

    val method02 = { println("method02")}
    method02()
    method02.invoke()

    var method03: (String) -> Unit = { }
    method03("method03")

    var method04: (Int) -> Unit = {
        when (it) {
            1 -> println("Input is 1")
            in 20 .. 30 -> println("Input is 20 ~ 30")
            else -> println("Input is others")
        }
    }
    method04(30)
    method04(31)

    var method05: (Int, Int) -> Unit = { aNumber: Int, bNumber: Int ->
        println(aNumber+bNumber)
    }
    method05(2, 3)

    var method06: (Int, Int) -> Unit = { aNumber: Int, _: Int ->
        println(aNumber)
    }
    method06(6, 7)

    var method07: (String) -> String = { str -> str }
    println(method07("str result"))
}
```

结果：

```
a:xxx, b:yyy
method02
method02
Input is 20 ~ 30
Input is others
5
6
str result
```

