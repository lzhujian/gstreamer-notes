## 1. Foundations

### Elements and Plugins

Elements是Gstreamer的核心，gstreamer开发中，element是继承自 *GstElement* 类的对象。将一系列的 element 连接起来构成pipeline或者bin，来提供功能。但是，仅仅开发一个 element 是不够的，需要 *plugin* 来封装element，让gstreamer能够加载它。plugin是能加载的代码块，通常为动态链接库（shared object or dynamic linked library）

### GstMiniObject, Buffers and Events

GstMiniObject is the structure used to hold these chunks of data. GStreamer的所有流数据（stream data）分割为 *chunk* 块，从一个element的srcpad传递到另一个element的sinkpad。*GstMiniObject* 是用于保存这些数据块的结构体。*GstMiniObject* 包含如下重要的类型：

* 确切的type指示该GstMiniObject是什么数据类型(event, buffer, ...)

* 引用计数指示当前element引用miniboject的个数，当refcount减到0时，miniobject将会disposed，内存将会释放。

