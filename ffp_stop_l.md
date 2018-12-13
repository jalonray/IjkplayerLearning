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


[返回](ijkmp_dec_ref_p.md)