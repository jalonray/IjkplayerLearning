# jni\_set\_media\_data\_source

方法实现在 ijkmedia/ijkplayer/android/ijkplayer_jni.c 中：

```
static int64_t jni_set_media_data_source(JNIEnv* env, jobject thiz, jobject             media_data_source)
{
    int64_t nativeMediaDataSource = 0;

    pthread_mutex_lock(&g_clazz.mutex);

    jobject old = (jobject) (intptr_t)                                                  J4AC_IjkMediaPlayer__mNativeMediaDataSource__get__catchAll(env, thiz);
    if (old) {
        J4AC_IMediaDataSource__close__catchAll(env, old);
        J4A_DeleteGlobalRef__p(env, &old);
        J4AC_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll(env, thiz, 0);
    }

    if (media_data_source) {
        jobject global_media_data_source = (*env)->NewGlobalRef(env,                    media_data_source);
        if (J4A_ExceptionCheck__catchAll(env) || !global_media_data_source)
            goto fail;

        nativeMediaDataSource = (int64_t) (intptr_t) global_media_data_source;
        J4AC_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll(env, thiz, (jlong)   nativeMediaDataSource);
    }

fail:
    pthread_mutex_unlock(&g_clazz.mutex);
    return nativeMediaDataSource;
}
```

```J4AC_IjkMediaPlayer__mNativeMediaDataSource__get__catchAll(env, thiz)```，获取原有的 MedaiDataSource，[方法详解](J4AC_IjkMediaPlayer__mNativeMediaDataSource__get__catchAll.md)

```
if (old) {
    J4AC_IMediaDataSource__close__catchAll(env, old);
    J4A_DeleteGlobalRef__p(env, &old);
    J4AC_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll(env, thiz, 0);
}
```
如果 old 不为空，先 ```J4AC_IMediaDataSource__close__catchAll(env, old);``` 关闭之前的 MediaDataSource，[方法详情](J4AC_IMediaDataSource__close__catchAll.md)。之后再删除全局引用 ```J4A_DeleteGlobalRef__p(env, &old);```，[方法详情](J4A_DeleteGlobalRef__p.md)。最后再设置 MediaDataSource 为空 ```J4AC_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll(env, thiz, 0);```，[方法详情](J4AC_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll.md)。

```
if (media_data_source) {
        jobject global_media_data_source = (*env)->NewGlobalRef(env,                    media_data_source);
        if (J4A_ExceptionCheck__catchAll(env) || !global_media_data_source)
            goto fail;

        nativeMediaDataSource = (int64_t) (intptr_t) global_media_data_source;
        J4AC_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll(env, thiz, (jlong)   nativeMediaDataSource);
    }
```

先 ```if (media_data_source) {``` 判空，再 ```jobject global_media_data_source = (*env)->NewGlobalRef(env, media_data_source);``` 获取全局引用，之后：

```
if (J4A_ExceptionCheck__catchAll(env) || !global_media_data_source)
            goto fail;
```

Exception 检查和引用的判空，若通不过检查则跳转失败 label。

```nativeMediaDataSource = (int64_t) (intptr_t) global_media_data_source;``` 得到 DataSource 的引用。```J4AC_IjkMediaPlayer__mNativeMediaDataSource__set__catchAll(env, thiz, (jlong)   nativeMediaDataSource);``` 设置 DataSource。

```
fail:
    pthread_mutex_unlock(&g_clazz.mutex);
    return nativeMediaDataSource;
```

释放锁，返回引用地址。

[返回](ijkplayer_main.md)