# Introduction #

By default, protobuf-net uses attributes for configuration. Note that in v2 everything can _also_ be configured at runtime without needing to edit any types.

## Contract types ##

Protobuf-net expects contract types to be explicitly marked as such; types can be marked `[ProtoContract]`, `[DataContract]` or `[XmlType]` for this purpose (`[Serializable]` is not required and is not used). `[ProtoContract]`, as the library-specific attribute, allows more control:

  * `SkipConstructor` - suppresses the type's constructor during deserialization
  * `UseProtomembersOnly` - in a type marked with multiple attributes, ignore any members not explicity marked with `[ProtoMember]`
  * `IgnoreListHandling` - even if the type looks like a list (`IEnumerable[<T>]` and `Add()`), use "instance" rules, not "list" rules

(and some more advanced properties relating to automatically inferring contracts)

## Members ##

Protobuf-net needs to be able to associate data with unique / reliable integer keys, since this is how the core protobuf spec (as defined by Google) identifies data. The minimum required is a key, so any of `[ProtoMember]`, `[DataMember]` or `[XmlElement]` can work (in the latter two cases, an `Order` **must** also be supplied).

If you can't edit the member code directly (for example generated code), you can also use `[ProtoPartialMember]` **at the type level** to tell it about the members to include.

Individual members can also be marked with `[DefaultValue]`, which is one of several ways of controlling conditional serialization.

`[ProtoMember]` has a range of specific settings for advanced functionality:

  * `DataFormat` - request a [specific format](https://developers.google.com/protocol-buffers/docs/proto#scalar) to be used when possible, for people familiar with the protobuf specification; common uses:
    * indicate that an `int` should use fixed-width or zig-zag encoding
    * serialize a nested object using "groups" rather than "nested" encoding; google prefer "nested" (see "groups", [here](https://developers.google.com/protocol-buffers/docs/proto#nested)), but "groups" can be more efficient, especially for large sub-objects
  * `IsRequired` - if true, `[DefaultValue]` is ignored; in future releases this may also be used to support full enforcement checks at the point of serialization/deserialization
  * `IsPacked` - packed encoding is a different layout that is especially efficient for arrays/lists of primitives like `int` - see "packed", [here](https://developers.google.com/protocol-buffers/docs/proto#options)
  * `OverwriteList` - protobuf is _defined_ as an appendable format; as such, lists are not wiped when deserializing. If your constructor populates a list/array by default, setting this to `true` will force protobuf-net to ignore the existing values; this does make it non-appendable, though. Other alternative options include:
    * suppressing the constructor
    * using a before-deserialization callback to clear the list
  * `AsReference` - enables reference tracking of the sub-object; this uses a **very different** serialization layout, but will only serialize that object **once** (giving it a unique token for referencing). This allows full-graph support for otherwise cyclic graphs. This can also be used to great effect to efficiently store common strings that are otherwise repeated many times in the data
  * `DynamicType` - stores additional `Type` information with the type (by default it includes the `AssemblyQualifiedName`, although this can be controlled by the user). This makes it possible to serialize weak models, i.e. where `object` is used for property members, however currently this is limited to **contract** types (not primitives), and does not work for types with inheritance (these limitations may be removed at a later time). Like with `AsReference`, this uses a very different layout format


## Inheritance ##

Protobuf-net needs to know about inheritance in advance, via `[ProtoInclude]`. Since they lack the necessary "key" information, `[XmlInclude]` and `[KnownType]` cannot work here.

## Enums ##

Enums _can_ be marked with `[ProtoEnum]`, but this is not required, unless you specifically need to override the values used on the wire.

## Callbacks ##

Protobuf-net supports the usual callback APIs, including `[OnSerializing]`, `[OnSerialized]`, `[OnDeserializing]`, `[OnDeserialized]`, `[ProtoBeforeSerialization]`, `[ProtoBeforeDeserialization]`, `[ProtoAfterSerialization]`, `[ProtoAfterDserialization]`.

## Why duplicate? ##

Not all frameworks have all attributes, so protobuf-net supplies a set that can be used anywhere, but also _tries_ to let you use your existing attributes where possible. Unless you are using the additional settings on the protobuf-net specific attributes, they work identically - for example, a type marked using `[ProtoContract]` and `[ProtoMember]` (without custom settings) will opeate identically to a type marked using `[DataContract]` and `[DataMember]`.