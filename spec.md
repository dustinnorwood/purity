# Motivation
Smart contracts are state machines. They are comprised of storage (state) and functions (state transitions). Additionally, the EVM provides access to data pertaining to the blockchain.

## Pure functions
Generally, all smart contract functions exist in a ReaderT BlockchainState (StateT ContractStorage m) monad stack. BlockchainState is the global, read-only data available during execution, and ContractStorage is the contract's internal storage, which the contract is allowed to read and write. However, smart contract function need not access storage, nor access the blockchain's state. Such functions are pure, and are typed as such. References to blockchain state or contract storage in pure functions results in a compilation error.

## ReaderT functions
Functions that require read-access to either the blockchain state or the contract's storage must be wrapped in a ReaderT monad, where the reader type is BlockchainState or ContractStorage, respectively.

## StateT functions
Functions that require write-access to the contract's storage must be wrapped in a StateT monad, where the state type is ContractStorage.

## ReaderT BlockchainState (StateT ContractStorage m) functions
Hypothetically, all functions in Purity contracts can be in the ReaderT BlockchainState (StateT ContractStorage m) monad. However, this is not necessary, can introduce execution-time bugs, and use more gas.

## Modifiers
A common pattern in smart contracts is the `onlyOwner` modifier, which prevents anyone except the creator of the contract from executing a function. We can define the `onlyOwner` pattern in Purity by the following:
```
data SampleContractData = { owner :: Address }

type OnlyOwner = State SampleContractData

getOwner :: OnlyOwner Address
getOwner = owner <$> get
  
guardOnlyOwner :: OnlyOwner ()
guardOnlyOwner = do
  sender <- asks msgSender
  owner <- getOwner
  unless (sender == owner) $ error "You're not the owner!"
  
protectedFunction :: a -> OnlyOwner a
protectedFunction a = do
  guardOnlyOwner
  return a
```

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
By themselves, fixed-size types occupy (32 * ceiling(size/32)) bytes of contract storage. As part of a record, fixed-size types are tightly packed, to conserve storage.

Example record syntax:
```
data MyRecord = MyRecord {
    myBool    :: Bool
  , myUint32  :: UInt32
  , myAddress :: Address
}
```
In `MyRecord`, the entries can be tightly packed into one 32-byte word. Even though the sum of `Bool`, `UInt32`, and `Address` is only 25 byte, the record entry reserves an entire 32-byte word for itself.

Example data syntax:
```
data Maybe a = Nothing | Just a
```
In `Maybe a`, there are two value constructors: `Nothing`, and `Just a`. To represent 2 or more constructors, a 32-byte word is dedicated to representing the constructor number (0 for `Nothing`, 1 for `Just a`, in this case), with the value constructor's data representation taking the necessary subsequent words.
