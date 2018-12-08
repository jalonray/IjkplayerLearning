#ffmpeg/libavutil/thread.h

下面是 ffmpeg/compat/os2threds.h 里的一些关键内容，封装了 Dos 的线程函数

```
/**
 * @file
 * os2threads to pthreads wrapper
 */

#ifndef COMPAT_OS2THREADS_H
#define COMPAT_OS2THREADS_H

#define INCL_DOS
#include <os2.h>

#undef __STRICT_ANSI__          /* for _beginthread() */
#include <stdlib.h>

#include <sys/builtin.h>
#include <sys/fmutex.h>

#include "libavutil/attributes.h"

typedef struct {
    TID tid;
    void *(*start_routine)(void *);
    void *arg;
    void *result;
} pthread_t;

typedef void pthread_mutexattr_t;

static av_always_inline int pthread_mutex_init(pthread_mutex_t *mutex,
                                               const pthread_mutexattr_t *attr)
{
    DosCreateMutexSem(NULL, (PHMTX)mutex, 0, FALSE);

    return 0;
}

```
```DosCreateMutexSem(NULL, (PHMTX)mutex, 0, FALSE);``` 的定义在[这里](http://www.edm2.com/os2api/Dos/DosCreateMutexSem.html)

下面是 ffmpeg/compat/w32pthreads.h 里的一些关键内容，封装了 Windows 下的线程函数

```
/**
 * @file
 * w32threads to pthreads wrapper
 */

#ifndef COMPAT_W32PTHREADS_H
#define COMPAT_W32PTHREADS_H

/* Build up a pthread-like API using underlying Windows API. Have only static
 * methods so as to not conflict with a potentially linked in pthread-win32
 * library.
 * As most functions here are used without checking return values,
 * only implement return values as necessary. */

#define WIN32_LEAN_AND_MEAN
#include <windows.h>
#include <process.h>

#if _WIN32_WINNT < 0x0600 && defined(__MINGW32__)
#undef MemoryBarrier
#define MemoryBarrier __sync_synchronize
#endif

#include "libavutil/attributes.h"
#include "libavutil/common.h"
#include "libavutil/internal.h"
#include "libavutil/mem.h"

typedef struct pthread_t {
    void *handle;
    void *(*func)(void* arg);
    void *arg;
    void *ret;
} pthread_t;

/* the conditional variable api for windows 6.0+ uses critical sections and
 * not mutexes */
typedef CRITICAL_SECTION pthread_mutex_t;

/* This is the CONDITION_VARIABLE typedef for using Windows' native
 * conditional variables on kernels 6.0+. */
#if HAVE_CONDITION_VARIABLE_PTR
typedef CONDITION_VARIABLE pthread_cond_t;
#else
typedef struct pthread_cond_t {
    void *Ptr;
} pthread_cond_t;
#endif

#if _WIN32_WINNT >= 0x0600
#define InitializeCriticalSection(x) InitializeCriticalSectionEx(x, 0, 0)
#define WaitForSingleObject(a, b) WaitForSingleObjectEx(a, b, FALSE)
#endif

static inline int pthread_mutex_init(pthread_mutex_t *m, void* attr)
{
    InitializeCriticalSection(m);
    return 0;
}

```
```InitializeCriticalSection(m);```的定义在[这里](https://docs.microsoft.com/en-us/windows/desktop/api/synchapi/nf-synchapi-initializecriticalsectionex)

最后是封装于 ffmpeg/libavutil/thread.h 的方法：

```
// This header should only be used to simplify code where
// threading is optional, not as a generic threading abstraction.

#ifndef AVUTIL_THREAD_H
#define AVUTIL_THREAD_H

#include "config.h"

#if HAVE_PTHREADS || HAVE_W32THREADS || HAVE_OS2THREADS

#define USE_ATOMICS 0

#if HAVE_PTHREADS
#include <pthread.h>

#if defined(ASSERT_LEVEL) && ASSERT_LEVEL > 1

#include "log.h"

#define ASSERT_PTHREAD_NORET(func, ...) do {                            \
    int ret = func(__VA_ARGS__);                                        \
    if (ret) {                                                          \
        av_log(NULL, AV_LOG_FATAL, AV_STRINGIFY(func)                   \
               " failed with error: %s\n", av_err2str(AVERROR(ret)));   \
        abort();                                                        \
    }                                                                   \
} while (0)

static inline int strict_pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr)
{
    if (attr) {
        ASSERT_PTHREAD_NORET(pthread_mutex_init, mutex, attr);
    } else {
        pthread_mutexattr_t local_attr;
        ASSERT_PTHREAD_NORET(pthread_mutexattr_init, &local_attr);
        ASSERT_PTHREAD_NORET(pthread_mutexattr_settype, &local_attr, PTHREAD_MUTEX_ERRORCHECK);
        ASSERT_PTHREAD_NORET(pthread_mutex_init, mutex, &local_attr);
        ASSERT_PTHREAD_NORET(pthread_mutexattr_destroy, &local_attr);
    }
    return 0;
}

#define pthread_mutex_init     strict_pthread_mutex_init
...
#endif

#elif HAVE_OS2THREADS
#include "compat/os2threads.h"
#else
#include "compat/w32pthreads.h"
#endif

#define AVMutex pthread_mutex_t

#define ff_mutex_init    pthread_mutex_init
```

上述代码对 Thread 函数引用分三个部分： 

```
// 三种环境
#if HAVE_PTHREADS || HAVE_W32THREADS || HAVE_OS2THREADS
...
// linux 下
#if HAVE_PTHREADS
#include <pthread.h>
...
// dos 下
#elif HAVE_OS2THREADS
#include "compat/os2threads.h"
// windows 下
#else
#include "compat/w32pthreads.h"
#endif
...
```
这样做是因为 ffmpeg 是跨平台的播放内核，但是在 android 里只用得上 linux 内核的线程函数。linux 线程函数可以看[这里](http://man7.org/linux/man-pages/man0/pthread.h.0p.html)

[返回](ijkplayer_main.md)