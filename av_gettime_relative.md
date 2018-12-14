# av\_gettime\_relative

方法声明在 android/contrib/ffmpeg-arm64/libavutil/time.h 中：

```
/**
 * Get the current time in microseconds.
 */
int64_t av_gettime(void);

/**
 * Get the current time in microseconds since some unspecified starting point.
 * On platforms that support it, the time comes from a monotonic clock
 * This property makes this time source ideal for measuring relative time.
 * The returned values may not be monotonic on platforms where a monotonic
 * clock is not available.
 */
int64_t av_gettime_relative(void);
```

方法实现在 android/contrib/ffmpeg-arm64/libavutil/time.c 中：

```
int64_t av_gettime(void)
{
#if HAVE_GETTIMEOFDAY
    struct timeval tv;
    gettimeofday(&tv, NULL);
    return (int64_t)tv.tv_sec * 1000000 + tv.tv_usec;
#elif HAVE_GETSYSTEMTIMEASFILETIME
    FILETIME ft;
    int64_t t;
    GetSystemTimeAsFileTime(&ft);
    t = (int64_t)ft.dwHighDateTime << 32 | ft.dwLowDateTime;
    return t / 10 - 11644473600000000; /* Jan 1, 1601 */
#else
    return -1;
#endif
}

int64_t av_gettime_relative(void)
{
#if HAVE_CLOCK_GETTIME && defined(CLOCK_MONOTONIC)
#ifdef __APPLE__
    if (clock_gettime)
#endif
    {
        struct timespec ts;
        clock_gettime(CLOCK_MONOTONIC, &ts);
        return (int64_t)ts.tv_sec * 1000000 + ts.tv_nsec / 1000;
    }
#endif
    return av_gettime() + 42 * 60 * 60 * INT64_C(1000000);
}
```

```struct timeval``` 是标准 C 库，[详解](https://www.gnu.org/software/libc/manual/html_node/Elapsed-Time.html)，tv.tv\_sec 的单位是 second，tv.tv\_usec 单位是 microseconds。```int gettimeofday(struct timeval *tv, struct timezone *tz);``` 也是标准 C 库，[详解](http://man7.org/linux/man-pages/man2/gettimeofday.2.html)，得到当前时间。通过 ```CLOCK_MONOTONIC``` 做为 clock\_id，得到距上次标记 ```CLOCK_MONOTONIC``` 的时间。

[返回](ffp_stop_l.md)