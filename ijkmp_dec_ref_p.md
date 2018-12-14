# ijkmp\_dec\_ref\_p

方法声明在 ijkmedia/ijkplayer/ijkplayer.h 中：

```
void ijkmp_dec_ref_p(IjkMediaPlayer **pmp);
```

实现在 ijkmedia/ijkplayer/ijkplayer.h 中：

```
void ijkmp_dec_ref_p(IjkMediaPlayer **pmp)
{
    if (!pmp)
        return;

    ijkmp_dec_ref(*pmp);
    *pmp = NULL;
}

void ijkmp_dec_ref(IjkMediaPlayer *mp)
{
    if (!mp)
        return;

    int ref_count = __sync_sub_and_fetch(&mp->ref_count, 1);
    if (ref_count == 0) {
        MPTRACE("ijkmp_dec_ref(): ref=0\n");
        ijkmp_shutdown(mp);
        ijkmp_destroy_p(&mp);
    }
}

void ijkmp_shutdown(IjkMediaPlayer *mp)
{
    return ijkmp_shutdown_l(mp);
}

void ijkmp_shutdown_l(IjkMediaPlayer *mp)
{
    assert(mp);

    MPTRACE("ijkmp_shutdown_l()\n");
    if (mp->ffplayer) {
        ffp_stop_l(mp->ffplayer);
        ffp_wait_stop_l(mp->ffplayer);
    }
    MPTRACE("ijkmp_shutdown_l()=void\n");
}

inline static void ijkmp_destroy_p(IjkMediaPlayer **pmp)
{
    if (!pmp)
        return;

    ijkmp_destroy(*pmp);
    *pmp = NULL;
}

inline static void ijkmp_destroy(IjkMediaPlayer *mp)
{
    if (!mp)
        return;

    ffp_destroy_p(&mp->ffplayer);
    if (mp->msg_thread) {
        SDL_WaitThread(mp->msg_thread, NULL);
        mp->msg_thread = NULL;
    }

    pthread_mutex_destroy(&mp->mutex);

    freep((void**)&mp->data_source);
    memset(mp, 0, sizeof(IjkMediaPlayer));
    freep((void**)&mp);
}
```

用于释放对 pmp 的引用。之后再看几个 ffp_* 的方法。

```ffp_stop_l(mp->ffplayer);``` 方法声明在 ijkmedia/ijkplayer/ff_ffplay.h 中：

```
/* playback controll */
int ffp_prepare_async_l(FFPlayer *ffp, const char *file_name);
int fp_start_from_l(FFPlayer *ffp, long msec);
int ffp_start_l(FFPlayer *ffp);
int ffp_pause_l(FFPlayer *ffp);
int ffp_is_paused_l(FFPlayer *ffp);
int ffp_stop_l(FFPlayer *ffp);
int ffp_wait_stop_l(FFPlayer *ffp);
```

通过定义知，是让 FFPlayer 停止播放的播放控制方法。[方法详解](ffp_stop_l.md)

同样的 ```ffp_wait_stop_l(mp->ffplayer);``` 是等待 FFPlayer 停止播放的控制方法。[方法详解](ffp_wait_stop_l.md)

之后 ```ijkmp_destroy_p(&mp);``` 销毁 mp 的内存。[方法详解](ijkmp_destroy_p.md)

[返回](ijkplayer_main.md)