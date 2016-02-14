title: lazy initialize in .net
date: 2016-02-14 17:49:51
categories:
- coding
tags:
- .net
---

Lazy initialization of an object means that its creation is deferred until it is first used.
|Type              |     Description
|   ---------------|-----------------
| Lazy<T>          |    A wrapper class that provides lazy initialization semantics for any class library or user-defined type.
| ThreadLocal<T>   |    Resembles Lazy<T> except that it provides lazy initialization semantics on a  thread-local basis. Every thread has access to its own unique value.
| LazyInitializer  |    Provides advanced static (Shared in Visual Basic) methods for lazy initialization of objects without the overhead of a class.

```csharp
   Lazy<Orders> _orders = new Lazy<Orders>(() => new Orders(100));
```

```csharp
   ThreadLocal<int> betterCounter = new ThreadLocal<int>(() => 1);
```
```csharp
// Assume that _orders contains null values, and
// we only need to initialize them if displayOrderInfo is true
if(displayOrderInfo == true)
{
 for (int i = 0; i < _orders.Length; i++)
 {
     // Lazily initialize the orders without wrapping them in a Lazy<T>
     LazyInitializer.EnsureInitialized(ref _orders[i], () =>
         {
             // Returns the value that will be placed in the ref parameter.
             return GetOrderForIndex(i);
         });
 }
}
```
