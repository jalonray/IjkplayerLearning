#libavutil_atomic

libavutil/atomic_suncc.h:

```
#include <atomic.h>
#include <mbarrier.h>

#include "atomic.h"
#define avpriv_atomic_ptr_cas atomic_ptr_cas_suncc
static inline void *atomic_ptr_cas_suncc(void * volatile *ptr,
                                         void *oldval, void *newval)
{
    return atomic_cas_ptr(ptr, oldval, newval);
}
```
atomic.h 是 Linux 下的标准函数库，详情见[源码](https://elixir.bootlin.com/linux/latest/source/include/linux/atomic.h)和[详解](https://www.kernel.org/doc/html/v4.12/core-api/atomic_ops.html)以及对应的[方法说明](https://docs.oracle.com/cd/E88353_01/html/E37855/atomic-cas-ptr-9f.html)

libavutil/atomic\_win32.h:

```
#define WIN32_LEAN_AND_MEAN
#include <windows.h>

#define avpriv_atomic_ptr_cas atomic_ptr_cas_win32
static inline void *atomic_ptr_cas_win32(void * volatile *ptr,
                                         void *oldval, void *newval)
{
    return InterlockedCompareExchangePointer(ptr, newval, oldval);
}
```
这是封装 Windows 下的原子操作库，详情见 [Windows.h 使用](https://docs.microsoft.com/en-us/windows/desktop/winprog/using-the-windows-headers)和对应的[方法说明](https://docs.microsoft.com/en-us/windows/desktop/api/winnt/nf-winnt-interlockedcompareexchangepointer)

libavutil/atomic\_gcc.h:

```
#include <stdint.h>

#include "atomic.h"
#define avpriv_atomic_ptr_cas atomic_ptr_cas_gcc
static inline void *atomic_ptr_cas_gcc(void * volatile *ptr,
                                       void *oldval, void *newval)
{
#if HAVE_SYNC_VAL_COMPARE_AND_SWAP
#ifdef __ARMCC_VERSION
    // armcc will throw an error if ptr is not an integer type
    volatile uintptr_t *tmp = (volatile uintptr_t*)ptr;
    return (void*)__sync_val_compare_and_swap(tmp, oldval, newval);
#else
    return __sync_val_compare_and_swap(ptr, oldval, newval);
#endif
#else
    __atomic_compare_exchange_n(ptr, &oldval, newval, 0, __ATOMIC_SEQ_CST, __ATOMIC_SEQ_CST);
    return oldval;
#endif
}
```
这是封装标准 C 库的原子操作库，详情见[库说明](http://pubs.opengroup.org/onlinepubs/009695399/basedefs/stdint.h.html)和方法详解 [\_\_sync\_val\_compare\_and\_swap]()，[\_\_atomic\_compare\_exchange\_n](https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html)

最后封装到 libavutil/atomic.h：

```
#include "config.h"

#if HAVE_ATOMICS_NATIVE

#if HAVE_ATOMICS_GCC
#include "atomic_gcc.h"
#elif HAVE_ATOMICS_WIN32
#include "atomic_win32.h"
#elif HAVE_ATOMICS_SUNCC
#include "atomic_suncc.h"
#endif

#else

...

/**
 * Atomic pointer compare and swap.
 *
 * @param ptr pointer to the pointer to operate on
 * @param oldval do the swap if the current value of *ptr equals to oldval
 * @param newval value to replace *ptr with
 * @return the value of *ptr before comparison
 */
void *avpriv_atomic_ptr_cas(void * volatile *ptr, void *oldval, void *newval);

#endif /* HAVE_ATOMICS_NATIVE */

#endif /* AVUTIL_ATOMIC_H */
```
[返回](avcodec_register_all.md)