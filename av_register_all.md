#av\_register\_all

```av_register_all();```声明在 libavformat/avformat.h 中：

```
/**
 * Initialize libavformat and register all the muxers, demuxers and
 * protocols. If you do not call this function, then you can select
 * exactly which formats you want to support.
 *
 * @see av_register_input_format()
 * @see av_register_output_format()
 */
void av_register_all(void);
```

实现在 libavformat/allformats.c 中：

```
#include "avformat.h"
#include "rtp.h"
#include "rdt.h"
#include "url.h"
#include "version.h"

#define REGISTER_MUXER(X, x)                                            \
    {                                                                   \
        extern AVOutputFormat ff_##x##_muxer;                           \
        if (CONFIG_##X##_MUXER)                                         \
            av_register_output_format(&ff_##x##_muxer);                 \
    }

#define REGISTER_DEMUXER(X, x)                                          \
    {                                                                   \
        extern AVInputFormat ff_##x##_demuxer;                          \
        if (CONFIG_##X##_DEMUXER)                                       \
            av_register_input_format(&ff_##x##_demuxer);                \
    }

#define REGISTER_MUXDEMUX(X, x) REGISTER_MUXER(X, x); REGISTER_DEMUXER(X, x)

void av_register_all(void)
{
    static int initialized;

    if (initialized)
        return;

    avcodec_register_all();

    /* (de)muxers */
    REGISTER_MUXER   (A64,              a64);
    ...
#if CONFIG_RTPDEC
    ff_register_rtp_dynamic_payload_handlers();
    ff_register_rdt_dynamic_payload_handlers();
#endif
    ...
    /* image demuxers */
    REGISTER_DEMUXER (IMAGE_BMP_PIPE,        image_bmp_pipe);
    ...
    /* external libraries */
    REGISTER_MUXER   (CHROMAPRINT,      chromaprint);
    initialized = 1;
}
```

```ff_register_rtp_dynamic_payload_handlers()``` 是用于注册 rtp 负载的 handler，[详细分析](ff_register_rtp_dynamic_payload_handlers.md)

[返回](ijkplayer_main.md)