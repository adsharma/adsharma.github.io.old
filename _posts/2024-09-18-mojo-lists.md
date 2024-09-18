## Mojo Lists vs Python Lists

Mojo seeks to provide a familar programming model to python programmers, while allowing them the option of using more performant low level constructs. The stdlib is similar to python stdlib, but not exactly the same.

### Iterating over lists

Python
```
a: List[int] = [1, 2, 3, 4]
for e in a:
    print(a)
```

Mojo
```
var a: List[Int] = List(1, 2, 3, 4)
for e in a:
    print(e)
```

Fails with hard to understand type errors on `print()`.

So the programmer is forced to write:

```
var a: List[Int] = List(1, 2, 3, 4)
for i in range(a.size):
    var e = a[i]
    print(e)
```

## Comparing Lists

Python

```
a = [1, 2, 3]
b = [4, 5, 6]
c = [4, 5, 6]
assert a != b
assert b == c
```

Mojo

```
import testing

var a: List[Int] = List(1, 2, 3)
var b: List[Int] = List(4, 5, 6)
var c: List[Int] = List(4, 5, 6)

testing.assert_not_equal(a, b)
testing.assert_equal(b, c)
```

Fails with `List` doesn't implement `__eq__` or `__ne__`.

## Summary

While mojo makes a good faith attempt to be approachable to python programmers, it has gaps in the API and error messages can be hard to decipher for the average python programmer.
