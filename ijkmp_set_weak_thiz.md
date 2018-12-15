# ijkmp\_set\_weak\_thiz

方法声明在 ijkmedia/ijkplayer/ijkplayer.h 中：

```
void *ijkmp_set_weak_thiz(IjkMediaPlayer *mp, void *weak_thiz);
```

方法实现在 ijkmedia/ijkplayer/ijkplayer.c 中：

```
void *ijkmp_set_weak_thiz(IjkMediaPlayer *mp, void *weak_thiz)
{
    void *prev_weak_thiz = mp->weak_thiz;

    mp->weak_thiz = weak_thiz;

    return prev_weak_thiz;
}
```

将 IjkMediaPlayer 的 weak\_this 赋为新的引用，并返回旧的。

[返回](ijkplayer_main.md)