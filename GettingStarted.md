# Getting Started #

protobuf-net is a **contract based** serializer for .NET code, that happens to write data in the "protocol buffers" serialization format engineered by Google. The API, however, is very different to Google's, and follows typical .NET patterns (it is broadly comparable, in usage, to `XmlSerializer`, `DataContractSerializer`, etc). It should work for most .NET languages that write standard types and can use attributes.

## Hello World ##

The simplest way to get started is simply to write your data; for example (I'll use C# for most examples; sorry about that):

```
    class Person {
        public int Id {get;set;}
        public string Name {get;set:}
        public Address Address {get;set;}
    }
    class Address {
        public string Line1 {get;set;}
        public string Line2 {get;set;}
    }
```

That is a good start, but **by itself** is not enough for protobuf-net. Unlike `XmlSerializer`, the member-names are not encoded in the data - instead, you must pick an integer to identify each member. Additionally, to show intent it is necessary to **show** that we intend this type to be serialized (i.e. that it is a data contract):

```
    [ProtoContract]
    class Person {
        [ProtoMember(1)]
        public int Id {get;set;}
        [ProtoMember(2)]
        public string Name {get;set:}
        [ProtoMember(3)]
        public Address Address {get;set;}
    }
    [ProtoContract]
    class Address {
        [ProtoMember(1)]
        public string Line1 {get;set;}
        [ProtoMember(2)]
        public string Line2 {get;set;}
    }
```

## Notes for Identifiers ##

  * they must be **positive** integers
  * they must be unique within a single type
    * but the same numbers can be re-used in sub-types if inheritance is enabled
  * the identifiers must not conflict with any inheritance identifiers (discussed later)
  * lower numbers take less space - don't start 100,000,000
  * the identifier is important; you can change the member-name, or shift it between a property and a field, but changing the identifier changes the data

## Notes on types ##

supported:

  * custom classes that:
    * are marked as data-contract
    * have a parameterless constructor
    * for Silverlight: are public
  * many common primitives etc
  * _single_ dimension arrays: `T[]`
  * `List<T>` / `IList<T>`
  * `Dictionary<TKey,TValue>` / `IDictionary<TKey,TValue>`
  * any type which implements `IEnumerable<T>` and has an `Add(T)` method

The code assumes that types will be mutable around the elected members. Accordingly, custom structs are not supported, since they should be immutable.

## Serializing Data ##

Since "protocol buffers" is a binary format, protobuf-net is based heavily around the Stream class; this makes it possible to use with a wide variety of implementations simply. For example, to write to a file:

```
    var person = new Person {
        Id = 12345, Name = "Fred",
        Address = new Address {
            Line1 = "Flat 1",
            Line2 = "The Meadows"
        }
    };
    using (var file = File.Create("person.bin")) {
        Serializer.Serialize(file, person);
    }
```

This writes a 32 byte file to "person.bin". It might not be obvious in the above, but `Serialize` is a **generic** method - the line could also be:

```
    using (var file = File.Create("person.bin")) {
        Serializer.Serialize<Person>(file, person);
    }
```

But most of the time we can let the compiler's generic type inference do the work for us.

## Deserializing Data ##

We also need to get out data back!

```
    Person newPerson;
    using (var file = File.OpenRead("person.bin")) {
        newPerson = Serializer.Deserialize<Person>(file);
    }
```

This reads the data back from "person.bin". Note we need to tell it the type this time (the `<Person>`), but otherwise the code is very similar.

## Deployment ##

You must deploy the protobuf-net dll alongside your project. Make sure you deploy the version appropriate to your environment.

## Inheritance ##

Inheritance must be explicitly declared, in a similar way that if must for `XmlSerializer` and `DataContractSerializer`. This is done via `[ProtoInclude(...)]` on each type with known sub-types:

```
    [ProtoContract]
    [ProtoInclude(7, typeof(SomeDerivedType)]
    class SomeBaseType {...}

    [ProtoContract]
    class SomeDerivedType {...}
```

There is no special significance in the `7` above; it is an integer key, just like every `[ProtoMember(...)]`. It must be unique in terms of `SomeBaseType` (no other `[ProtoInclude(...)]` or `[ProtoMember(...)]` in `SomeBaseType` can use `7`), but does not need to be unique globally.

## Don't Like Attributes? ##

In v2, everything that can be done with attributes can also be configured at runtime via `RuntimeTypeModel`. The `Serializer.*` methods are basically just shortcuts to `RuntimeTypeModel.Default.*`, so to manipulate the behaviour of `Serializer.*`, you must configure `RuntimeTypeModel.Default`.