# ijkPlayer Android 源码研究

## 加载 ijkplayer.so 初始化

当前目录：android/ijkplayer/ijkplayer-armv7a/src/main/jni/

ijkmedia/ijkplayer/android/ijkplayer\_jni.c

头部：

```
#include <assert.h>
#include <string.h>
#include <pthread.h>
#include <jni.h>
#include <unistd.h>
#include "j4a/class/java/util/ArrayList.h"
#include "j4a/class/android/os/Bundle.h"
#include "j4a/class/tv/danmaku/ijk/media/player/IjkMediaPlayer.h"
#include "j4a/class/tv/danmaku/ijk/media/player/misc/IMediaDataSource.h"
#include "j4a/class/tv/danmaku/ijk/media/player/misc/IAndroidIO.h"
#include "ijksdl/ijksdl_log.h"
#include "../ff_ffplay.h"
#include "ffmpeg_api_jni.h"
#include "ijkplayer_android_def.h"
#include "ijkplayer_android.h"
#include "ijksdl/android/ijksdl_android_jni.h"
#include "ijksdl/android/ijksdl_codec_android_mediadef.h"
#include "ijkavformat/ijkavformat.h"

#define JNI_MODULE_PACKAGE      "tv/danmaku/ijk/media/player"
#define JNI_CLASS_IJKPLAYER     "tv/danmaku/ijk/media/player/IjkMediaPlayer"
#define JNI_IJK_MEDIA_EXCEPTION "tv/danmaku/ijk/media/player/exceptions/IjkMediaException"

#define IJK_CHECK_MPRET_GOTO(retval, env, label) \
    JNI_CHECK_GOTO((retval != EIJK_INVALID_STATE), env, "java/lang/IllegalStateException", NULL, label); \
    JNI_CHECK_GOTO((retval != EIJK_OUT_OF_MEMORY), env, "java/lang/OutOfMemoryError", NULL, label); \
    JNI_CHECK_GOTO((retval == 0), env, JNI_IJK_MEDIA_EXCEPTION, NULL, label);

static JavaVM* g_jvm;

typedef struct player_fields_t {
    pthread_mutex_t mutex;
    jclass clazz;
} player_fields_t;
static player_fields_t g_clazz;
```

初始化：

```
static JNINativeMethod g_methods[] = {
    {
        "_setDataSource",
        "(Ljava/lang/String;[Ljava/lang/String;[Ljava/lang/String;)V",
        (void *) IjkMediaPlayer_setDataSourceAndHeaders
    },
    { "_setDataSourceFd",       "(I)V",     (void *) IjkMediaPlayer_setDataSourceFd },
    { "_setDataSource",         "(Ltv/danmaku/ijk/media/player/misc/IMediaDataSource;)V", (void *)IjkMediaPlayer_setDataSourceCallback },
    { "_setAndroidIOCallback",  "(Ltv/danmaku/ijk/media/player/misc/IAndroidIO;)V", (void *)IjkMediaPlayer_setAndroidIOCallback },

    { "_setVideoSurface",       "(Landroid/view/Surface;)V", (void *) IjkMediaPlayer_setVideoSurface },
    { "_prepareAsync",          "()V",      (void *) IjkMediaPlayer_prepareAsync },
    { "_start",                 "()V",      (void *) IjkMediaPlayer_start },
    { "_stop",                  "()V",      (void *) IjkMediaPlayer_stop },
    { "seekTo",                 "(J)V",     (void *) IjkMediaPlayer_seekTo },
    { "_pause",                 "()V",      (void *) IjkMediaPlayer_pause },
    { "isPlaying",              "()Z",      (void *) IjkMediaPlayer_isPlaying },
    { "getCurrentPosition",     "()J",      (void *) IjkMediaPlayer_getCurrentPosition },
    { "getDuration",            "()J",      (void *) IjkMediaPlayer_getDuration },
    { "_release",               "()V",      (void *) IjkMediaPlayer_release },
    { "_reset",                 "()V",      (void *) IjkMediaPlayer_reset },
    { "setVolume",              "(FF)V",    (void *) IjkMediaPlayer_setVolume },
    { "getAudioSessionId",      "()I",      (void *) IjkMediaPlayer_getAudioSessionId },
    { "native_init",            "()V",      (void *) IjkMediaPlayer_native_init },
    { "native_setup",           "(Ljava/lang/Object;)V", (void *) IjkMediaPlayer_native_setup },
    { "native_finalize",        "()V",      (void *) IjkMediaPlayer_native_finalize },

    { "_setOption",             "(ILjava/lang/String;Ljava/lang/String;)V", (void *) IjkMediaPlayer_setOption },
    { "_setOption",             "(ILjava/lang/String;J)V",                  (void *) IjkMediaPlayer_setOptionLong },

    { "_getColorFormatName",    "(I)Ljava/lang/String;",    (void *) IjkMediaPlayer_getColorFormatName },
    { "_getVideoCodecInfo",     "()Ljava/lang/String;",     (void *) IjkMediaPlayer_getVideoCodecInfo },
    { "_getAudioCodecInfo",     "()Ljava/lang/String;",     (void *) IjkMediaPlayer_getAudioCodecInfo },
    { "_getMediaMeta",          "()Landroid/os/Bundle;",    (void *) IjkMediaPlayer_getMediaMeta },
    { "_setLoopCount",          "(I)V",                     (void *) IjkMediaPlayer_setLoopCount },
    { "_getLoopCount",          "()I",                      (void *) IjkMediaPlayer_getLoopCount },
    { "_getPropertyFloat",      "(IF)F",                    (void *) ijkMediaPlayer_getPropertyFloat },
    { "_setPropertyFloat",      "(IF)V",                    (void *) ijkMediaPlayer_setPropertyFloat },
    { "_getPropertyLong",       "(IJ)J",                    (void *) ijkMediaPlayer_getPropertyLong },
    { "_setPropertyLong",       "(IJ)V",                    (void *) ijkMediaPlayer_setPropertyLong },
    { "_setStreamSelected",     "(IZ)V",                    (void *) ijkMediaPlayer_setStreamSelected },

    { "native_profileBegin",    "(Ljava/lang/String;)V",    (void *) IjkMediaPlayer_native_profileBegin },
    { "native_profileEnd",      "()V",                      (void *) IjkMediaPlayer_native_profileEnd },

    { "native_setLogLevel",     "(I)V",                     (void *) IjkMediaPlayer_native_setLogLevel },
    { "_setFrameAtTime",        "(Ljava/lang/String;JJII)V", (void *) IjkMediaPlayer_setFrameAtTime },
};

JNIEXPORT jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
    JNIEnv* env = NULL;

    g_jvm = vm;
    if ((*vm)->GetEnv(vm, (void**) &env, JNI_VERSION_1_4) != JNI_OK) {
        return -1;
    }
    assert(env != NULL);

    pthread_mutex_init(&g_clazz.mutex, NULL );

    // FindClass returns LocalReference
    IJK_FIND_JAVA_CLASS(env, g_clazz.clazz, JNI_CLASS_IJKPLAYER);
    (*env)->RegisterNatives(env, g_clazz.clazz, g_methods, NELEM(g_methods) );

    ijkmp_global_init();
    ijkmp_global_set_inject_callback(inject_callback);

    FFmpegApi_global_init(env);

    return JNI_VERSION_1_4;
}

JNIEXPORT void JNI_OnUnload(JavaVM *jvm, void *reserved)
{
    ijkmp_global_uninit();

    pthread_mutex_destroy(&g_clazz.mutex);
}
```
首先分析 JNI\_OnLoad

```
JNIEXPORT jint JNI_OnLoad(JavaVM *vm, void *reserved)
{
    JNIEnv* env = NULL;
    g_jvm = vm;
    if ((*vm)->GetEnv(vm, (void**) &env, JNI_VERSION_1_4) != JNI_OK) {
        return -1;
    }
    assert(env != NULL);

    pthread_mutex_init(&g_clazz.mutex, NULL );

    // FindClass returns LocalReference
    IJK_FIND_JAVA_CLASS(env, g_clazz.clazz, JNI_CLASS_IJKPLAYER);
    (*env)->RegisterNatives(env, g_clazz.clazz, g_methods, NELEM(g_methods) );

    ijkmp_global_init();
    ijkmp_global_set_inject_callback(inject_callback);

    FFmpegApi_global_init(env);

    return JNI_VERSION_1_4;
}
```

### 第一句```g_jvm = vm;```

```g_jvm = vm;``` 定义在：
ijkmedia/ijksdl/android/ijksdl\_android\_jni.c

```
static JavaVM *g_jvm;

static pthread_key_t g_thread_key;
static pthread_once_t g_key_once = PTHREAD_ONCE_INIT;
```

### 第二句```pthread_mutex_init(&g_clazz.mutex, NULL);```

```pthread_mutex_init(&g_clazz.mutex, NULL);``` 是定义在 ffmpeg 中的，ffmpeg 在 ijkplayer 根目录的 extra/ 里。该函数定义在 ffmpeg/compat/os2threads.h 或是 ffmpeg/compat/w32pthreads.h 中，用于适配不同的底层。然后封装在 ffmpeg/libavutil/thread.h 里统一调用。

[Thread.h 详情](ffmpeg_libavutil_thread.md)

### 第三句```IJK_FIND_JAVA_CLASS(env, g_clazz.clazz, JNI_CLASS_IJKPLAYER);```

```IJK_FIND_JAVA_CLASS(env, g_clazz.clazz, JNI_CLASS_IJKPLAYER);``` 定义在：
ijkmedia/ijksdl/android/ijksdl\_android\_jni.h

```
#define IJK_FIND_JAVA_CLASS(env__, var__, classsign__) \
    do { \
        jclass clazz = (*env__)->FindClass(env__, classsign__); \
        if (J4A_ExceptionCheck__catchAll(env) || !(clazz)) { \
            ALOGE("FindClass failed: %s", classsign__); \
            return -1; \
        } \
        var__ = (*env__)->NewGlobalRef(env__, clazz); \
        if (J4A_ExceptionCheck__catchAll(env) || !(var__)) { \
            ALOGE("FindClass::NewGlobalRef failed: %s", classsign__); \
            (*env__)->DeleteLocalRef(env__, clazz); \
            return -1; \
        } \
        (*env__)->DeleteLocalRef(env__, clazz); \
    } while(0);
```
通过 classsign__ 和 env__ 得到 jvm 中 class 的实例引用，并以 GlobalRef 的方式赋予 var__。对于 GlobalRef 和 LocalRef 的详细讲解，可参考 [Google 文档](https://developer.android.com/training/articles/perf-jni#local-and-global-references) 和 [java 文档](https://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/design.html#wp1242)

### 第四句 ```(*env)->RegisterNatives(env, g_clazz.clazz, g_methods, NELEM(g_methods) );```

注册 g_methods 中的所有 Native 方法。[方法详解](https://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/functions.html#wp5833)

之后的 Java 和 Native 的方法表便可从 g_methods 中查找。

### 第五句```ijkmp_global_init();```

```ijkmp_global_init();``` 实现在：ijkmedia/ijkplayer/ijkplayer.c 中

```
void ijkmp_global_init()
{
    ffp_global_init();
}
```

```ffp_global_init();``` 实现在：ijkmedia/ijkplayer/ff\_ffplay.c 中

```
static AVPacket flush_pkt;
static bool g_ffmpeg_global_inited = false;

// lock manager 的回调
static int lockmgr(void **mtx, enum AVLockOp op)
{
    switch (op) {
    case AV_LOCK_CREATE:
        *mtx = SDL_CreateMutex();
        if (!*mtx) {
            av_log(NULL, AV_LOG_FATAL, "SDL_CreateMutex(): %s\n", SDL_GetError());
            return 1;
        }
        return 0;
    case AV_LOCK_OBTAIN:
        return !!SDL_LockMutex(*mtx);
    case AV_LOCK_RELEASE:
        return !!SDL_UnlockMutex(*mtx);
    case AV_LOCK_DESTROY:
        SDL_DestroyMutex(*mtx);
        return 0;
    }
    return 1;
}

// av_log_set_call_back 的回调
static void ffp_log_callback_brief(void *ptr, int level, const char *fmt, va_list vl)
{
    if (level > av_log_get_level())
        return;

    int ffplv __unused = log_level_av_to_ijk(level);
    VLOG(ffplv, IJK_LOG_TAG, fmt, vl);
}

void ffp_global_init()
{
    if (g_ffmpeg_global_inited)
        return;

    ALOGD("ijkmediaplayer version : %s", ijkmp_version());
    /* register all codecs, demux and protocols */
    avcodec_register_all();
#if CONFIG_AVDEVICE
    avdevice_register_all();
#endif
#if CONFIG_AVFILTER
    avfilter_register_all();
#endif
    av_register_all();

    ijkav_register_all();

    avformat_network_init();

    av_lockmgr_register(lockmgr);
    av_log_set_callback(ffp_log_callback_brief);

    av_init_packet(&flush_pkt);
    flush_pkt.data = (uint8_t *)&flush_pkt;

    g_ffmpeg_global_inited = true;
}
```
依然逐句分析

```avcodec_register_all()```，注册所有的 codec 方法，[方法详解](avcodec_register_all.md)。

之后是两个配制：

```
#if CONFIG_AVDEVICE
...
#endif
#if CONFIG_AVFILTER
...
#endif
```
定义位于 ffmpeg/libffmpeg/config.h：

```
/* Automatically generated by configure - do not modify! */
#define CONFIG_AVDEVICE 0
#define CONFIG_AVFILTER 0
``` 

可见是通过配制文件生成的。

之后是```avdevice_register_all()```，注册所有的硬件方法，[方法详解](libavdevice_avdevice.md)。

```avfilter_register_all();```，注册所有的 filter 方法，[方法详解](libavfilter_avfilter.md)。

```av_register_all();```，初始化 libavformat 并注册所有的 muxers，demuxers，和 protocols，[方法详解](av_register_all.md)。

```ijkav_register_all();```，注册所有 ijkplayer 自定的一些协议，[方法详解](ijkav_register_all.md)。

```avformat_network_init();```，初始化网络的内容，[方法详解](avformat_network_init.md)。

```av_lockmgr_register(lockmgr);```，注册一个用户提供的锁 lock manager，[方法详解](av_lockmgr_register.md)。

```av_log_set_callback(ffp_log_callback_brief);```，设置 logging 的回调，[方法详解](av_log_set_callback.md)。

```av_init_packet(&flush_pkt);```，初始化一个 packet，[方法详解](av_init_packet.md)。

```flush_pkt.data = (uint8_t *)&flush_pkt;```，截 flush_pkt 的前 8 位作 data。

```g_ffmpeg_global_inited = true;```，标记初始化完成。

### 第六句 ```ijkmp_global_set_inject_callback(inject_callback);```

将之赋予类型为 FFPlayer 的 app_ctx->func_on_app_event 上，在对应 app_event 发生变化时便可及时获取回调。

[方法详解](ijkmp_global_set_inject_callback.md)

回调的实现在 ijkplayer_ini.c 中：

```
// NOTE: support to be called from read_thread
static int
inject_callback(void *opaque, int what, void *data, size_t data_size)
{
    JNIEnv     *env     = NULL;
    jobject     jbundle = NULL;
    int         ret     = -1;
    SDL_JNI_SetupThreadEnv(&env);

    jobject weak_thiz = (jobject) opaque;
    if (weak_thiz == NULL )
        goto fail;
    switch (what) {
        case AVAPP_CTRL_WILL_HTTP_OPEN:
        case AVAPP_CTRL_WILL_LIVE_OPEN:
        case AVAPP_CTRL_WILL_CONCAT_SEGMENT_OPEN: {
            AVAppIOControl *real_data = (AVAppIOControl *)data;
            real_data->is_handled = 0;

            jbundle = J4AC_Bundle__Bundle__catchAll(env);
            if (!jbundle) {
                ALOGE("%s: J4AC_Bundle__Bundle__catchAll failed for case %d\n", __func__, what);
                goto fail;
            }
            J4AC_Bundle__putString__withCString__catchAll(env, jbundle, "url", real_data->url);
            J4AC_Bundle__putInt__withCString__catchAll(env, jbundle, "segment_index", real_data->segment_index);
            J4AC_Bundle__putInt__withCString__catchAll(env, jbundle, "retry_counter", real_data->retry_counter);
            real_data->is_handled = J4AC_IjkMediaPlayer__onNativeInvoke(env, weak_thiz, what, jbundle);
            if (J4A_ExceptionCheck__catchAll(env)) {
                goto fail;
            }

            J4AC_Bundle__getString__withCString__asCBuffer(env, jbundle, "url", real_data->url, sizeof(real_data->url));
            if (J4A_ExceptionCheck__catchAll(env)) {
                goto fail;
            }
            ret = 0;
            break;
        }
        case AVAPP_EVENT_WILL_HTTP_OPEN:
        case AVAPP_EVENT_DID_HTTP_OPEN:
        case AVAPP_EVENT_WILL_HTTP_SEEK:
        case AVAPP_EVENT_DID_HTTP_SEEK: {
            AVAppHttpEvent *real_data = (AVAppHttpEvent *) data;
            jbundle = J4AC_Bundle__Bundle__catchAll(env);
            if (!jbundle) {
                ALOGE("%s: J4AC_Bundle__Bundle__catchAll failed for case %d\n", __func__, what);
                goto fail;
            }
            J4AC_Bundle__putString__withCString__catchAll(env, jbundle, "url", real_data->url);
            J4AC_Bundle__putLong__withCString__catchAll(env, jbundle, "offset", real_data->offset);
            J4AC_Bundle__putInt__withCString__catchAll(env, jbundle, "error", real_data->error);
            J4AC_Bundle__putInt__withCString__catchAll(env, jbundle, "http_code", real_data->http_code);
            J4AC_Bundle__putLong__withCString__catchAll(env, jbundle, "file_size", real_data->filesize);
            J4AC_IjkMediaPlayer__onNativeInvoke(env, weak_thiz, what, jbundle);
            if (J4A_ExceptionCheck__catchAll(env))
                goto fail;
            ret = 0;
            break;
        }
        case AVAPP_CTRL_DID_TCP_OPEN:
        case AVAPP_CTRL_WILL_TCP_OPEN: {
            AVAppTcpIOControl *real_data = (AVAppTcpIOControl *)data;
            jbundle = J4AC_Bundle__Bundle__catchAll(env);
            if (!jbundle) {
                ALOGE("%s: J4AC_Bundle__Bundle__catchAll failed for case %d\n", __func__, what);
                goto fail;
            }
            J4AC_Bundle__putInt__withCString__catchAll(env, jbundle, "error", real_data->error);
            J4AC_Bundle__putInt__withCString__catchAll(env, jbundle, "family", real_data->family);
            J4AC_Bundle__putString__withCString__catchAll(env, jbundle, "ip", real_data->ip);
            J4AC_Bundle__putInt__withCString__catchAll(env, jbundle, "port", real_data->port);
            J4AC_Bundle__putInt__withCString__catchAll(env, jbundle, "fd", real_data->fd);
            J4AC_IjkMediaPlayer__onNativeInvoke(env, weak_thiz, what, jbundle);
            if (J4A_ExceptionCheck__catchAll(env))
                goto fail;
            ret = 0;
            break;
        }
        default: {
            ret = 0;
        }
    }
fail:
    SDL_JNI_DeleteLocalRefP(env, &jbundle);
    return ret;
}
```

### 第七句 ```FFmpegApi_global_init(env);```

该方法声明在 ijkmedia/ijkplayer/android/ffmpeg_api_jni.h 中：

```
int FFmpegApi_global_init(JNIEnv *env);
```

实现在 ijkmedia/ijkplayer/android/ffmpeg_api_jni.c 中：

```
#define JNI_CLASS_FFMPEG_API "tv/danmaku/ijk/media/player/ffmpeg/FFmpegApi"

int FFmpegApi_global_init(JNIEnv *env)
{
    int ret = 0;

    IJK_FIND_JAVA_CLASS(env, g_clazz.clazz, JNI_CLASS_FFMPEG_API);
    (*env)->RegisterNatives(env, g_clazz.clazz, g_methods, NELEM(g_methods));

    return ret;
}
```

将 FFmpegApi 的 JAVA 类加载到 JVM 当中。

### 第八句 ```return JNI_VERSION_1_4;```

返回 JNI_VERSION_1_4。


## IjkMediaPlayer 数据结构

声明在 ijkmedia/ijkplayer/ijkplayer.h 中：

```
typedef struct IjkMediaPlayer IjkMediaPlayer;
```

实现在 ijkmedia/ijkplayer/ijkplayer_internal.h 中：

```
#ifndef IJKPLAYER_ANDROID__IJKPLAYER_INTERNAL_H
#define IJKPLAYER_ANDROID__IJKPLAYER_INTERNAL_H

#include <assert.h>
#include "ijksdl/ijksdl.h"
#include "ff_fferror.h"
#include "ff_ffplay.h"
#include "ijkplayer.h"

struct IjkMediaPlayer {
    volatile int ref_count;
    pthread_mutex_t mutex;
    FFPlayer *ffplayer;

    int (*msg_loop)(void*);
    SDL_Thread *msg_thread;
    SDL_Thread _msg_thread;

    int mp_state;
    char *data_source;
    void *weak_thiz;

    int restart;
    int restart_from_beginning;
    int seek_req;
    long seek_msec; 
};

#endif
```


## message\_loop 状态机

于 ijkmedia/ijkplayer/android/ijkplayer\_jni.c 中：

```
static int message_loop(void *arg);

static int message_loop(void *arg)
{
    MPTRACE("%s\n", __func__);

    JNIEnv *env = NULL;
    if (JNI_OK != SDL_JNI_SetupThreadEnv(&env)) {
        ALOGE("%s: SetupThreadEnv failed\n", __func__);
        return -1;
    }

    IjkMediaPlayer *mp = (IjkMediaPlayer*) arg;
    JNI_CHECK_GOTO(mp, env, NULL, "mpjni: native_message_loop: null mp", LABEL_RETURN);

    message_loop_n(env, mp);

LABEL_RETURN:
    ijkmp_dec_ref_p(&mp);

    MPTRACE("message_loop exit");
    return 0;
}
```

前面是一些初始化和空判断，后面是引用的释放。中间 ```message_loop_n(env, mp);``` 是重点函数。

```
static void message_loop_n(JNIEnv *env, IjkMediaPlayer *mp)
{
    jobject weak_thiz = (jobject) ijkmp_get_weak_thiz(mp);
    JNI_CHECK_GOTO(weak_thiz, env, NULL, "mpjni: message_loop_n: null weak_thiz", LABEL_RETURN);

    while (1) {
        AVMessage msg;

        int retval = ijkmp_get_msg(mp, &msg, 1);
        if (retval < 0)
            break;

        // block-get should never return 0
        assert(retval > 0);

        switch (msg.what) {
        case FFP_MSG_FLUSH:
            MPTRACE("FFP_MSG_FLUSH:\n");
            post_event(env, weak_thiz, MEDIA_NOP, 0, 0);
            break;
        case FFP_MSG_ERROR:
            MPTRACE("FFP_MSG_ERROR: %d\n", msg.arg1);
            post_event(env, weak_thiz, MEDIA_ERROR, MEDIA_ERROR_IJK_PLAYER, msg.arg1);
            break;
        case FFP_MSG_PREPARED:
            MPTRACE("FFP_MSG_PREPARED:\n");
            post_event(env, weak_thiz, MEDIA_PREPARED, 0, 0);
            break;
        case FFP_MSG_COMPLETED:
            MPTRACE("FFP_MSG_COMPLETED:\n");
            post_event(env, weak_thiz, MEDIA_PLAYBACK_COMPLETE, 0, 0);
            break;
        case FFP_MSG_VIDEO_SIZE_CHANGED:
            MPTRACE("FFP_MSG_VIDEO_SIZE_CHANGED: %d, %d\n", msg.arg1, msg.arg2);
            post_event(env, weak_thiz, MEDIA_SET_VIDEO_SIZE, msg.arg1, msg.arg2);
            break;
        case FFP_MSG_SAR_CHANGED:
            MPTRACE("FFP_MSG_SAR_CHANGED: %d, %d\n", msg.arg1, msg.arg2);
            post_event(env, weak_thiz, MEDIA_SET_VIDEO_SAR, msg.arg1, msg.arg2);
            break;
        case FFP_MSG_VIDEO_RENDERING_START:
            MPTRACE("FFP_MSG_VIDEO_RENDERING_START:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_VIDEO_RENDERING_START, 0);
            break;
        case FFP_MSG_AUDIO_RENDERING_START:
            MPTRACE("FFP_MSG_AUDIO_RENDERING_START:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_AUDIO_RENDERING_START, 0);
            break;
        case FFP_MSG_VIDEO_ROTATION_CHANGED:
            MPTRACE("FFP_MSG_VIDEO_ROTATION_CHANGED: %d\n", msg.arg1);
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_VIDEO_ROTATION_CHANGED, msg.arg1);
            break;
        case FFP_MSG_AUDIO_DECODED_START:
            MPTRACE("FFP_MSG_AUDIO_DECODED_START:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_AUDIO_DECODED_START, 0);
            break;
        case FFP_MSG_VIDEO_DECODED_START:
            MPTRACE("FFP_MSG_VIDEO_DECODED_START:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_VIDEO_DECODED_START, 0);
            break;
        case FFP_MSG_OPEN_INPUT:
            MPTRACE("FFP_MSG_OPEN_INPUT:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_OPEN_INPUT, 0);
            break;
        case FFP_MSG_FIND_STREAM_INFO:
            MPTRACE("FFP_MSG_FIND_STREAM_INFO:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_FIND_STREAM_INFO, 0);
            break;
        case FFP_MSG_COMPONENT_OPEN:
            MPTRACE("FFP_MSG_COMPONENT_OPEN:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_COMPONENT_OPEN, 0);
            break;
        case FFP_MSG_BUFFERING_START:
            MPTRACE("FFP_MSG_BUFFERING_START:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_BUFFERING_START, msg.arg1);
            break;
        case FFP_MSG_BUFFERING_END:
            MPTRACE("FFP_MSG_BUFFERING_END:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_BUFFERING_END, msg.arg1);
            break;
        case FFP_MSG_BUFFERING_UPDATE:
            // MPTRACE("FFP_MSG_BUFFERING_UPDATE: %d, %d", msg.arg1, msg.arg2);
            post_event(env, weak_thiz, MEDIA_BUFFERING_UPDATE, msg.arg1, msg.arg2);
            break;
        case FFP_MSG_BUFFERING_BYTES_UPDATE:
            break;
        case FFP_MSG_BUFFERING_TIME_UPDATE:
            break;
        case FFP_MSG_SEEK_COMPLETE:
            MPTRACE("FFP_MSG_SEEK_COMPLETE:\n");
            post_event(env, weak_thiz, MEDIA_SEEK_COMPLETE, 0, 0);
            break;
        case FFP_MSG_ACCURATE_SEEK_COMPLETE:
            MPTRACE("FFP_MSG_ACCURATE_SEEK_COMPLETE:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_MEDIA_ACCURATE_SEEK_COMPLETE, msg.arg1);
            break;
        case FFP_MSG_PLAYBACK_STATE_CHANGED:
            break;
        case FFP_MSG_TIMED_TEXT:
            if (msg.obj) {
                jstring text = (*env)->NewStringUTF(env, (char *)msg.obj);
                post_event2(env, weak_thiz, MEDIA_TIMED_TEXT, 0, 0, text);
                J4A_DeleteLocalRef__p(env, &text);
            }
            else {
                post_event2(env, weak_thiz, MEDIA_TIMED_TEXT, 0, 0, NULL);
            }
            break;
        case FFP_MSG_GET_IMG_STATE:
            if (msg.obj) {
                jstring file_name = (*env)->NewStringUTF(env, (char *)msg.obj);
                post_event2(env, weak_thiz, MEDIA_GET_IMG_STATE, msg.arg1, msg.arg2, file_name);
                J4A_DeleteLocalRef__p(env, &file_name);
            }
            else {
                post_event2(env, weak_thiz, MEDIA_GET_IMG_STATE, msg.arg1, msg.arg2, NULL);
            }
            break;
        case FFP_MSG_VIDEO_SEEK_RENDERING_START:
            MPTRACE("FFP_MSG_VIDEO_SEEK_RENDERING_START:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_VIDEO_SEEK_RENDERING_START, msg.arg1);
            break;
        case FFP_MSG_AUDIO_SEEK_RENDERING_START:
            MPTRACE("FFP_MSG_AUDIO_SEEK_RENDERING_START:\n");
            post_event(env, weak_thiz, MEDIA_INFO, MEDIA_INFO_AUDIO_SEEK_RENDERING_START, msg.arg1);
            break;
        default:
            ALOGE("unknown FFP_MSG_xxx(%d)\n", msg.what);
            break;
        }
        msg_free_res(&msg);
    }

LABEL_RETURN:
    ;
}
```

```int retval = ijkmp_get_msg(mp, &msg, 1);``` 得到当前发送的 msg。[方法详解](ijkmp_get_msg.md)

```FFP_MSG_FLUSH``` 等一系列 EVENT 的[声明详解](ff_ffmsg.md)

对于每个 msg，都有或是 ```post_event``` 方法，或是 ```post_event2```，用来传 event。

```
inline static void post_event(JNIEnv *env, jobject weak_this, int what, int arg1, int arg2)
{
    // MPTRACE("post_event(%p, %p, %d, %d, %d)", (void*)env, (void*) weak_this, what, arg1, arg2);
    J4AC_IjkMediaPlayer__postEventFromNative(env, weak_this, what, arg1, arg2, NULL);
    // MPTRACE("post_event()=void");
}

inline static void post_event2(JNIEnv *env, jobject weak_this, int what, int arg1, int arg2, jobject obj)
{
    // MPTRACE("post_event2(%p, %p, %d, %d, %d, %p)", (void*)env, (void*) weak_this, what, arg1, arg2, (void*)obj);
    J4AC_IjkMediaPlayer__postEventFromNative(env, weak_this, what, arg1, arg2, obj);
    // MPTRACE("post_event2()=void");
}
```

可看到，都会走一个统一的回调方法 ```J4AC_IjkMediaPlayer__postEventFromNative```，方法声明在 ijkmedia/ijkj4a/j4a/class/tv/danmaku/ijk/media/player/IjkMediaPlayer.h 中：

```
#define J4AC_IjkMediaPlayer__postEventFromNative J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__postEventFromNative
```

实现在对应的 ijkmedia/ijkj4a/j4a/class/tv/danmaku/ijk/media/player/IjkMediaPlayer.c 中：

```
typedef struct J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer {
    jclass id;

    jfieldID field_mNativeMediaPlayer;
    jfieldID field_mNativeMediaDataSource;
    jfieldID field_mNativeAndroidIO;
    jmethodID method_postEventFromNative;
    jmethodID method_onSelectCodec;
    jmethodID method_onNativeInvoke;
} J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer;
static J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer;

void J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__postEventFromNative(JNIEnv *env, jobject weakThiz, jint what, jint arg1, jint arg2, jobject obj)
{
    (*env)->CallStaticVoidMethod(env, class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer.id, class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer.method_postEventFromNative, weakThiz, what, arg1, arg2, obj);
}
```

其中，```method_postEventFromNative``` 的初始化：

```
int J4A_loadClass__J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer(JNIEnv *env)
{
    ...
    
    sign = "tv/danmaku/ijk/media/player/IjkMediaPlayer";
    class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer.id = J4A_FindClass__asGlobalRef__catchAll(env, sign);
    if (class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer.id == NULL)
        goto fail;
    
    ...
    
    class_id = class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer.id;
    name     = "postEventFromNative";
    sign     = "(Ljava/lang/Object;IIILjava/lang/Object;)V";
    class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer.method_postEventFromNative = J4A_GetStaticMethodID__catchAll(env, class_id, name, sign);
    if (class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer.method_postEventFromNative == NULL)
        goto fail;
        
    ...
}
```

可知，是对应着 tv/danmaku/ijk/media/player/IjkMediaPlayer.Java 中的 postEventFromNative 方法。下面来分析一下 android/ijkplayer/ijkplayer-java/src/main/java/tv/danmaku/ijk/media/player/IjkMediaPlayer.java：

```
/*
 * Called from native code when an interesting event happens. This method
 * just uses the EventHandler system to post the event back to the main app
 * thread. We use a weak reference to the original IjkMediaPlayer object so
 * that the native code is safe from the object disappearing from underneath
 * it. (This is the cookie passed to native_setup().)
 */
@CalledByNative
private static void postEventFromNative(Object weakThiz, int what,
        int arg1, int arg2, Object obj) {
    if (weakThiz == null)
        return;

    @SuppressWarnings("rawtypes")
    IjkMediaPlayer mp = (IjkMediaPlayer) ((WeakReference) weakThiz).get();
    if (mp == null) {
        return;
    }

    if (what == MEDIA_INFO && arg1 == MEDIA_INFO_STARTED_AS_NEXT) {
        // this acquires the wakelock if needed, and sets the client side
        // state
        mp.start();
    }
    if (mp.mEventHandler != null) {
        Message m = mp.mEventHandler.obtainMessage(what, arg1, arg2, obj);
        mp.mEventHandler.sendMessage(m);
    }
}
```

这样就和 Java 的代码联系在一起了。然后再看 mEventHandler 声明：

```
private EventHandler mEventHandler;
```

初始化：

```
private void initPlayer(IjkLibLoader libLoader) {
    loadLibrariesOnce(libLoader);
    initNativeOnce();

    Looper looper;
    if ((looper = Looper.myLooper()) != null) {
        mEventHandler = new EventHandler(this, looper);
    } else if ((looper = Looper.getMainLooper()) != null) {
        mEventHandler = new EventHandler(this, looper);
    } else {
        mEventHandler = null;
    }

    /*
     * Native setup requires a weak reference to our object. It's easier to
     * create it here than in C++.
     */
    native_setup(new WeakReference<IjkMediaPlayer>(this));
}
```

EventHandler 是 IjkMediaPlayer 中的静态内部类，主要看 handleMessage 方法：

```
@Override
public void handleMessage(Message msg) {
    IjkMediaPlayer player = mWeakPlayer.get();
    if (player == null || player.mNativeMediaPlayer == 0) {
        DebugLog.w(TAG,
                "IjkMediaPlayer went away with unhandled events");
        return;
    }

    switch (msg.what) {
    case MEDIA_PREPARED:
        player.notifyOnPrepared();
        return;

    case MEDIA_PLAYBACK_COMPLETE:
        player.stayAwake(false);
        player.notifyOnCompletion();
        return;

    case MEDIA_BUFFERING_UPDATE:
        long bufferPosition = msg.arg1;
        if (bufferPosition < 0) {
            bufferPosition = 0;
        }

        long percent = 0;
        long duration = player.getDuration();
        if (duration > 0) {
            percent = bufferPosition * 100 / duration;
        }
        if (percent >= 100) {
            percent = 100;
        }

        // DebugLog.efmt(TAG, "Buffer (%d%%) %d/%d",  percent, bufferPosition, duration);
        player.notifyOnBufferingUpdate((int)percent);
        return;

    case MEDIA_SEEK_COMPLETE:
        player.notifyOnSeekComplete();
        return;

    case MEDIA_SET_VIDEO_SIZE:
        player.mVideoWidth = msg.arg1;
        player.mVideoHeight = msg.arg2;
        player.notifyOnVideoSizeChanged(player.mVideoWidth, player.mVideoHeight,
                player.mVideoSarNum, player.mVideoSarDen);
        return;

    case MEDIA_ERROR:
        DebugLog.e(TAG, "Error (" + msg.arg1 + "," + msg.arg2 + ")");
        if (!player.notifyOnError(msg.arg1, msg.arg2)) {
            player.notifyOnCompletion();
        }
        player.stayAwake(false);
        return;

    case MEDIA_INFO:
        switch (msg.arg1) {
            case MEDIA_INFO_VIDEO_RENDERING_START:
                DebugLog.i(TAG, "Info: MEDIA_INFO_VIDEO_RENDERING_START\n");
                break;
        }
        player.notifyOnInfo(msg.arg1, msg.arg2);
        // No real default action so far.
        return;
    case MEDIA_TIMED_TEXT:
        if (msg.obj == null) {
            player.notifyOnTimedText(null);
        } else {
            IjkTimedText text = new IjkTimedText(new Rect(0, 0, 1, 1), (String)msg.obj);
            player.notifyOnTimedText(text);
        }
        return;
    case MEDIA_NOP: // interface test message - ignore
        break;

    case MEDIA_SET_VIDEO_SAR:
        player.mVideoSarNum = msg.arg1;
        player.mVideoSarDen = msg.arg2;
        player.notifyOnVideoSizeChanged(player.mVideoWidth, player.mVideoHeight,
                player.mVideoSarNum, player.mVideoSarDen);
        break;

    default:
        DebugLog.e(TAG, "Unknown message type " + msg.what);
    }
}
```

每种回调可以慢慢分析，此不赘述。但是可以看出，这里的信息定义与之前的相比并不完整，MEDIA_PREPARED 等的[声明详解](media_msg.md)，此处可以扩展以获得更多的信息。


## native 初始化

通过 g_method，可以找到 ```"native_init", "()V", (void *) IjkMediaPlayer_native_init```，这是在 example 中 IjkMediaPlayer.java 的构造方法中调用的。

native 方法实现在 ijkmedia/ijkplayer/android/ijkplayer_jni.c 中：

```
static void
IjkMediaPlayer_native_init(JNIEnv *env)
{
    MPTRACE("%s\n", __func__);
}
```

仅仅是一个日志。


## 初始化设置

通过 g_method，可以找到 ```"native_setup", "(Ljava/lang/Object;)V", (void *) IjkMediaPlayer_native_setup```，这是在 example 中 IjkMediaPlayer.java 的构造方法中调用的。

native 方法实现在 ijkmedia/ijkplayer/android/ijkplayer_jni.c 中：

```
static void IjkMediaPlayer_native_setup(JNIEnv *env, jobject thiz, jobject weak_this);

static void
IjkMediaPlayer_native_setup(JNIEnv *env, jobject thiz, jobject weak_this)
{
    MPTRACE("%s\n", __func__);
    IjkMediaPlayer *mp = ijkmp_android_create(message_loop);
    JNI_CHECK_GOTO(mp, env, "java/lang/OutOfMemoryError", "mpjni: native_setup: ijkmp_create() failed", LABEL_RETURN);

    jni_set_media_player(env, thiz, mp);
    ijkmp_set_weak_thiz(mp, (*env)->NewGlobalRef(env, weak_this));
    ijkmp_set_inject_opaque(mp, ijkmp_get_weak_thiz(mp));
    ijkmp_set_ijkio_inject_opaque(mp, ijkmp_get_weak_thiz(mp));
    ijkmp_android_set_mediacodec_select_callback(mp, mediacodec_select_callback, ijkmp_get_weak_thiz(mp));

LABEL_RETURN:
    ijkmp_dec_ref_p(&mp);
}
```

逐步分析：

```IjkMediaPlayer *mp = ijkmp_android_create(message_loop);``` 创建一个 IjkMediaPlayer 的 Android 下的实例。[方法详解](ijkmp_android_create.md)

```
JNI_CHECK_GOTO(mp, env, "java/lang/OutOfMemoryError", "mpjni: native_setup: ijkmp_create() failed", LABEL_RETURN);
```

判空。

```jni_set_media_player(env, thiz, mp);``` 存储 mp 的引用。[方法详解](jni_set_media_player.md)

```ijkmp_set_weak_thiz(mp, (*env)->NewGlobalRef(env, weak_this));``` 让 IjkMediaPlayer 得到 this 的引用。[方法详解](ijkmp_set_weak_thiz.md)

```ijkmp_set_inject_opaque(mp, ijkmp_get_weak_thiz(mp));``` 注入 opaque。[方法详解](ijkmp_set_inject_opaque.md)

```ijkmp_set_ijkio_inject_opaque(mp, ijkmp_get_weak_thiz(mp));``` 注入 ijkio 的 opaque。[方法详解](ijkmp_set_ijkio_inject_opaque.md)

```
ijkmp_android_set_mediacodec_select_callback(mp, 
        mediacodec_select_callback, ijkmp_get_weak_thiz(mp));
```

设置 media codec 选择之后的回调。[方法详解](ijkmp_android_set_mediacodec_select_callback.md)

```ijkmp_dec_ref_p(&mp);``` 最后释放 mp 的引用。

```mediacodec_select_callback``` 的实现是：

```
static bool mediacodec_select_callback(void *opaque, ijkmp_mediacodecinfo_context *mcc)
{
    JNIEnv *env = NULL;
    jobject weak_this = (jobject) opaque;
    const char *found_codec_name = NULL;

    if (JNI_OK != SDL_JNI_SetupThreadEnv(&env)) {
        ALOGE("%s: SetupThreadEnv failed\n", __func__);
        return -1;
    }

    found_codec_name = J4AC_IjkMediaPlayer__onSelectCodec__withCString__asCBuffer(env, weak_this, mcc->mime_type, mcc->profile, mcc->level, mcc->codec_name, sizeof(mcc->codec_name));
    if (J4A_ExceptionCheck__catchAll(env) || !found_codec_name) {
        ALOGE("%s: onSelectCodec failed\n", __func__);
        goto fail;
    }

fail:
    return found_codec_name;
}
```

会在回调中返回 codec 的名称。


## 播放

### setDataSource

通过 ijkmedia/ijkplayer/android/ijkplayer_jni.c 的 g_method 表，可以看到 setDataSource 对应几个 native 方法。

```
static JNINativeMethod g_methods[] = {
    {
        "_setDataSource",
        "(Ljava/lang/String;[Ljava/lang/String;[Ljava/lang/String;)V",
        (void *) IjkMediaPlayer_setDataSourceAndHeaders
    },
    { "_setDataSourceFd",       "(I)V",     (void *) IjkMediaPlayer_setDataSourceFd },
    { "_setDataSource",         "(Ltv/danmaku/ijk/media/player/misc/IMediaDataSource;)V", (void *)IjkMediaPlayer_setDataSourceCallback },
    ...
}
```

先看第三个方法，```IjkMediaPlayer_setDataSourceCallback```，因为在 Example 中，封装的 setDataSource，包含 url 和 path，最后指向的都是这个方法。

前面的两个方法的基本流程都是一样的：

```
static void IjkMediaPlayer...setDataSource...(JNIEnv *env, jobject thiz, jstring path, ...)
{
    ...
    IjkMediaPlayer *mp = jni_get_media_player(env, thiz);
    ...
    retval = ijkmp_set_data_source(mp, c_path);
    ...
    ijkmp_dec_ref_p(&mp);
}
```

下面再详细分析第三个方法。

```IjkMediaPlayer_setDataSourceCallback``` 实现在 ijkmedia/ijkplayer/android/ijkplayer_jni.c :

```
static void
IjkMediaPlayer_setDataSourceCallback(JNIEnv *env, jobject thiz, jobject callback)
{
    MPTRACE("%s\n", __func__);
    int retval = 0;
    char uri[128];
    int64_t nativeMediaDataSource = 0;
    IjkMediaPlayer *mp = jni_get_media_player(env, thiz);
    JNI_CHECK_GOTO(callback, env, "java/lang/IllegalArgumentException", "mpjni: setDataSourceCallback: null fd", LABEL_RETURN);
    JNI_CHECK_GOTO(mp, env, "java/lang/IllegalStateException", "mpjni: setDataSourceCallback: null mp", LABEL_RETURN);

    nativeMediaDataSource = jni_set_media_data_source(env, thiz, callback);
    JNI_CHECK_GOTO(nativeMediaDataSource, env, "java/lang/IllegalStateException", "mpjni: jni_set_media_data_source: NewGlobalRef", LABEL_RETURN);

    ALOGV("setDataSourceCallback: %"PRId64"\n", nativeMediaDataSource);
    snprintf(uri, sizeof(uri), "ijkmediadatasource:%"PRId64, nativeMediaDataSource);

    retval = ijkmp_set_data_source(mp, uri);

    IJK_CHECK_MPRET_GOTO(retval, env, LABEL_RETURN);

LABEL_RETURN:
    ijkmp_dec_ref_p(&mp);
}
```

依然逐步分析：

```IjkMediaPlayer *mp = jni_get_media_player(env, thiz);```，获得 IjkMediaPlayer 的引用，[方法详解](jni_get_media_player.md)。

```JNI_CHECK_GOTO(condition__, env__, exception__, msg__, label__)```，判空检查，若为空，则抛出异常，跳转到 label__ 处，[方法详解](JNI_CHECK_GOTO.md)。

```nativeMediaDataSource = jni_set_media_data_source(env, thiz, callback);```，设置视频的 MediaDataSource。[方法详解](jni_set_media_data_source.md)。

```retval = ijkmp_set_data_source(mp, uri);```，[方法详解](ijkmp_set_data_source.md)。

```IJK_CHECK_MPRET_GOTO(retval, env, LABEL_RETURN);``` 检查返回值，判断 set_data_source  是否成功。

```ijkmp_dec_ref_p(&mp);``` 对应释放对 mp 的引用。[方法详解](ijkmp_dec_ref_p.md)。


### 准备同步

通过 ijkmedia/ijkplayer/android/ijkplayer_jni.c 的 g_method 表，可以看到 prepareAsync 对应几个 native 方法：

```
static JNINativeMethod g_methods[] = {
    ...
    { "_prepareAsync", "()V", (void *) IjkMediaPlayer_prepareAsync },
    ...
}
```

实现：

```
static void
IjkMediaPlayer_prepareAsync(JNIEnv *env, jobject thiz)
{
    MPTRACE("%s\n", __func__);
    int retval = 0;
    IjkMediaPlayer *mp = jni_get_media_player(env, thiz);
    JNI_CHECK_GOTO(mp, env, "java/lang/IllegalStateException", "mpjni: prepareAsync: null mp", LABEL_RETURN);

    retval = ijkmp_prepare_async(mp);
    IJK_CHECK_MPRET_GOTO(retval, env, LABEL_RETURN);

LABEL_RETURN:
    ijkmp_dec_ref_p(&mp);
}
```

方法 ```int jni_prepare_async(IjkMediaPlayer *mp)``` 声明和实现都在 ijkmedia/ijkplayer/ijkplayer.c 中：

```
int ijkmp_prepare_async(IjkMediaPlayer *mp)
{
    assert(mp);
    MPTRACE("ijkmp_prepare_async()\n");
    pthread_mutex_lock(&mp->mutex);
    int retval = ijkmp_prepare_async_l(mp);
    pthread_mutex_unlock(&mp->mutex);
    MPTRACE("ijkmp_prepare_async()=%d\n", retval);
    return retval;
}

static int ijkmp_prepare_async_l(IjkMediaPlayer *mp)
{
    assert(mp);

    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_IDLE);
    // MPST_RET_IF_EQ(mp->mp_state, MP_STATE_INITIALIZED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_ASYNC_PREPARING);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_PREPARED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_STARTED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_PAUSED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_COMPLETED);
    // MPST_RET_IF_EQ(mp->mp_state, MP_STATE_STOPPED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_ERROR);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_END);

    assert(mp->data_source);

    ijkmp_change_state_l(mp, MP_STATE_ASYNC_PREPARING);

    msg_queue_start(&mp->ffplayer->msg_queue);

    // released in msg_loop
    ijkmp_inc_ref(mp);
    mp->msg_thread = SDL_CreateThreadEx(&mp->_msg_thread, ijkmp_msg_loop, mp, "ff_msg_loop");
    // msg_thread is detached inside msg_loop
    // TODO: 9 release weak_thiz if pthread_create() failed;

    int retval = ffp_prepare_async_l(mp->ffplayer, mp->data_source);
    if (retval < 0) {
        ijkmp_change_state_l(mp, MP_STATE_ERROR);
        return retval;
    }

    return 0;
}
```

依然逐步分析：

```
MPST_RET_IF_EQ(mp->mp_state, MP_STATE_IDLE);
// MPST_RET_IF_EQ(mp->mp_state, MP_STATE_INITIALIZED);
...
// MPST_RET_IF_EQ(mp->mp_state, MP_STATE_STOPPED);
MPST_RET_IF_EQ(mp->mp_state, MP_STATE_ERROR);
MPST_RET_IF_EQ(mp->mp_state, MP_STATE_END);
```

第一部分是用于状态机的检查，只有在 ```MP_STATE_INITIALIZED``` 和 ```MP_STATE_STOPPED``` 这两种状态时才能够执行 prepare\_async。

```
ijkmp_change_state_l(mp, MP_STATE_ASYNC_PREPARING);
```

这里是改变状态为 ```MP_STATE_ASYNC_PREPAREING```。[方法详解](ijkmp_change_state_l.md)

```
msg_queue_start(&mp->ffplayer->msg_queue);
```

在上面的[方法详解](ijkmp_change_state_l.md)中已经介绍过这个方法了。这个会激活 mp->ffplayer->msg_queue，并在 msg_queue 中加入一个类型为 ```FFP_MSG_FLUSH``` 的 Message。

```
mp->msg_thread = SDL_CreateThreadEx(&mp->_msg_thread, ijkmp_msg_loop, mp, "ff_msg_loop");
```

用于创建一个新的 Thread。[方法详解](SDL_CreateThreadEx.md)

```
int retval = ffp_prepare_async_l(mp->ffplayer, mp->data_source);
```

这里是真正实现 prepare\_async 逻辑的方法，在 ijkmedia/ijkplayer/ff_ffplay.c 中：

```
int ffp_prepare_async_l(FFPlayer *ffp, const char *file_name)
{
    assert(ffp);
    assert(!ffp->is);
    assert(file_name);

    if (av_stristart(file_name, "rtmp", NULL) ||
        av_stristart(file_name, "rtsp", NULL)) {
        // There is total different meaning for 'timeout' option in rtmp
        av_log(ffp, AV_LOG_WARNING, "remove 'timeout' option for rtmp.\n");
        av_dict_set(&ffp->format_opts, "timeout", NULL, 0);
    }

    /* there is a length limit in avformat */
    if (strlen(file_name) + 1 > 1024) {
        av_log(ffp, AV_LOG_ERROR, "%s too long url\n", __func__);
        if (avio_find_protocol_name("ijklongurl:")) {
            av_dict_set(&ffp->format_opts, "ijklongurl-url", file_name, 0);
            file_name = "ijklongurl:";
        }
    }

    av_log(NULL, AV_LOG_INFO, "===== versions =====\n");
    ffp_show_version_str(ffp, "ijkplayer",      ijk_version_info());
    ffp_show_version_str(ffp, "FFmpeg",         av_version_info());
    ffp_show_version_int(ffp, "libavutil",      avutil_version());
    ffp_show_version_int(ffp, "libavcodec",     avcodec_version());
    ffp_show_version_int(ffp, "libavformat",    avformat_version());
    ffp_show_version_int(ffp, "libswscale",     swscale_version());
    ffp_show_version_int(ffp, "libswresample",  swresample_version());
    av_log(NULL, AV_LOG_INFO, "===== options =====\n");
    ffp_show_dict(ffp, "player-opts", ffp->player_opts);
    ffp_show_dict(ffp, "format-opts", ffp->format_opts);
    ffp_show_dict(ffp, "codec-opts ", ffp->codec_opts);
    ffp_show_dict(ffp, "sws-opts   ", ffp->sws_dict);
    ffp_show_dict(ffp, "swr-opts   ", ffp->swr_opts);
    av_log(NULL, AV_LOG_INFO, "===================\n");

    av_opt_set_dict(ffp, &ffp->player_opts);
    if (!ffp->aout) {
        ffp->aout = ffpipeline_open_audio_output(ffp->pipeline, ffp);
        if (!ffp->aout)
            return -1;
    }

#if CONFIG_AVFILTER
    if (ffp->vfilter0) {
        GROW_ARRAY(ffp->vfilters_list, ffp->nb_vfilters);
        ffp->vfilters_list[ffp->nb_vfilters - 1] = ffp->vfilter0;
    }
#endif

    VideoState *is = stream_open(ffp, file_name, NULL);
    if (!is) {
        av_log(NULL, AV_LOG_WARNING, "ffp_prepare_async_l: stream_open failed OOM");
        return EIJK_OUT_OF_MEMORY;
    }

    ffp->is = is;
    ffp->input_filename = av_strdup(file_name);
    return 0;
}
```

逐步分析：

```
if (av_stristart(file_name, "rtmp", NULL) ||
        av_stristart(file_name, "rtsp", NULL)) {
    ...
}
```

av_stristart 的声明在 android/contrib/ffmpeg-arm64/libavutil/avstring.h 中：

```
/**
 * Return non-zero if pfx is a prefix of str independent of case. If
 * it is, *ptr is set to the address of the first character in str
 * after the prefix.
 *
 * @param str input string
 * @param pfx prefix to test
 * @param ptr updated if the prefix is matched inside str
 * @return non-zero if the prefix matches, zero otherwise
 */
int av_stristart(const char *str, const char *pfx, const char **ptr);
```

判断一个字符串的前缀是否是给定的字符串。在此处就是判断是 rtmp 或 rtsp。

```
// There is total different meaning for 'timeout' option in rtmp
av_log(ffp, AV_LOG_WARNING, "remove 'timeout' option for rtmp.\n");
av_dict_set(&ffp->format_opts, "timeout", NULL, 0);
```

通过注释可知，timeout 的选项对 rtmp 来说意义各别的不一样，所以就设置 timeout 项为 NULL。[方法详解](av_dict_set.md)

