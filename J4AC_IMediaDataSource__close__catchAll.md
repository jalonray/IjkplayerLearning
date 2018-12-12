# J4AC\_IMediaDataSource\_\_close\_\_catchAll

方法定义在 ijkmedia/ijkj4a/j4a/class/tv/danmaku/ijk/media/player/misc/IMediaDataSource.h 中：

```
#define J4AC_IMediaDataSource__close__catchAll J4AC_tv_danmaku_ijk_media_player_misc_IMediaDataSource__close__catchAll
```

实现在对应的 ijkmedia/ijkj4a/j4a/class/tv/danmaku/ijk/media/player/misc/IMediaDataSource.c 中：

```
typedef struct J4AC_tv_danmaku_ijk_media_player_misc_IMediaDataSource {
    jclass id;

    jmethodID method_readAt;
    jmethodID method_getSize;
    jmethodID method_close;
} J4AC_tv_danmaku_ijk_media_player_misc_IMediaDataSource;
static J4AC_tv_danmaku_ijk_media_player_misc_IMediaDataSource class_J4AC_tv_danmaku_ijk_media_player_misc_IMediaDataSource;
 
void J4AC_tv_danmaku_ijk_media_player_misc_IMediaDataSource__close(JNIEnv *env, jobject thiz)
{
    (*env)->CallVoidMethod(env, thiz, class_J4AC_tv_danmaku_ijk_media_player_misc_IMediaDataSource.method_close);
}

void J4AC_tv_danmaku_ijk_media_player_misc_IMediaDataSource__close__catchAll(JNIEnv *env, jobject thiz)
{
    J4AC_tv_danmaku_ijk_media_player_misc_IMediaDataSource__close(env, thiz);
    J4A_ExceptionCheck__catchAll(env);
}
```

调用对应的 close 方法。

[返回](jni_set_media_data_source.md)