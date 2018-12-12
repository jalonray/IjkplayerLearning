# J4AC\_IjkMediaPlayer\_\_mNativeMediaDataSource\_\_set\_\_catchAll

方法定义在 ijkmedia/ijkj4a/j4a/class/tv/danmaku/ijk/media/player/IjkMediaPlayer.h 中：

```
void J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll(JNIEnv *env, jobject thiz, jlong value);

#define J4AC_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll
```

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

void J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaDataSource__set(JNIEnv *env, jobject thiz, jlong value)
{
    (*env)->SetLongField(env, thiz, class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer.field_mNativeMediaDataSource, value);
}

void J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll(JNIEnv *env, jobject thiz, jlong value)
{
    J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaDataSource__set(env, thiz, value);
    J4A_ExceptionCheck__catchAll(env);
}
```

设置 MedataDataSource 的 jFiledId。

再看 ```bool J4A_ExceptionCheck__catchAll(JNIEnv *env);``` 方法，定义在 ijkmedia/ijkj4a/j4a/j4a_base.h 中：

```
bool J4A_ExceptionCheck__catchAll(JNIEnv *env);
```

实现在 ijkmedia/ijkj4a/j4a/j4a_base.c 中：

```
bool J4A_ExceptionCheck__catchAll(JNIEnv *env)
{
    if ((*env)->ExceptionCheck(env)) {
        (*env)->ExceptionDescribe(env);
        (*env)->ExceptionClear(env);
        return true;
    }

    return false;
}
```

检查 env 中有无 Exception。

[返回](jni_set_media_data_source.md)