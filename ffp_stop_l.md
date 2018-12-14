# ffp\_stop\_l

方法声明在 ijkmedia/ijkplayer/ff_ffplay.h 中：

```
/* playback controll */
int ffp_stop_l(FFPlayer *ffp);
```

实现在 ijkmedia/ijkplayer/ff_ffplay.c 中：

```
int ffp_stop_l(FFPlayer *ffp)
{
    assert(ffp);
    VideoState *is = ffp->is;
    if (is) {
        is->abort_request = 1;
        toggle_pause(ffp, 1);
    }

    msg_queue_abort(&ffp->msg_queue);
    if (ffp->enable_accurate_seek && is && is->accurate_seek_mutex
        && is->audio_accurate_seek_cond && is->video_accurate_seek_cond) {
        SDL_LockMutex(is->accurate_seek_mutex);
        is->audio_accurate_seek_req = 0;
        is->video_accurate_seek_req = 0;
        SDL_CondSignal(is->audio_accurate_seek_cond);
        SDL_CondSignal(is->video_accurate_seek_cond);
        SDL_UnlockMutex(is->accurate_seek_mutex);
    }
    return 0;
}

static void toggle_pause(FFPlayer *ffp, int pause_on)
{
    SDL_LockMutex(ffp->is->play_mutex);
    toggle_pause_l(ffp, pause_on);
    SDL_UnlockMutex(ffp->is->play_mutex);
}
```

逐步分析

```VideoState *is = ffp->is;``` 获得 ffp 当前的状态；

```
if (is) {
    is->abort_request = 1;
    toggle_pause(ffp, 1);
}
```

判断 is 是否为空，不为空则置为 1，然后触发 pause。

下面分析 ```toggle_pause(ffp, 1)```：

```SDL_LockMutex(ffp->is->play_mutex);``` 和 ```SDL_UnlockMutex(ffp->is->play_mutex);```，定义和实现都在 ijkplayer/ijkmedia/ijksdl/ijksdl_mutex.c 中：

```
int SDL_LockMutex(SDL_mutex *mutex)
{
    assert(mutex);
    if (!mutex)
        return -1;

    return pthread_mutex_lock(&mutex->id);
}

int SDL_UnlockMutex(SDL_mutex *mutex)
{
    assert(mutex);
    if (!mutex)
        return -1;

    return pthread_mutex_unlock(&mutex->id);
}
```

加锁和释放锁。

```toggle_pause_l(ffp, pause_on);```，真正实现 pause 的地方：

```
static void toggle_pause_l(FFPlayer *ffp, int pause_on)
{
    VideoState *is = ffp->is;
    if (is->pause_req && !pause_on) {
        set_clock(&is->vidclk, get_clock(&is->vidclk), is->vidclk.serial);
        set_clock(&is->audclk, get_clock(&is->audclk), is->audclk.serial);
    }
    is->pause_req = pause_on;
    ffp->auto_resume = !pause_on;
    stream_update_pause_l(ffp);
    is->step = 0;
}
```

```VideoState *is = ffp->is;``` 获得 ffp 当前的状态；

```
if (is->pause_req && !pause_on) {
    set_clock(&is->vidclk, get_clock(&is->vidclk), is->vidclk.serial);
    set_clock(&is->audclk, get_clock(&is->audclk), is->audclk.serial);
}
```

若当前正在处于 pause 的请求当中，并请求不是 pause_on（1）时，设置时钟：

```
static double get_clock(Clock *c)
{
    if (*c->queue_serial != c->serial)
        return NAN;
    if (c->paused) {
        return c->pts;
    } else {
        double time = av_gettime_relative() / 1000000.0;
        return c->pts_drift + time - (time - c->last_updated) * (1.0 - c->speed);
    }
}

static void set_clock_at(Clock *c, double pts, int serial, double time)
{
    c->pts = pts;
    c->last_updated = time;
    c->pts_drift = c->pts - time;
    c->serial = serial;
}

static void set_clock(Clock *c, double pts, int serial)
{
    double time = av_gettime_relative() / 1000000.0;
    set_clock_at(c, pts, serial, time);
}
```

```av_gettime_relative()``` 获取相对时间，[方法详解](av_gettime_relative.md)。

看下 Clock 的声明：

```
typedef struct Clock {
    double pts;           /* clock base */
    double pts_drift;     /* clock base minus time at which we updated the clock */
    double last_updated;
    double speed;
    int serial;           /* clock is based on a packet with this serial */
    int paused;
    int *queue_serial;    /* pointer to the current packet queue serial, used for obsolete clock detection */
} Clock;
```

是一个同步时钟。不过在我们现在讨论的 pause 方法中并不会走到这里，接着讨论后面：

```is->pause_req = pause_on;``` 设置 pause_req 为 pause 值，这里为 true，这是一个“请求暂停”的 flag。```ffp->auto_resume = !pause_on;``` 设置 auto_resume 为 pause 的反值，这里是 false，```stream_update_pause_l(ffp);``` 更新 stream 的状态为 pause。

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
```

```if (!is->step && (is->pause_req || is->buffering_on))``` step 为 false 并 请求了 pause 或是正在 buffer 时，```stream_toggle_pause_l(ffp, 1);``` 设 stream 的暂停为 true，否则 ```stream_toggle_pause_l(ffp, 0);``` 设 stream 的暂停为 false，也就是 resume。

实现如下：

```
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

```if (is->paused && !pause_on) {``` 若已经暂停了；```is->frame_timer += av_gettime_relative() / 1000000.0 - is->vidclk.last_updated;``` 设一帧 frame_timer 的时间为现在至上次更新的时间；```if (is->read_pause_return != AVERROR(ENOSYS)) {``` 若有错误，```is->vidclk.paused = 0;``` pause 置为 false；```set_clock(&is->vidclk, get_clock(&is->vidclk), is->vidclk.serial);``` 和 ```set_clock(&is->audclk, get_clock(&is->audclk), is->audclk.serial);``` 对对应的 serial 更新 video 和 audio clock；到这对应 ```if (is->paused && !pause_on) {``` 的部分，下面是通用逻辑，```set_clock(&is->extclk, get_clock(&is->extclk), is->extclk.serial);``` 更新绝对 clock 对应 serial 的 clock；```if (is->step && (is->pause_req || is->buffering_on)) {``` 若 step 为 true，并请求了 pause 或是正在 buffer，则 ```is->paused = is->vidclk.paused = is->extclk.paused = pause_on;``` 所有的 clock 设置为 pause；否则在设置所有的 clock 设置为 pause 之后，```SDL_AoutPauseAudio(ffp->aout, pause_on);``` 暂停 Audio，[方法详情](SDL_AoutPauseAudio.md)。

[返回](ijkmp_dec_ref_p.md)