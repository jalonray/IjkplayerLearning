# ijkmp\_android\_create

方法声明在 ijkmedia/ijkplayer/android/ijkplayer_android.h 中：

```
// ref_count is 1 after open
IjkMediaPlayer *ijkmp_android_create(int(*msg_loop)(void*));
```

实现在 ijkmedia/ijkplayer/android/ijkplayer_android.c 中：

```
IjkMediaPlayer *ijkmp_android_create(int(*msg_loop)(void*))
{
    IjkMediaPlayer *mp = ijkmp_create(msg_loop);
    if (!mp)
        goto fail;

    mp->ffplayer->vout = SDL_VoutAndroid_CreateForAndroidSurface();
    if (!mp->ffplayer->vout)
        goto fail;

    mp->ffplayer->pipeline = ffpipeline_create_from_android(mp->ffplayer);
    if (!mp->ffplayer->pipeline)
        goto fail;

    ffpipeline_set_vout(mp->ffplayer->pipeline, mp->ffplayer->vout);

    return mp;

fail:
    ijkmp_dec_ref_p(&mp);
    return NULL;
}
```

```IjkMediaPlayer *mp = ijkmp_create(msg_loop);``` 是通过一个 msg\_loop 去创建一个 IjkMediaPlayer 的实例，msg\_loop 是一个状态机，用于更新当前播放器的状态。[方法详解](ijkmp_create.md)

```mp->ffplayer->vout = SDL_VoutAndroid_CreateForAndroidSurface();``` 设置 IjkMediaPlayer 中 FFPlayer 的 Video Out 指向为一个新创建的 Android Surface。[方法详解](SDL_VoutAndroid_CreateForAndroidSurface.md)

```mp->ffplayer->pipeline = ffpipeline_create_from_android(mp->ffplayer);``` 设置 IjkMediaPlayer 中 FFPlayer 的 pipeline 指向为一个新创建的 Android pipeline。[方法详解](ffpipeline_create_from_android.md)

```ffpipeline_set_vout(mp->ffplayer->pipeline, mp->ffplayer->vout);``` 将 FFPlayer 的 pipeline 和 vout 绑定。[方法详解](ffpipeline_set_vout.md)

```ijkmp_dec_ref_p(&mp);``` 错误的情况下，对 mp 的引用减 1。

[返回](ijkplayer_main.md)