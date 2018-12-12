# J4AC\_IjkMediaPlayer\_\_mNativeMediaDataSource\_\_get\_\_catchAll

在 ijkmedia/ijkj4a/j4a/class/tv/danmaku/ijk/media/player/IjkMediaPlayer.h 中有下述两定义。

```jlong J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaDataSource__get__catchAll(JNIEnv *env, jobject thiz);``` 

```#define J4AC_IjkMediaPlayer__mNativeMediaDataSource__get__catchAll J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaDataSource__get__catchAll```

实现在 ijkmedia/ijkj4a/j4a/class/tv/danmaku/ijk/media/player/IjkMediaPlayer.c 中：

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

jlong J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaDataSource__get(JNIEnv *env, jobject thiz)
{
    return (*env)->GetLongField(env, thiz, class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer.field_mNativeMediaDataSource);
}
 
jlong J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaDataSource__get__catchAll(JNIEnv *env, jobject thiz)
{
    jlong ret_value = J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaDataSource__get(env, thiz);
    if (J4A_ExceptionCheck__catchAll(env)) {
        return 0;
    }

    return ret_value;
}
```

获取 field_mNativeMeiaDataSource 的 FieldId。

[返回](jni_set_media_data_source.md)