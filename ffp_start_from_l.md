# ffp\_start\_from\_l

方法声明在 ijkmedia/ijkplayer/ff_ffplay.h 中：

```
/* playback controll */
...
int ffp_start_from_l(FFPlayer *ffp, long msec);
///
```

实现在 ijkmedia/ijkplayer/ff_ffplay.c 中：

```
int ffp_start_from_l(FFPlayer *ffp, long msec)
{
    assert(ffp);
    VideoState *is = ffp->is;
    if (!is)
        return EIJK_NULL_IS_PTR;

    ffp->auto_resume = 1;
    ffp_toggle_buffering(ffp, 1);
    ffp_seek_to_l(ffp, msec);
    return 0;
}
```

开始判空检查不赘述。

```
ffp->auto_resume = 1;
```

标记自动继续为 true。

```
ffp_toggle_buffering(ffp, 1);
```

触发开始加载缓存。[方法详解](ffp_toggle_buffering.md)

```
ffp_seek_to_l(ffp, msec);
```

跳转到要开始播放的时间点。[方法详解](ffp_seek_to_l.md)

[返回](ijkplayer_main.md)