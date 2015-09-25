The following is not complete, but is some of the key features available with protobuf-net:

  * fast binary serialization
  * conforms to the "protobuf" specification (with additional features available optionally)
  * highly version tolerant
  * attribute-based configuration, or explicit configuration with parallel and independent models
  * fully thread-safe, so a single model can be used efficiently
  * optimized IL generation with internal caching
  * static serialization dll generation if required
  * serialization callback support
  * automatic string re-use when deserializing
  * optional string re-use when serializing (changes the layout)
  * tree-based serialization with cycle detection (full-graph optional, see later)
  * class, struct, and interface support (note: interfaces cannot be used for the "root" type)
  * fully appendable format (for building long lists of streaming data)
  * streamable deserialization for consuming an endless stream (sockets etc)
  * support for lists and individual items
  * conditional serialization (based on default values or on user-code)
  * support for parameterless constructors, zero-constructors, or user-supplied factory methods for type construction
  * inheritance support, including abstract type support and inheritance of inter-related generic types
  * support for all standard BCL types
  * optional reference-tracking for full-graph support
  * optional support for embedding limited type information, i.e. `object`
  * direct WCF hooks for "full" .NET to .NET (this does not apply to SL/WP7 etc)
  * can be used to implement `ISerializable` to streamline remoting
  * optional automatic member selection (contract inference)
  * optional "contract first" schema language (".proto") can be used for type generation, but is not required
  * support for common "proxy" types (EF, NHibernate, etc)

I have certainly missed a great many features here. If there are specific features you are interested in, or want an example for, let me know.