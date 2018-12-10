# av\_lockmgr\_register

声明位于 ijkplayer/ijkplayer-x86_64/src/main/jni/ffmpeg/include/libavcodec/avcodec.h 中：

```
/**
 * Lock operation used by lockmgr
 */
enum AVLockOp {
  AV_LOCK_CREATE,  ///< Create a mutex
  AV_LOCK_OBTAIN,  ///< Lock the mutex
  AV_LOCK_RELEASE, ///< Unlock the mutex
  AV_LOCK_DESTROY, ///< Free mutex resources
};

/**
 * Register a user provided lock manager supporting the operations
 * specified by AVLockOp. The "mutex" argument to the function points
 * to a (void *) where the lockmgr should store/get a pointer to a user
 * allocated mutex. It is NULL upon AV_LOCK_CREATE and equal to the
 * value left by the last call for all other ops. If the lock manager is
 * unable to perform the op then it should leave the mutex in the same
 * state as when it was called and return a non-zero value. However,
 * when called with AV_LOCK_DESTROY the mutex will always be assumed to
 * have been successfully destroyed. If av_lockmgr_register succeeds
 * it will return a non-negative value, if it fails it will return a
 * negative value and destroy all mutex and unregister all callbacks.
 * av_lockmgr_register is not thread-safe, it must be called from a
 * single thread before any calls which make use of locking are used.
 *
 * @param cb User defined callback. av_lockmgr_register invokes calls
 *           to this callback and the previously registered callback.
 *           The callback will be used to create more than one mutex
 *           each of which must be backed by its own underlying locking
 *           mechanism (i.e. do not use a single static object to
 *           implement your lock manager). If cb is set to NULL the
 *           lockmgr will be unregistered.
 */
int av_lockmgr_register(int (*cb)(void **mutex, enum AVLockOp op));
```

实现在 libavcodec/utils.c 中：

```
// 默认的回调。

#if HAVE_PTHREADS || HAVE_W32THREADS || HAVE_OS2THREADS
static int default_lockmgr_cb(void **arg, enum AVLockOp op)
{
    void * volatile * mutex = arg;
    int err;

    switch (op) {
    case AV_LOCK_CREATE:
        return 0;
    case AV_LOCK_OBTAIN:
        if (!*mutex) {
            pthread_mutex_t *tmp = av_malloc(sizeof(pthread_mutex_t));
            if (!tmp)
                return AVERROR(ENOMEM);
            if ((err = pthread_mutex_init(tmp, NULL))) {
                av_free(tmp);
                return AVERROR(err);
            }
            if (avpriv_atomic_ptr_cas(mutex, NULL, tmp)) {
                pthread_mutex_destroy(tmp);
                av_free(tmp);
            }
        }

        if ((err = pthread_mutex_lock(*mutex)))
            return AVERROR(err);

        return 0;
    case AV_LOCK_RELEASE:
        if ((err = pthread_mutex_unlock(*mutex)))
            return AVERROR(err);

        return 0;
    case AV_LOCK_DESTROY:
        if (*mutex)
            pthread_mutex_destroy(*mutex);
        av_free(*mutex);
        avpriv_atomic_ptr_cas(mutex, *mutex, NULL);
        return 0;
    }
    return 1;
}
static int (*lockmgr_cb)(void **mutex, enum AVLockOp op) = default_lockmgr_cb;
#else
static int (*lockmgr_cb)(void **mutex, enum AVLockOp op) = NULL;
#endif
 
...

// 可自定义回调
int av_lockmgr_register(int (*cb)(void **mutex, enum AVLockOp op))
{
    if (lockmgr_cb) {
        // There is no good way to rollback a failure to destroy the
        // mutex, so we ignore failures.
        lockmgr_cb(&codec_mutex,    AV_LOCK_DESTROY);
        lockmgr_cb(&avformat_mutex, AV_LOCK_DESTROY);
        lockmgr_cb     = NULL;
        codec_mutex    = NULL;
        avformat_mutex = NULL;
    }

    if (cb) {
        void *new_codec_mutex    = NULL;
        void *new_avformat_mutex = NULL;
        int err;
        if (err = cb(&new_codec_mutex, AV_LOCK_CREATE)) {
            return err > 0 ? AVERROR_UNKNOWN : err;
        }
        if (err = cb(&new_avformat_mutex, AV_LOCK_CREATE)) {
            // Ignore failures to destroy the newly created mutex.
            cb(&new_codec_mutex, AV_LOCK_DESTROY);
            return err > 0 ? AVERROR_UNKNOWN : err;
        }
        lockmgr_cb     = cb;
        codec_mutex    = new_codec_mutex;
        avformat_mutex = new_avformat_mutex;
    }

    return 0;
}
```

对应不同的处理器，在根目录下有 android/contrib/ffmpeg-***/libavcodec/utils.c 中。不过代码相同。

[返回](ijkplayer_main.md)