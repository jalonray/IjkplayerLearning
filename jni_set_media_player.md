# jni\_set\_media\_player

方法声明和实现都在 ijkmedia/ijkplayer/android/ijkplayer_jni.c 中：

```
static IjkMediaPlayer *jni_set_media_player(JNIEnv* env, jobject thiz, IjkMediaPlayer *mp)
{
    pthread_mutex_lock(&g_clazz.mutex);

    IjkMediaPlayer *old = (IjkMediaPlayer*) (intptr_t) J4AC_IjkMediaPlayer__mNativeMediaPlayer__get__catchAll(env, thiz);
    if (mp) {
        ijkmp_inc_ref(mp);
    }
    J4AC_IjkMediaPlayer__mNativeMediaPlayer__set__catchAll(env, thiz, (intptr_t) mp);

    pthread_mutex_unlock(&g_clazz.mutex);

    // NOTE: ijkmp_dec_ref may block thread
    if (old != NULL ) {
        ijkmp_dec_ref_p(&old);
    }

    return old;
}
```

```pthread_mutex_lock(&g_clazz.mutex);``` 加锁；

```
IjkMediaPlayer *old = (IjkMediaPlayer*) (intptr_t) J4AC_IjkMediaPlayer__mNativeMediaPlayer__get__catchAll(env, thiz);
```

获取旧的 Native Player。[方法详解](J4AC_IjkMediaPlayer__mNativeMediaPlayer__get__catchAll.md)

```
if (mp) {
    ijkmp_inc_ref(mp);
}
```

若传入的 IjkMediaPlayer 不为空，则为记录引用增加。

```pthread_mutex_unlock(&g_clazz.mutex);``` 释放锁。

```
// NOTE: ijkmp_dec_ref may block thread
if (old != NULL ) {
    ijkmp_dec_ref_p(&old);
}
```

若旧的 Player 不为空，就是已经有播放器了，则释放对旧有播放器的引用。

```return old;``` 返回旧播放器的引用。

[返回](ijkplayer_main.md)