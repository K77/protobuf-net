# Use "Protocol Buffers" serialization from your .NET code #

## Introduction ##

**protocol buffers** is the name of the binary serialization format used by Google for much of their data communications. It is designed to be:

**small in size - efficient data storage (far smaller than xml)** cheap to process - both at the client and server
**platform independent - portable between different programming architectures** extensible - to add new data to old messages

**protobuf-net** is a .NET implementation of this, allowing you to serialize your .NET objects efficiently and easily. It is compatible with most of the .NET family, including .NET 2.0/3.0/3.5, .NET CF 2.0/3.5, Mono 2.x, Silverlight 2, etc.

## OMG! Where did the code go!! ##

If you're watching the trunk, don't panic! I've finally made some progress on v2 (the rewrite without generics, to better support CF, 1.1, etc). Oh, and it also now includes full and partial pre-compilation. It is **very** early. For production use, use the [v1 branch](http://code.google.com/p/protobuf-net/source/browse/#svn/branches/v1).

# Caveat for Compact Framework #

I'm aware of a [problem with CF](http://marcgravell.blogspot.com/2009/03/compact-framework-woes-revisted.html) that can cause a `MissingMethodException`. Work is in progress to fix this, but this is going to take some time (it is a **big** refactor).

# Work in Progress #

  * Big refactor - to workaround Compact Framework generics limitation

# Next on the List #

  * RPC/http improvements
  * RPC/mvc implementation
  * RPC/socket implementation

# New 16 Jul 2009 #

  * Now with [Visual Studio 2008 support](http://marcgravell.blogspot.com/2009/07/protobuf-net-now-with-added-orcas.html)

http://lh6.ggpht.com/_feJtUp7IGuI/Sl7rs9FkdHI/AAAAAAAAAJY/zoDaMfuthIw/proto-vs_thumb%5B1%5D.png?imgmax=800

# New 03 Jun 2009 #

  * Now with added VB! The trunk now includes vb.xslt, allowing code-generation directly into VB; thanks to Michael Thaler and Rüdiger Klaehn for contributing this.

# New 02 Jun 2009 #

  * Support for the new "packed" encoding type for arrays/lists/etc of simple types; use `Options` on the `[ProtoMember]` to enable this
  * Updated to use protoc 2.1
  * Improved control of namespaces when using protogen; see `-ns` and `-t:import` for more details

# New 02 Apr 2009 #

  * prototype RPC/http stack now available in the build; note that the RPC features are in flux; if you use them, you should be willing to upgrade client and server **at the same time** when you next update (I cannot guarantee that the "final" RPC details will be the same)
  * ProtoGen fix for path resolution

# New 26 Mar 2009 #

  * First cut of client/server RPC stack over http (source only; will be released in binary after the [big refactor](http://marcgravell.blogspot.com/2009/03/compact-framework-woes-revisted.html))

# Newish #

  * Callbacks - you can mark methods (in the contract root) with standard `[OnSerialized]` etc attributes to inject code into the serialization pipeline (although you cannot take over the binary output)
  * Field access now optimized (property access has been optimized for a long time)
  * protobuf-net now has fully implemented inheritance support (meaning: I teased out the last few gremlins).
  * protobuf-net now includes a code-generator, "protogen". For example `protogen -i:person.proto -o:person.cs`
> Protogen will create a C# class file that represents your entities (the .proto is not used by protobuf-net after this). For full usage details, use `/?` at the command line.

## General ##

For details about Protocol Buffers, see [here](http://code.google.com/p/protobuf/).

Unlike the C++/Java .proto compilers and immutable classes, this implementation is based around regular (mutable) .NET classes, attributed as-per WCF data-contracts. This allows existing frameworks (including WCF and LINQ-to-SQL) to use protocol-buffer classes without modification. You may also wish to know about the [C# port](http://github.com/jskeet/dotnet-protobufs/tree/master) of the Java version, led by Jon Skeet, and [Proto#](http://code.google.com/p/protosharp/) (which uses a similar philosophy to protobuf-net), led by Torbjörn Gyllebring.

The code is written in C#, but should be usable from any .NET language.

This approach also allows for very simple code generation, as the real code is in the serializer; there is no entity code, and no "builder" classes - just the data contracts.

Where service contracts are included, the default approach is to use WCF (since this is already based on service-contracts), but the concept should work with any alternative RPC channel, such as remoting.

# Example #

To take the example from the encoding document, here is the fully working .NET representation:

```
    [DataContract]
    public sealed class Test1
    {
        [DataMember(Name="a", Order=1, IsRequired=true)]
        public int A { get; set; }
    }
    [DataContract]
    public sealed class Test2
    {
        [DataMember(Name = "b", Order = 2, IsRequired = true)]
        public string B { get; set; }
    }
    [DataContract]
    public sealed class Test3
    {
        [DataMember(Name = "c", Order = 3, IsRequired = true)]
        public Test1 C { get; set; }
    }
```

If you are familiar with WCF data contracts, that will be immediately familiar. Service contracts are defined similarly, but note that we need to add a "behavior" attribute for WCF to use the new serializer (this may be configurable entirely via configuration at a later date):

```
    [ServiceContract]
    public interface IFoo
    {
        [OperationContract, ProtoBehavior]
        Test3 Bar(Test1 value);
    }
```


Note that `[DataContract]` / `[DataMember]` are not the only choices; if you don't have data-contracts (for example, MS .NET 2.0, or [mono](http://www.mono-project.com/Main_Page)), then you can use other attributes; the bespoke `[ProtoContract]` / `[ProtoMember]` (which also allow more fine-grained control over the serialization), and `[XmlType]` / `[XmlElement]`. The most important point is that we must be able to identify a unique integer tag for each serializable property.

But hopefully by supporting WCF data-contracts we should be able to cover a wide range of existing systems.

There is also support for regular .NET binary serialization (`ISerializable`), and full support for object streaming/pipe-lining via `IEnumerable<T>`.

## Performance ##

It is pretty quick ;-p
A full discussion is [here](Performance.md).

# Compatibility #

## .NET 3.0 ##
protobuf-net is designed with .NET 3.0 in mind, as protocol buffers are a natural fit with WCF's data-contracts. This also provides an immediate RPC channel (via `[ProtoBehavior]`) while we wait for the "proto-RPC" (insert actual name here...) conversations to bottom-out. Perhaps as importantly - it allows you to use protobuf-net with your existing WCF/LINQ-to-SQL objects.

## .NET 2.0 ##
The projects have a "Net20" VS configuration, which removes all .NET 3.0 code/references  for a pure .NET 2.0 build (using the custom NET\_3\_0 compilation symbol; the test project also uses NET\_3\_5 for JSON etc). The test rig is not fully .NET 2.0 compatible, but the main code project builds cleanly in .NET 2.0.

Note that `[DataContract]`/`[DataMember]` will not be available, so just use `[ProtoContract]`/`[ProtoMember]` instead.

## Mono ##
(credit: M. David Peterson)

protobuf-net works fully with mono, with a NAnt script is available [here](http://nuxleus.googlecode.com/svn/trunk/nuxleus/Source/CodeSamples/Protobuf_Serialization_Comparison_Test/Protobuf_Serialization_Comparison_Test.build)

so all you should need to do is (watch for wrap):

`$ svn co http://nuxleus.googlecode.com/svn/trunk/nuxleus/Source/CodeSamples/Protobuf_Serialization_Comparison_Test/`

`$ nant`

## C# 2.0 ##
The main code uses only C# 2 language features, so can be compiled in a v2 compiler, including VS2005 and MSBuild from the MS .NET 2.0 distribution.

## Silverlight 2 ##
In progress - again, thanks go to M. David Peterson for help with validation.

There is a protobuf-net\_Silverlight library project in the repo, which builds protobuf-net as a Silverlight assembly. There is also an example (validation) Silverlight application.

The only code differences is that the ISerializable and ServiceModel support is removed; the latter is most vexing, as it will make it hard to inject .proto directly into WCF (which is possible in the full-fat framework). Maybe Silverlight is a candidate for a dedicated http-RPC protocol (already under discussions, but early days).

## Compact Framework 2.0 / 3.5 ##
Tested with CF 2.0 & 3.5 via emulator (Windows Mobile 5.0 Pocket PC SDK).

One note here is that `Delegate.CreateDelegate` is not available under CF 2.0, so performance won't be quite as good as with the other frameworks (but it works). This is re-instated in CF 3.5.

# Author #
The .proto serialization format is a credit to Google's ingenuity.

This specific .NET implementation is designed and written by [Marc Gravell](mailto:marc.gravell@gmail.com), Lead Programmer for [RM](http://www.rm.com/)'s internal Information Systems: Development department.

Credit for various compatibility issues (mono/Silverlight) goes to M. David Peterson.

Visual Basic stylesheet contributed by Michael Thaler and Rüdiger Klaehn.

Initial code for the Visual Studio "Custom Tool" by Shaun Cooley.

Thanks also to Michael Giagnocavo, for numerous good ideas from real usage.