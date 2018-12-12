# ijkmp\_set\_data\_source

方法声明在 ijkmedia/ijkplayer/ijkplayer.h 中：

```
int ijkmp_set_data_source(IjkMediaPlayer *mp, const char *url);
```

实现在 ijkmedia/ijkplayer/ijkplayer.c 中：

```
#define MP_RET_IF_FAILED(ret) \
    do { \
        int retval = ret; \
        if (retval != 0) return (retval); \
    } while(0)

#define MPST_RET_IF_EQ_INT(real, expected, errcode) \
    do { \
        if ((real) == (expected)) return (errcode); \
    } while(0)

#define MPST_RET_IF_EQ(real, expected) \
     MPST_RET_IF_EQ_INT(real, expected, EIJK_INVALID_STATE)

static int ijkmp_set_data_source_l(IjkMediaPlayer *mp, const char *url)
{
    assert(mp);
    assert(url);

    // MPST_RET_IF_EQ(mp->mp_state, MP_STATE_IDLE);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_INITIALIZED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_ASYNC_PREPARING);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_PREPARED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_STARTED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_PAUSED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_COMPLETED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_STOPPED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_ERROR);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_END);

    freep((void**)&mp->data_source);
    mp->data_source = strdup(url);
    if (!mp->data_source)
        return EIJK_OUT_OF_MEMORY;

    ijkmp_change_state_l(mp, MP_STATE_INITIALIZED);
    return 0;
}

int ijkmp_set_data_source(IjkMediaPlayer *mp, const char *url)
{
    assert(mp);
    assert(url);
    MPTRACE("ijkmp_set_data_source(url=\"%s\")\n", url);
    pthread_mutex_lock(&mp->mutex);
    int retval = ijkmp_set_data_source_l(mp, url);
    pthread_mutex_unlock(&mp->mutex);
    MPTRACE("ijkmp_set_data_source(url=\"%s\")=%d\n", url, retval);
    return retval;
}
```

[返回](ijkplayer_main.md)