# 1. Foundations

## Elements and Plugins

Elements是Gstreamer的核心，gstreamer开发中，element是继承自 *GstElement* 类的对象。将一系列的 element 连接起来构成pipeline或者bin，来提供更复杂的功能。但是，仅仅开发一个 element 是不够的，需要 *plugin* 来封装element，让gstreamer能够加载它。plugin是能加载的代码块，通常为动态链接库（shared object or dynamic linked library）

## GstMiniObject, Buffers and Events

GStreamer的所有流数据（stream data）分割为 *chunk* 块，从一个element的srcpad传递到另一个element的sinkpad。*GstMiniObject* 是用于保存这些数据块的结构体。*GstMiniObject* 包含如下重要的类型：

* 确切的type指示该GstMiniObject是什么数据类型(event, buffer, ...)

* 引用计数指示当前element引用miniobject的个数，当refcount减到0时，miniobject将会disposed，内存将会释放。

数据传输时，GstMiniObject定义了2种类型：buffers（content）和 events（control）。

**Buffers**可以包含2个连接的pads能够处理的任何类型的数据。通常，buffer包含了一个element流向另一个element的某种音频或视频数据chunk。

Buffers也可以包含metadata来描述buffer的内容，一些重要的metadata类型有：

* 指向一个或多个GstMemory对象的指针，GstMemory对象是封装内存区域的引用对象(refcounted object).

* 指示缓冲区内容的显示时间戳(display timestamp).

Events包含两个连接的pads的流的state信息。Events只有在元素显示支持的时候才会被发送，否则core将（尝试）自动处理事件。Events用于指示，媒体类型、媒体流结束或者应该刷新cache。

Events可能包含如下：

* 指示包含event类型的subtype

* 事件的其他内容取决于具体的事件类型

## Media types and Properties

GStreamer使用类型系统来确保元素之间的数据传输采用可识别的格式。类型系统对于确保元素之间pads连接时正确匹配所需要的参数特定格式同样重要。*在元素之间建立的每个连接都有一个指定的类型和一组可选的属性*。 在 [Caps negotiation](https://gstreamer.freedesktop.org/documentation/plugin-development/advanced/negotiation.html) 中查看有关 caps 协商的更多信息。

### The Basic Types

GStreamer already supports many basic media types. Following is a table of a few of the basic types used for buffers in GStreamer. The table contains the name ("media type") and a description of the type, the properties associated with the type, and the meaning of each property. A full list of supported types is included in List of Defined Types.

GStreamer 已经支持许多基本的媒体类型。 以下是 GStreamer 中用于缓冲区的一些基本类型的表格，包含媒体类型和类型描述、与类型关联的属性以及每个属性的含义。受支持类型的完整列表详见 [List of Defined Types](https://gstreamer.freedesktop.org/documentation/plugin-development/advanced/media-types.html#list-of-defined-types)
