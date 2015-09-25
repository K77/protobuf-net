# Performance #

## Test Conditions ##
The test rig is in the [examples project](http://code.google.com/p/protobuf-net/source/browse/#svn/trunk/Examples). Timings (ms) are measured with `Stopwatch`; serializations are to `Stream.Null`; deserialization are from a `MemoryStream`; each cycle follows a preliminary seriliazation (for JIT) and a forced `GC.Collect`. Size is bytes.

## Encoding Document ##

A first set of numbers, based on the samples in the [encoding document](http://code.google.com/apis/protocolbuffers/docs/encoding.html), running in release mode at the console, with the MS .NET 3.5 (SP1 beta) runtime.

([r145](https://code.google.com/p/protobuf-net/source/detail?r=145); 1,000,000 iterations)

### Test1 ###

|**Serializer**|**size**|**serialize**|**deserialize**|
|:-------------|:-------|:------------|:--------------|
|protobuf-net  |3       |268          |1,881          |
|[proto#](http://code.google.com/p/protosharp/)|3       |76           |1,792          |
|`BinaryFormatter`|153     |6,694        |8,420          |
|`SoapFormatter`|687     |28,609       |55,125         |
|`XmlSerializer`|153     |14,594       |19,819         |
|`DataContractSerializer`|205     |3,263        |10,516         |
|`DataContractJsonSerializer`|26      |2,854        |15,621         |

### Test2 ###

|**Serializer**|**size**|**serialize**|**deserialize**|
|:-------------|:-------|:------------|:--------------|
|protobuf-net  |9       |356          |2,178          |
|[proto#](http://code.google.com/p/protosharp/)|9       |255          |2,139          |
|`BinaryFormatter`|161     |6,438        |8,622          |
|`SoapFormatter`|702     |28,164       |56,211         |
|`XmlSerializer`|157     |14,037       |18,572         |
|`DataContractSerializer`|151     |2,876        |9,387          |
|`DataContractJsonSerializer`|15      |1,919        |11,335         |

### Test3 ###

|**Serializer**|**size**|**serialize**|**deserialize**|
|:-------------|:-------|:------------|:--------------|
|protobuf-net  |5       |842          |3,535          |
|[proto#](http://code.google.com/p/protosharp/)|5       |345          |3,497          |
|`BinaryFormatter`|251     |11,618       |15,178         |
|`SoapFormatter`|928     |47,179       |79,218         |
|`XmlSerializer`|170     |15,373       |22,216         |
|`DataContractSerializer`|212     |3,778        |11,455         |
|`DataContractJsonSerializer`|32      |4,129        |18,728         |

### Conclusion ###

For small fragments, proto is an order of magnitude quicker, but all variants are still extremely fast (on the µs scale). Note that the short length of the type/field names positively skews the size for the MS serializers (i.e. makes them shorter). In general the difference is even larger; protocol buffers don't use names, so aren't impacted.

## Northwind ##

A more realistic performance metric is obtained using a Northwind data-extract via LINQ-to-SQL; in this case we can't present binary formatter numbers (`BinaryFormatter` / `SoapFormatter`) because some of the classes involved (from LINQ-to-SQL) do not support such usage.

This metric based on the data from [nwind.proto.bin](http://code.google.com/p/protobuf-net/source/browse/#svn/trunk/Tools), which has nearly 3000 objects (orders and order-lines).

([r145](https://code.google.com/p/protobuf-net/source/detail?r=145); 1000 iterations; proto# is [r75](http://code.google.com/p/protosharp/source/detail?r=75))

|**Serializer**|**size**|**serialize**|**deserialize**|
|:-------------|:-------|:------------|:--------------|
|protobuf-net (1)|133,010 |5,779        |13,224         |
|protobuf-net (2)|132,263 |4,657        |14,045         |
|[proto#](http://code.google.com/p/protosharp/) (3)|130,111 |4,598        |14,939         |
|`DataContractSerializer`|736,574 |15,100       |71,688         |
|`DataContractJsonSerializer`|490,425 |29,399       |136,421        |

1 = using length-prefixed sub-messages

2 = using grouped sub-messages

3 = using packed encoding for `DateTime`, `Decimal` etc

### Conclusion ###

**needs update (post [r122](https://code.google.com/p/protobuf-net/source/detail?r=122)):** have improved performance for length-prefix [de](de.md)serialization.

The first observation has to be that using groups (rather than length-prefixed) sub-messages provides a noticeable improvement in performance; this is because the length must be written first, and calculating this (which includes all descendents) is almost as expensive as performing the serialization. The system deliberately avoids the need to buffer the entire sub-message binary, since this could be very large and would involve nested buffers for each level, bloating memory usage. Likewise, deserializing a nested message currently involves indirection via a length-limited sub-stream (although this could perhaps be improved).

In contrast; to writing groups is simple: write a starter; write the data; write a terminator - the cost of calculating the length is eliminated.

Both variants provide a performance improvement over `DataContractSerializer`, with deserialization being most noticeable.

### Additional ###

As part of the "remoting" unit tests, I added a clone of the Northwind sample, but without using the LINQ classes. Fortunately `Serializer.ChangeType()` makes it easy to fill this clone from the existing objects! This allows us to benchmark xml/binary serializer performance for the large payloads. For `BinaryFormatter` / `XmlSerializer`, I have also implemented `ISerializable` (via a `byte[]`) / `IXmlSerializable` (via base-64) using protobuf-net for comparison. Note that with the known issues with generics, `SoapFormatter` simply isn't worth the pain of testing for complex data...

Simple compatible classes (using length-prefix for sub-messages):

|**Serializer**|**size**|**serialize**|**deserialize**|
|:-------------|:-------|:------------|:--------------|
|protobuf-net (1)|133,010 |6,001        |12,140         |
|protobuf-net (2)|132,263 |4,750        |11,933         |
|[proto#](http://code.google.com/p/protosharp/) (3)|130,111 |4,844        |13,879         |
|`BinaryFormatter`|276,302 |69,371       |58,617         |
|`XmlSerializer`|1,043,137|29,465       |38,770         |
|`DataContractSerializer`|772,406 |15,159       |67,924         |
|`DataContractJsonSerializer`|490,425 |28,338       |135,935        |

Advanced (`ISerializable` / `IXmlSerializable`) compatible classes (using length-prefix for sub-messages):

|**Serializer**|**size**|**serialize**|**deserialize**|
|:-------------|:-------|:------------|:--------------|
|protobuf-net  |133,010 |6,165        |12,538         |
|`BinaryFormatter`|133,155 |7,504        |12,284         |
|`XmlSerializer`|177,410 |7,599        |19,668         |

This shows that protocol buffers can be used to offer reduced size and increased performance, even with the standard serializers. In particular, note that when coupled with protobuf-net, `XmlSerializer` only needs 1/6th of the space with comparable time, and `BinaryFormatter` halves both space and time.

## Remoting ##

The test project includes remoting examples (`SmallPayload()` and `LargePayload()`), designed to evaluate the performance of using protobuf-net to implement ISerializable, rather than using the native formatter. I don't have exact numbers at the moment, but for **local** tests (i.e. 2 `AppDomain`s in the same process) the performance depends on the payload:

  * For large payloads (I have used the Northwind example, above), protobuf-net halves round-trip time, presumably by improving on the `BinaryFormatter` speed
  * For small payloads, protobuf-net makes things _slower_

It is expected that for **remote** tests the bandwidth will be the biggest factor; as such protocol-buffer usage should show a considerable improvement; figures to follow...