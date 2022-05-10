# Sometimes and Request Pads

相对于一直可用的 _always_ pads, _sometimes_ pads 只有在特定场景下创建，_request_ pads则是application请求时创建。pads availability(always, sometimes, request) 在 pad template 可以查看。

## Sometimes Pads

_Sometimes Pads_ 在特定条件下创建，通常取决于流的内容。demuxer通常需要解析stream header，决定在应用中嵌入哪些流(video, audi, subtitle), 然后为每一路流创建一个 _sometimes pad_. 当数据流dispose时 _Sometimes pads_ 也会dispose(i.e. state PAUSED -> READY). 因为可能重新activate pipeline 并且 seek 到 EOS 之前的播放位置，因此不应该在 EOS 时dispose pad。收到EOS后，stream仍然保持可用，至少保持到流数据dispose。任何情况下，element总是pad的owner.

```c
static GstStaticPadTemplate src_factory =
GST_STATIC_PAD_TEMPLATE (
  "src_%u",
  GST_PAD_SRC,
  GST_PAD_SOMETIMES,
  GST_STATIC_CAPS ("ANY")
);

static void
gst_my_filter_class_init (GstMyFilterClass *klass)
{
  GstElementClass *element_class = GST_ELEMENT_CLASS (klass);
[..]
  gst_element_class_add_pad_template (element_class,
    gst_static_pad_template_get (&src_factory));
[..]
}

/* create pad and add to element */
padname = g_strdup_printf ("src_%u", n);
pad = gst_pad_new_from_static_template (src_factory, padname);
g_free (padname);

/* here, you would set _event () and _query () functions */

/* need to activate the pad before adding */
gst_pad_set_active (pad, TRUE);

gst_element_add_pad (element, pad);

/* push event / data to pad */
gst_pad_push_event (pad, gst_event_ref (event));
gst_pad_push (pad, sub);
```

## Request Pads

_Request pads_ 类似于 _sometimes pads_，区别在于 _request pads_ 在element外部请求时创建。通常应用于muxers，对于应用需要输出的流，请求一个sink pad。也运用于输入或输出pads数量可变的elements，比如tee(multi-output) or input-selector (multi-input).

要实现 _request pads_, 需要提供 GST_PAD_REQUEST padtemplate，实现 GstElement 的 request_new_pad 虚函数. 清理时需要实现 release_pad 虚函数.

```c
static GstPad * gst_my_filter_request_new_pad   (GstElement     *element,
                         GstPadTemplate *templ,
                                                 const gchar    *name,
                                                 const GstCaps  *caps);

static void gst_my_filter_release_pad (GstElement *element,
                                       GstPad *pad);

static GstStaticPadTemplate sink_factory =
GST_STATIC_PAD_TEMPLATE (
  "sink_%u",
  GST_PAD_SINK,
  GST_PAD_REQUEST,
  GST_STATIC_CAPS ("ANY")
);

static void
gst_my_filter_class_init (GstMyFilterClass *klass)
{
  GstElementClass *element_class = GST_ELEMENT_CLASS (klass);
[..]
  gst_element_class_add_pad_template (klass,
    gst_static_pad_template_get (&sink_factory));
[..]
  element_class->request_new_pad = gst_my_filter_request_new_pad;
  element_class->release_pad = gst_my_filter_release_pad;
}

static GstPad *
gst_my_filter_request_new_pad (GstElement     *element,
                   GstPadTemplate *templ,
                   const gchar    *name,
                               const GstCaps  *caps)
{
  GstPad *pad;
  GstMyFilterInputContext *context;

  context = g_new0 (GstMyFilterInputContext, 1);
  pad = gst_pad_new_from_template (templ, name);
  gst_pad_set_element_private (pad, context);

  /* normally, you would set _chain () and _event () functions here */

  gst_element_add_pad (element, pad);

  return pad;
}

static void
gst_my_filter_release_pad (GstElement *element,
                           GstPad *pad)
{
  GstMyFilterInputContext *context;

  context = gst_pad_get_element_private (pad);
  g_free (context);

  gst_element_remove_pad (element, pad);
}
```
