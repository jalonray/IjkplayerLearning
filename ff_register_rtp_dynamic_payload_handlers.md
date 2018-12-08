#ff\_register\_rtp\_dynamic\_payload\_handlers

的声明在 libavformat/rtpdec.h 中：

```
typedef struct RTPDynamicProtocolHandler RTPDynamicProtocolHandler;

struct RTPDynamicProtocolHandler {
    const char *enc_name;
    enum AVMediaType codec_type;
    enum AVCodecID codec_id;
    enum AVStreamParseType need_parsing;
    int static_payload_id; /* 0 means no payload id is set. 0 is a valid
                            * payload ID (PCMU), too, but that format doesn't
                            * require any custom depacketization code. */
    int priv_data_size;

    /** Initialize dynamic protocol handler, called after the full rtpmap line is parsed, may be null */
    int (*init)(AVFormatContext *s, int st_index, PayloadContext *priv_data);
    /** Parse the a= line from the sdp field */
    int (*parse_sdp_a_line)(AVFormatContext *s, int st_index,
                            PayloadContext *priv_data, const char *line);
    /** Free any data needed by the rtp parsing for this dynamic data.
      * Don't free the protocol_data pointer itself, that is freed by the
      * caller. This is called even if the init method failed. */
    void (*close)(PayloadContext *protocol_data);
    /** Parse handler for this dynamic packet */
    DynamicPayloadPacketHandlerProc parse_packet;
    int (*need_keyframe)(PayloadContext *context);

    struct RTPDynamicProtocolHandler *next;
};

void ff_register_rtp_dynamic_payload_handlers(void);
```

实现在 libavformat/rtpdec.c 中：

```
#include "libavutil/mathematics.h"
#include "libavutil/avstring.h"
#include "libavutil/intreadwrite.h"
#include "libavutil/time.h"

#include "avformat.h"
#include "network.h"
#include "srtp.h"
#include "url.h"
#include "rtpdec.h"
#include "rtpdec_formats.h"
 
static RTPDynamicProtocolHandler *rtp_first_dynamic_payload_handler = NULL;

void ff_register_dynamic_payload_handler(RTPDynamicProtocolHandler *handler)
{
    handler->next = rtp_first_dynamic_payload_handler;
    rtp_first_dynamic_payload_handler = handler;
}

void ff_register_rtp_dynamic_payload_handlers(void)
{
    ff_register_dynamic_payload_handler(&ff_ac3_dynamic_handler);
    ...
}
```
[返回](av_register_all.md)