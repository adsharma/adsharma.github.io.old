## FlatBuffer as serialization agnostic IDL

Many popular IDLs mix the following two separate concerns into one leading to less than optimal results.

* Language agnostic type system supporting enums, types and variants/unions.
* RPC oriented wire format with backward compatibility as a first class concern

Mixing these two concerns has many downsides. The first one is the weakening of the type system.
Suppose we have a union containing two things:

```
union {
    Cat,
    Dog,
}
```

The generated code from the IDL compiler may add a third state - "not defined" or similar to deal with the
case that the RPC was sent from an old client. This concern now permeates code at other layers in the stack
where one doesn't necessarily need to worry about this third state. This could happen for example if we fail
the request at the lower levels of the stack and deal with only two possibilities in the application logic.

Secondly, scalars and types may have different syntax in how optionality is expressed. My understanding is that
this is also driven by the serialization concern in the flatbuffer IDL.

## What about serialziation then?

Yes, someone has to do the work for serialization. A good way forward is to do it in a language specific library.
Rust has `serde` and Python has pure-protobuf.

```
@message
@dataclass
class Cat:
    name: str
    age: int
```

## Example IDL

This work separates these two concerns as described above. It adopts the flatbuffer IDL (since it doesn't have the
field numbering concern in the IDL, significantly simplyfying the language). Attributes have been used to support 
high level type safe concepts such as Protocols and Views (a concept similar to graphql fragments).

Here is a flatbuffer derived IDL file.

https://github.com/adsharma/flattools/blob/705998739bc69783ffa637b5f37d305d0d671b90/tests/parser-cases/color.fbs#L1-L33

Running it through the generator results in the following Rust code:

https://github.com/adsharma/flattools/blob/705998739bc69783ffa637b5f37d305d0d671b90/tests/expected/golden-color.rs#L1-L47

Similar output exists for Kotlin, Swift and Python3 + dataclasses in the same directory.

### Backward Compatibility

Using the `deprecated` attribute is recommended.

### Future Improvements

* Expand the set of supported languages
* Add decorators for serialization
