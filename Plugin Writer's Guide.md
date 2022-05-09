# 1. Foundations

## Elements and Plugins

Elements是Gstreamer的核心，gstreamer开发中，element是继承自 *GstElement* 类的对象。将一系列的 element 连接起来构成pipeline或者bin，来提供功能。但是，仅仅开发一个 element 是不够的，需要 *plugin* 来封装element，让gstreamer能够加载它。plugin是能加载的代码块，通常为动态链接库（shared object or dynamic linked library）

## GstMiniObject, Buffers and Events

GStreamer的所有流数据（stream data）分割为 *chunk* 块，从一个element的srcpad传递到另一个element的sinkpad。*GstMiniObject* 是用于保存这些数据块的结构体。*GstMiniObject* 包含如下重要的类型：

* 确切的type指示该GstMiniObject是什么数据类型(event, buffer, ...)

* 引用计数指示当前element引用miniboject的个数，当refcount减到0时，miniobject将会disposed，内存将会释放。

数据传输时，GstMiniObject定义了2种类型：events（control）和buffers（content）。

**Buffers**可以包含2个连接的pads能够处理的任何类型的数据。通常，buffer包含了一个element流向另一个element的某种音频或视频数据chunk。

Buffers也可以包含metadata来描述buffer的内容，一些重要的metadata类型有：

* Pointers to one or more GstMemory objects. GstMemory objects are refcounted objects that encapsulate a region of memory.

* A timestamp indicating the preferred display timestamp of the content in the buffer.

Events contain information on the state of the stream flowing between the two linked pads. Events will only be sent if the element explicitly supports them, else the core will (try to) handle the events automatically. Events are used to indicate, for example, a media type, the end of a media stream or that the cache should be flushed.

Events may contain several of the following items:

* A subtype indicating the type of the contained event.

* The other contents of the event depend on the specific event type.
