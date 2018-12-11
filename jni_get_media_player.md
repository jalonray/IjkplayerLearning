# jni\_get\_media\_player

方法实现在 ijkmedia/ijkplayer/android/ijkplayer_jni.c :

```
static IjkMediaPlayer *jni_get_media_player(JNIEnv* env, jobject thiz)
{
    pthread_mutex_lock(&g_clazz.mutex);

    IjkMediaPlayer *mp = (IjkMediaPlayer *) (intptr_t) J4AC_IjkMediaPlayer__mNativeMediaPlayer__get__catchAll(env, thiz);
    if (mp) {
        ijkmp_inc_ref(mp);
    }

    pthread_mutex_unlock(&g_clazz.mutex);
    return mp;
}
```

```pthread_mutex_lock(&g_clazz.mutex);```，加锁。

方法 ```J4AC_IjkMediaPlayer__mNativeMediaPlayer__get__catchAll``` 定义在 ijkmedia/ijkj4a/j4a/class/tv/danmaku/ijk/media/player/IjkMediaPlayer.c 中：

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

jlong J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaPlayer__get(JNIEnv *env, jobject thiz)
{
    return (*env)->GetLongField(env, thiz, class_J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer.field_mNativeMediaPlayer);
}

jlong J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaPlayer__get__catchAll(JNIEnv *env, jobject thiz)
{
    jlong ret_value = J4AC_tv_danmaku_ijk_media_player_IjkMediaPlayer__mNativeMediaPlayer__get(env, thiz);
    if (J4A_ExceptionCheck__catchAll(env)) {
        return 0;
    }

    return ret_value;
}
```

```J4AC_IjkMediaPlayer__mNativeMediaPlayer__get__catchAll``` 方法会获得 IjkMediaPlayer 在内存中的地址。之后 ```IjkMediaPlayer *mp = (IjkMediaPlayer *) (intptr_t)...``` 通过类型强转得到引用。之后

```
if (mp) {
    ijkmp_inc_ref(mp);
}
```

方法 ```ijkmp_inc_ref()``` 的实现在 ijkmedia/ijkplayer/ijkplayer.c 中：

```
void ijkmp_inc_ref(IjkMediaPlayer *mp)
{
    assert(mp);
    __sync_fetch_and_add(&mp->ref_count, 1);
}
```

```type __sync_fetch_and_add(type *ptr, type value, ...);``` 是 GCC 的标准库函数，[详情](https://gcc.gnu.org/onlinedocs/gcc-4.1.0/gcc/Atomic-Builtins.html)

用于增加引用次数。

```pthread_mutex_unlock(&g_clazz.mutex);``` 解锁。

[返回](ijkplayer_main.md)
