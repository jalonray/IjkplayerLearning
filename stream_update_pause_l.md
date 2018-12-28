# stream\_update\_pause\_l

方法声明和定义都在 ijkmedia/ijkplayer/ff_ffplay.c 中：

```
static void stream_update_pause_l(FFPlayer *ffp)
{
    VideoState *is = ffp->is;
    if (!is->step && (is->pause_req || is->buffering_on)) {
        stream_toggle_pause_l(ffp, 1);
    } else {
        stream_toggle_pause_l(ffp, 0);
    }
}

/* pause or resume the video */
static void stream_toggle_pause_l(FFPlayer *ffp, int pause_on)
{
    VideoState *is = ffp->is;
    if (is->paused && !pause_on) {
        is->frame_timer += av_gettime_relative() / 1000000.0 - is->vidclk.last_updated;

#ifdef FFP_MERGE
        if (is->read_pause_return != AVERROR(ENOSYS)) {
            is->vidclk.paused = 0;
        }
#endif
        set_clock(&is->vidclk, get_clock(&is->vidclk), is->vidclk.serial);
        set_clock(&is->audclk, get_clock(&is->audclk), is->audclk.serial);
    } else {
    }
    set_clock(&is->extclk, get_clock(&is->extclk), is->extclk.serial);
    if (is->step && (is->pause_req || is->buffering_on)) {
        is->paused = is->vidclk.paused = is->extclk.paused = pause_on;
    } else {
        is->paused = is->audclk.paused = is->vidclk.paused = is->extclk.paused = pause_on;
        SDL_AoutPauseAudio(ffp->aout, pause_on);
    }
}
```

逐步分析：

```
if (!is->step && (is->pause_req || is->buffering_on)) {
	stream_toggle_pause_l(ffp, 1);
}
```

若 ```step``` 为 false，并且有暂停请求或是已经缓存时，触发暂停请求。

```
else {
    stream_toggle_pause_l(ffp, 0);
}
```

若其他的情况，则触发取消暂停请求。

下面分析 

```
/* pause or resume the video */
static void stream_toggle_pause_l(FFPlayer *ffp, int pause_on)
```

```
if (is->paused && !pause_on) {
	...
}
```

若视频已经暂停了，并且要请求继续播放时的逻辑。

```
is->frame_timer += av_gettime_relative() / 1000000.0 - is->vidclk.last_updated;
```

先设置 ```frame_timer``` 为 ```vidclk.last_updated``` 和 ```av_gettime_relative()``` 之和。




[返回](ffp_toggle_buffering.md)