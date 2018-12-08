#libavfilter/opencl_allkernels.h

声明：
libavfilter/opencl_allkernels.h 中：

```
#ifndef AVFILTER_OPENCL_ALLKERNELS_H
#define AVFILTER_OPENCL_ALLKERNELS_H

#include "avfilter.h"
#include "config.h"

void ff_opencl_register_filter_kernel_code_all(void);

#endif /* AVFILTER_OPENCL_ALLKERNELS_H */
```

实现在 libavfilter/opencl_allkernels.c 中：

```
#include "opencl_allkernels.h"
#if CONFIG_OPENCL
#include "libavutil/opencl.h"
#include "deshake_opencl_kernel.h"
#include "unsharp_opencl_kernel.h"
#endif

#define OPENCL_REGISTER_KERNEL_CODE(X, x)                                              \
    {                                                                                  \
        if (CONFIG_##X##_FILTER) {                                                     \
            av_opencl_register_kernel_code(ff_kernel_##x##_opencl);                    \
        }                                                                              \
    }

void ff_opencl_register_filter_kernel_code_all(void)
{
 #if CONFIG_OPENCL
   OPENCL_REGISTER_KERNEL_CODE(DESHAKE,     deshake);
   OPENCL_REGISTER_KERNEL_CODE(UNSHARP,     unsharp);
 #endif
}
```

方法```int av_opencl_register_kernel_code(const char *kernel_code)```声明在 libavutil/opencl.h 中：

```
/**
 * Register kernel code.
 *
 *  The registered kernel code is stored in a global context, and compiled
 *  in the runtime environment when av_opencl_init() is called.
 *
 * @param kernel_code    kernel code to be compiled in the OpenCL runtime environment
 * @return  >=0 on success, a negative error code in case of failure
 */
int av_opencl_register_kernel_code(const char *kernel_code);
```

实现在 libavutil/opencl.c 

```
#include "opencl.h"
#include "avstring.h"
#include "log.h"
#include "avassert.h"
#include "opt.h"

#if HAVE_THREADS
#include "thread.h"
#include "atomic.h"

static pthread_mutex_t * volatile atomic_opencl_lock = NULL;
#define LOCK_OPENCL pthread_mutex_lock(atomic_opencl_lock)
#define UNLOCK_OPENCL pthread_mutex_unlock(atomic_opencl_lock)
#else
#define LOCK_OPENCL
#define UNLOCK_OPENCL
#endif

#define MAX_KERNEL_CODE_NUM 200

typedef struct {
    int is_compiled;
    const char *kernel_string;
} KernelCode;

typedef struct {
    const AVClass *class;
    int log_offset;
    void *log_ctx;
    int init_count;
    int opt_init_flag;
     /**
     * if set to 1, the OpenCL environment was created by the user and
     * passed as AVOpenCLExternalEnv when initing ,0:created by opencl wrapper.
     */
    int is_user_created;
    int platform_idx;
    int device_idx;
    cl_platform_id platform_id;
    cl_device_type device_type;
    cl_context context;
    cl_device_id device_id;
    cl_command_queue command_queue;
    int kernel_code_count;
    KernelCode kernel_code[MAX_KERNEL_CODE_NUM];
    AVOpenCLDeviceList device_list;
} OpenclContext;

static const AVClass openclutils_class = {
    .class_name                = "opencl",
    .option                    = opencl_options,
    .item_name                 = av_default_item_name,
    .version                   = LIBAVUTIL_VERSION_INT,
    .log_level_offset_offset   = offsetof(OpenclContext, log_offset),
    .parent_log_context_offset = offsetof(OpenclContext, log_ctx),
};

static OpenclContext opencl_ctx = {&openclutils_class};
 
int av_opencl_register_kernel_code(const char *kernel_code)
{
    int i, ret = init_opencl_mtx( );
    if (ret < 0)
        return ret;
    LOCK_OPENCL;
    if (opencl_ctx.kernel_code_count >= MAX_KERNEL_CODE_NUM) {
        av_log(&opencl_ctx, AV_LOG_ERROR,
               "Could not register kernel code, maximum number of registered kernel code %d already reached\n",
               MAX_KERNEL_CODE_NUM);
        ret = AVERROR(EINVAL);
        goto end;
    }
    for (i = 0; i < opencl_ctx.kernel_code_count; i++) {
        if (opencl_ctx.kernel_code[i].kernel_string == kernel_code) {
            av_log(&opencl_ctx, AV_LOG_WARNING, "Same kernel code has been registered\n");
            goto end;
        }
    }
    opencl_ctx.kernel_code[opencl_ctx.kernel_code_count].kernel_string = kernel_code;
    opencl_ctx.kernel_code[opencl_ctx.kernel_code_count].is_compiled = 0;
    opencl_ctx.kernel_code_count++;
end:
    UNLOCK_OPENCL;
    return ret;
}

static inline int init_opencl_mtx(void)
{
#if HAVE_THREADS
    if (!atomic_opencl_lock) {
        int err;
        pthread_mutex_t *tmp = av_malloc(sizeof(pthread_mutex_t));
        if (!tmp)
            return AVERROR(ENOMEM);
        if ((err = pthread_mutex_init(tmp, NULL))) {
            av_free(tmp);
            return AVERROR(err);
        }
        if (avpriv_atomic_ptr_cas((void * volatile *)&atomic_opencl_lock, NULL, tmp)) {
            pthread_mutex_destroy(tmp);
            av_free(tmp);
        }
    }
#endif
    return 0;
}
```
[返回](libavfilter_avfilter.md)