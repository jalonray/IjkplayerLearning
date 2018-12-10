# ijkPlayer Android 源码研究

## 初始化

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

### 第四句```ijkmp_global_init();```

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

### 第五句 ```ijkmp_global_set_inject_callback(inject_callback);```


