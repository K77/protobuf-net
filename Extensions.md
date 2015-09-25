# NEEDS REDOIntroduction #

The .proto architecture allows you to use [extensions](http://code.google.com/apis/protocolbuffers/docs/proto.html#extensions) so that fields can be added to a message without anything breaking; this document discusses use of extensions in protobuf-net.


# Details #

Objects in protobuf-net are not extensible by default, since they are just regular .NET classes; this means that any unexpected fields will be silently dropped during deserialization, and will be lost during serialization. It is, however, trivial to support extensions.

The key to extension support is the [IExtensible](http://code.google.com/p/protobuf-net/source/browse/trunk/protobuf-net/IExtensible.cs) interface. This provides hooks to store and retrieve additional binary data against an instance. There are three ways of implementing this interface.

## Inherit from Extensible ##

The simplest way to make a .NET type extensible is to derive from [Extensible](http://code.google.com/p/protobuf-net/source/browse/trunk/protobuf-net/Extensible.cs):

```
    [ProtoContract]
    public class SimpleExtensible : Extensible
    {
        [ProtoMember(1)]
        public int KnownField {get;set;}
    }
```

This has the advantage of simplicity, but ties you into a specific object hierarchy.

## Simple Buffer ##

The next approach is to define a `byte[]` field in your object to store the extra data, and simply call the static `Extensible` methods for implementation:
```
    [ProtoContract]
    public class StandaloneExtensible : IExtensible
    {
        [ProtoMember(1)]
        public int KnownField { get; set; }

        private byte[] buffer;
        Stream IExtensible.BeginAppend() {
            return Extensible.BeginAppend();
        }
        Stream IExtensible.BeginQuery() {
            return Extensible.BeginQuery(buffer);
        }
        void IExtensible.EndAppend(Stream stream, bool commit) {
            buffer = Extensible.EndAppend(buffer, stream, commit);
        }
        void IExtensible.EndQuery(Stream stream) {
            Extensible.EndQuery(stream);
        }
        int IExtensible.GetLength() {
            return Extensible.GetLength(buffer);
        }
    }
```

This places no inheritance constraints on your objects, but is still simple to implement.

## Custom Implementations ##

The final option is to provide a bespoke `IExtensible` implementation, perhaps reading/writing to a file-system etc. Implementation notes are provided in the [interface documentation](http://code.google.com/p/protobuf-net/source/browse/trunk/protobuf-net/IExtensible.cs).

# Accessing Extension Fields #

You can access data at runtime via the static `Extension` methods (noting that you need to consider whether values are singleton or repeated):

```
    // store a float value as tag 3
    Extensible.AppendValue<float>(obj, 3, 123.45F);

    // retrieve the value or the type's default if not found [singleton]
    float value = Extensible.GetValue<float>(obj, 3);

    // retrieve the value, indicating whether it was found [singleton]
    bool found = Extensible.TryGetValue<float>(obj, 3, out value);


    // store two strings in tag 4
    Extensible.AppendValue<string>(obj, 4, "foo");
    Extensible.AppendValue<string>(obj, 4, "bar");

    // read any stored values [repeated]
    foreach(string s in Extensible.GetValues<string>(obj, 4))
    {
        Console.WriteLine(s);
    }
```

## Data Formats ##

All of the above methods have overloads to specify the [data-format](DataFormat.md) to use (zigzag vs fixed-length, etc). The wire-type will be selected automatically based on the type and data-format, but must be compatible for all stored values (even if a value stored in a different data-format has been superceded).

## Extension Tags ##

At present, the tags used for extension data must be separate to the known tags defined for the message. In this case, use the property accessor instead (support for reading known tags through `Extensible` may be added later).