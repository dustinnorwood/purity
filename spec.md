# Data encoding specification
There are general data types in Purity: value types, and pointer types. Value types are fixed in size, whereas pointer types generally hold dynamically-sized data, and use their pointer as a fixed-size reference their contents.
## Fixed size types
Fixed size types have a size, in bytes, known at compile-time. 
- Basic fixed-value types are:
- Bool (1 bytes)
- UInt8 .. UInt256 (1 - 32 bytes)
- Int8 .. Int256 (1 - 32 bytes)
- Address (20 bytes)
- Records
- Tuples
- Dependently-typed data types
By themselves, fixed-size types occupy 32*ceiling(size/32) bytes of contract storage. As part of a record, fixed-size types are tightly packed, to conserve storage.

Example record syntax:
```data MyRecord = MyRecord {
    myBool    :: Bool
  , myUint32  :: UInt32
  , myAddress :: Address
}```
In `MyRecord`, the entries can be tightly packed into one 32-byte word. Even though the sum of `Bool`, `UInt32`, and `Address` is only 25 byte, the record entry reserves an entire 32-byte word for itself.

Example data syntax:
```data Maybe a = Nothing | Just a```
In `Maybe a`, there are two value constructors: `Nothing`, and `Just a`. To represent 2 or more constructors, a 32-byte word is dedicated to representing the constructor number (0 for `Nothing`, 1 for `Just a`, in this case), with the value constructor's data representation taking the necessary subsequent words.
