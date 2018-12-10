# ijkav\_register\_all

```ijkav_register_all``` 方法实现在 ijkmedia/ijkplayer/ijkavformat/allformats.c 中：

```
/**
 * This file is part of ijkPlayer.
 * Based on libavformat/allformats.c
 */

#include "libavformat/avformat.h"
#include "libavformat/url.h"
#include "libavformat/version.h"

#define IJK_REGISTER_DEMUXER(x)                                         \
    {                                                                   \
        extern AVInputFormat ijkff_##x##_demuxer;                       \
        ijkav_register_input_format(&ijkff_##x##_demuxer);              \
    }

#define IJK_REGISTER_PROTOCOL(x)                                        \
    {                                                                   \
        extern URLProtocol ijkimp_ff_##x##_protocol;                        \
        int ijkav_register_##x##_protocol(URLProtocol *protocol, int protocol_size);\
        ijkav_register_##x##_protocol(&ijkimp_ff_##x##_protocol, sizeof(URLProtocol));  \
    }

static struct AVInputFormat *ijkav_find_input_format(const char *iformat_name)
{
    AVInputFormat *fmt = NULL;
    if (!iformat_name)
        return NULL;
    while ((fmt = av_iformat_next(fmt))) {
        if (!fmt->name)
            continue;
        if (!strcmp(iformat_name, fmt->name))
            return fmt;
    }
    return NULL;
}

static void ijkav_register_input_format(AVInputFormat *iformat)
{
    if (ijkav_find_input_format(iformat->name)) {
        av_log(NULL, AV_LOG_WARNING, "skip     demuxer : %s (duplicated)\n", iformat->name);
    } else {
        av_log(NULL, AV_LOG_INFO,    "register demuxer : %s\n", iformat->name);
        av_register_input_format(iformat);
    }
}

void ijkav_register_all(void)
{
    static int initialized;

    if (initialized)
        return;
    initialized = 1;

    av_register_all();

    /* protocols */
    av_log(NULL, AV_LOG_INFO, "===== custom modules begin =====\n");
#ifdef __ANDROID__
    IJK_REGISTER_PROTOCOL(ijkmediadatasource);
#endif
    IJK_REGISTER_PROTOCOL(ijkio);
    IJK_REGISTER_PROTOCOL(async);
    IJK_REGISTER_PROTOCOL(ijklongurl);
    IJK_REGISTER_PROTOCOL(ijktcphook);
    IJK_REGISTER_PROTOCOL(ijkhttphook);
    IJK_REGISTER_PROTOCOL(ijksegment);
    /* demuxers */
    IJK_REGISTER_DEMUXER(ijklivehook);
    av_log(NULL, AV_LOG_INFO, "===== custom modules end =====\n");
}
```
上面的 av\_register\_all() 应该在之前已经调用过了- -。

前面的宏定义 ```IJK_REGISTER_PROTOCOL(x)``` 在预编译时会生成对应的 ```extern URLProtocol ijkimp_ff_##x##_protocol;``` 对象声明和 ```int ijkav_register_##x##_protocol(URLProtocol *protocol, int protocol_size);``` 方法声明，并调用执行对应的方法。

再看后面的 ```IJK_REGISTER_PROTOCOL(x)``` 方法的调用，每个参数，ijkio，async... 等等都有对应的 c 文件，内有对应 ```URLProtocol ijkimp_ff_##x##_protocol``` 的实例。

之后是 ```int ijkav_register_##x##_protocol(URLProtocol *protocol, int protocol_size);``` 的实现，对应在根目录的 android/contrib/ffmpeg-armv7a/libavformat/ijkutils.c 中：

```
#include <stdlib.h>
#include "url.h"

#define IJK_FF_PROTOCOL(x)                                                                          \
extern URLProtocol ff_##x##_protocol;                                                               \
int ijkav_register_##x##_protocol(URLProtocol *protocol, int protocol_size);                        \
int ijkav_register_##x##_protocol(URLProtocol *protocol, int protocol_size)                         \
 {                                                                                                   \
    if (protocol_size != sizeof(URLProtocol)) {                                                     \
        av_log(NULL, AV_LOG_ERROR, "ijkav_register_##x##_protocol: ABI mismatch.\n");               \
        return -1;                                                                                  \
    }                                                                                               \
    memcpy(&ff_##x##_protocol, protocol, protocol_size);                                            \
    return 0;                                                                                       \
}

 #define IJK_DUMMY_PROTOCOL(x)                                       \
 IJK_FF_PROTOCOL(x);                                                 \
static const AVClass ijk_##x##_context_class = {                    \
    .class_name = #x,                                               \
    .item_name  = av_default_item_name,                             \
    .version    = LIBAVUTIL_VERSION_INT,                            \
    };                                                              \
                                                                     \
URLProtocol ff_##x##_protocol = {                                   \
     .name                = #x,                                      \
     .url_open2           = ijkdummy_open,                           \
     .priv_data_size      = 1,                                       \
     .priv_data_class     = &ijk_##x##_context_class,                \
};

static int ijkdummy_open(URLContext *h, const char *arg, int flags, AVDictionary **options)
{
    return -1;
}
 
IJK_FF_PROTOCOL(async);
IJK_DUMMY_PROTOCOL(ijkmediadatasource);
IJK_DUMMY_PROTOCOL(ijkhttphook);
IJK_DUMMY_PROTOCOL(ijklongurl);
IJK_DUMMY_PROTOCOL(ijksegment);
IJK_DUMMY_PROTOCOL(ijktcphook);
IJK_DUMMY_PROTOCOL(ijkio);
```

详情： [URLProtocol](URLProtocol.md)，[ijkio](ijkio.md)，[ijkmediadatasource](ijkmediadatasource.md)... 均定义在 ijkmedia/ijkplayer/ijkavformat/ 下。

[返回](ijkplayer_main.md)