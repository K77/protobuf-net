This page is an archive of early changes only, and is not maintained. Please see the [Nuget Package](http://nuget.org/packages/protobuf-net/) release notes for more recent changes.


# Downloads #

are avaialable via [Downloads](http://code.google.com/p/protobuf-net/downloads/list).

# History #

== 28 Jan 2010; [r281](https://code.google.com/p/protobuf-net/source/detail?r=281)

  * string interning
  * CF nongeneric bugfix
  * allow duplicated enums (different name, same wire-value **and** enum-value)
  * added memcached transcoder

== 25 Nov 2009; [r276](https://code.google.com/p/protobuf-net/source/detail?r=276)

  * uri encoding fix
  * support IDictionary<,>
  * optimised constructor calls

== 09 Oct 2009: [r274](https://code.google.com/p/protobuf-net/source/detail?r=274)

  * flags enums support; ushort support
  * better custom attribute support
  * WCF endpoint configuration support

## 16 Jul 2009: [r262](https://code.google.com/p/protobuf-net/source/detail?r=262) ##

  * Visual Studio support (early)

## 01 Jul 2009: [r261](https://code.google.com/p/protobuf-net/source/detail?r=261) ##

  * BREAKING CHANGE: protogen using wrong type for fixed-length ints
  * VB support
  * Automatic case handling in .proto ($fixCase - C# only at the moment)
  * Namespace fixes
  * BUGFIX: assembly resolution from attribute
  * Allow unknown enums to carry through without raising an exception; stored in the
> extension object if supported, else drops

## 02 Jun 2009: [r255](https://code.google.com/p/protobuf-net/source/detail?r=255) ##

  * Packed encoding support for lists, arrays, etc
  * Support for packed and deprecated from .proto
  * Updated protoc 2.1

## 22 May 2009 ##

  * Improved support for namespaces when using multiple .proto files;
    * added -ns:{ns} to specify a default namespace (used when there is no package)
    * added -p:import={imports} to specify namespaces to import ("using")
    * fixed bug with default namespace

## 02 Apr 2009 ##

  * Allow data-member offset in Silverlight
  * Fix bug deserializing nested list/enumerables
  * More RPC prep
  * ProtoGen now uses exe path correctly

## 26 Mar 2009 ##

  * Early (not final) draft of working RPC, including C# 3.0 extension / lambda usage.
  * Beginnings of major refactor to split the model from attributes
  * Added support for multiple services on http endpoint

## 02 Mar 2009 ##

  * Strong naming of CF20/CF35/Mono

## 19 Feb 2009 ##

  * Better support for **Specified; non-public; get-only; instance**

## 16 Feb 2009 ##

  * Use Reflection.Emit to access fields (rather than reflection) when available
  * Introduce callback interface (not yet integrated)
  * Add TryReadLengthPrefix to public API (for async network usage)

## 13 Feb 2009: [r230](https://code.google.com/p/protobuf-net/source/detail?r=230) ##

  * Support {name}Specified pattern
  * xslt changes for above (includes ComponentModel etc); general xslt tidy
  * Better trace info when deserialization explodes (in exception data)
  * Complain if [ProtoInclude](ProtoInclude.md) conflicts with an existing tag

## 09 Feb 2009: [r225](https://code.google.com/p/protobuf-net/source/detail?r=225) ##

  * Fixed inheritance issue when a property (or list) is defined a **subtype** of a contract; [de](de.md)serializes starting
> > from the highest contract ancestor (previously started from member type)
  * Consider abstract types as contracts (allows for abstract contract ancestors for above change)
  * Fixed inheritance issue when [de](de.md)serializing via Serializer
  * Removed [ProtoContract](ProtoContract.md) from Extensible - breaks derived classes

## 10 Dec 2008: [r219](https://code.google.com/p/protobuf-net/source/detail?r=219) ##

  * Allow offset via attribute for WCF (data-member) usage; allows compatibility between WCF client/LINQ server
  * Thread safety of Build() method (per-type lock, with zero-lock short-circuit once built)
  * Workaround for mono compiler (generics) bug; verifies constraint at runtime rather than compile
  * Integrated mono/nant into build script and zip

## 25 Nov 2008: [r214](https://code.google.com/p/protobuf-net/source/detail?r=214) ##

  * ProtoGen: simpler field names; default unspecified field-type to double (not int)
  * Fixed private class issue on CF 2.0

## 21 Nov 2008: [r212](https://code.google.com/p/protobuf-net/source/detail?r=212) ##

  * Implicit fields skips delegates (including event-backers) and [NonSerialized](NonSerialized.md)
  * Detect CF 2.0 non-public runtime bug and offer advice

## 20 Nov 2008: [r211](https://code.google.com/p/protobuf-net/source/detail?r=211) ##

  * Allow (optional) implicit serialization of fields (like BinaryFormatter) or the entire public API
  * Make QuickStart customer/contact public for CF 2.0 security glitch
  * Bugfix with inheritance (affected all except MemoryStream)

## 11 Nov 2008: [r202](https://code.google.com/p/protobuf-net/source/detail?r=202) ##

  * Minor buffer / encoding optimisations
  * .proto parser (ProtoGen)
  * Support dictionary / keyvaluepair
  * Buildkit
  * Start of RPC stack

## 13 Oct 2008: [r177](https://code.google.com/p/protobuf-net/source/detail?r=177) ##

  * Allow creation via non-public constructors, and allow Merge on objects without any parameterless constructor
  * Clarified error message for (failed) creation without a parameterless constructor
  * Support fields in addition to properties
  * Add quotes to string-based defaults (waiting on escape clarification)

## 06 Oct 2008: [r172](https://code.google.com/p/protobuf-net/source/detail?r=172) ##

  * Allow  [negative enum values](http://groups.google.com/group/protobuf/browse_thread/thread/8f6e9ba3ee923bdb)
  * Changed GetPassThru to side-step Mono compiler issue (note compiler bug fixed in mono 2.0)
  * QuickStart sample begun; SubStream re-introduced for NetworkStream sample
> > "WithLengthPrefix" methods added to assist with sockets-based usage

## 18 Sep 2008: [r164](https://code.google.com/p/protobuf-net/source/detail?r=164) ##

  * DateTime/TimeSpan support for tick-level precision and/or Fixed64 encoding
  * Moved WCF unit tests to non-default port (false positive failures due to conflicts)
  * Fixed Int64 encoding bug (whoops!)
  * Allow tag inference (comparable to WCF behaviour, opt-in only)
  * Removed class/new constraints for [most](most.md) Serializer methods
  * Support direct serialization of `IList<T>`/`T[]`/`IEnumerable<T>`+`Add(T)` objects
  * Added global default for InferTagFromName
  * Enabled WCF serialization of list/array
  * Primatives / string / etc can now be serialized directly (without an entity)

## 20 Aug 2008: [r153](https://code.google.com/p/protobuf-net/source/detail?r=153) ##

Oops - a bit of a change gap...

  * Refactored property implementation for performance
  * Support for multiple readers for a single property
  * Various .proto emitter fixes
  * Support for DateTime etc as "bcl" messages
  * Uses optimistic length guess (rather than full length calculation) for sub-messages
  * Support for full streaming (IEnumerable

&lt;T&gt;

/Add rather than IList

&lt;T&gt;

)
  * Re-worked array/list to peek for next item (replaces naïve from [r90](https://code.google.com/p/protobuf-net/source/detail?r=90))
  * Multi-byte utf8 fixes
  * Various other fixes

## 24 Jul 2008: [r90](https://code.google.com/p/protobuf-net/source/detail?r=90) ##

  * Performance; "workspace" buffer now always 0-based; uses lightweight recursion check up to threshold; length prediction streamlined; encode now writes to stream (not workspace)
  * Support for single-dimension arrays (naïve implementation)
  * Support for Compact Framework 2.0 & 3.5
  * Support for extension fields encoded as groups (includes dropping data for inextensible objects)
  * Fixed handling of type-init / dynamic-invoke exceptions
  * Code tidied as per code analysis / source analysis

## 23 Jul 2008: [r72](https://code.google.com/p/protobuf-net/source/detail?r=72) ##

  * Support for deserializing known sub-object fields encoded using groups (data is serialized as length-prefixed; not supported for extension fields)
  * Support for [extension](Extensions.md) fields
  * Recursion detection when serializing


## 22 Jul 2008: [r53](https://code.google.com/p/protobuf-net/source/detail?r=53) ##

  * Optimised field lookup during deserialization
  * Wcf example
  * Silverlight example
  * Support for enums
  * Compatibility tweaks (C# 2 / mono / .NET 2.0 / VS2005 / Sliverlight)