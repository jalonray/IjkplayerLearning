#libavfilter/avfilter.h

```void avfilter_register_all(void)``` 定义在 libavfilter/avfilter.h 中：

```
 /**
  * Register a filter. This is only needed if you plan to use
  * avfilter_get_by_name later to lookup the AVFilter structure by name. A
  * filter can still by instantiated with avfilter_graph_alloc_filter even if it
  * is not registered.
  *
  * @param filter the filter to register
  * @return 0 if the registration was successful, a negative value
  * otherwise
  */
 int avfilter_register(AVFilter *filter);

/** Initialize the filter system. Register all builtin filters. */
void avfilter_register_all(void);
```

实现在 libavfilter/allfilters.c 中：

```
#include "avfilter.h"
#include "config.h"
#include "opencl_allkernels.h"

#define REGISTER_FILTER(X, x, y)                                        \
    {                                                                   \
        extern AVFilter ff_##y##_##x;                                   \
        if (CONFIG_##X##_FILTER)                                        \
            avfilter_register(&ff_##y##_##x);                           \
    }

#define REGISTER_FILTER_UNCONDITIONAL(x)                                \
    {                                                                   \
        extern AVFilter ff_##x;                                         \
        avfilter_register(&ff_##x);                                     \
    }

void avfilter_register_all(void)
{
    static int initialized;

     if (initialized)
        return;
    initialized = 1;

    REGISTER_FILTER(ABENCH,         abench,         af);
    ...
    /* multimedia filters */
    REGISTER_FILTER(ABITSCOPE,      abitscope,      avf);
    ...
    /* multimedia sources */
    REGISTER_FILTER(AMOVIE,         amovie,         avsrc);
    REGISTER_FILTER(MOVIE,          movie,          avsrc);
     
    /* those filters are part of public or internal API => registered
     * unconditionally */
    REGISTER_FILTER_UNCONDITIONAL(asrc_abuffer);
    ...
    ff_opencl_register_filter_kernel_code_all();
}
```

```ff_opencl_register_filter_kernel_code_all();``` 用于注册一些 opencl filter 的内核，[详细分析](libavfilter_opencl_allkernels.md)

类型 AVFilter 的定义也在 libavfilter/avfilter.h 中：

```
/**
 * Filter definition. This defines the pads a filter contains, and all the
 * callback functions used to interact with the filter.
 */
typedef struct AVFilter {
    /**
     * Filter name. Must be non-NULL and unique among filters.
     */
    const char *name;

    /**
     * A description of the filter. May be NULL.
     *
     * You should use the NULL_IF_CONFIG_SMALL() macro to define it.
     */
    const char *description;

    /**
     * List of inputs, terminated by a zeroed element.
     *
     * NULL if there are no (static) inputs. Instances of filters with
     * AVFILTER_FLAG_DYNAMIC_INPUTS set may have more inputs than present in
     * this list.
     */
    const AVFilterPad *inputs;
    /**
     * List of outputs, terminated by a zeroed element.
     *
     * NULL if there are no (static) outputs. Instances of filters with
     * AVFILTER_FLAG_DYNAMIC_OUTPUTS set may have more outputs than present in
     * this list.
     */
    const AVFilterPad *outputs;
     
    /**
     * A class for the private data, used to declare filter private AVOptions.
     * This field is NULL for filters that do not declare any options.
     *
     * If this field is non-NULL, the first member of the filter private data
     * must be a pointer to AVClass, which will be set by libavfilter generic
     * code to this class.
     */
    const AVClass *priv_class;

    /**
     * A combination of AVFILTER_FLAG_*
     */
    int flags;

    /*****************************************************************
     * All fields below this line are not part of the public API. They
     * may not be used outside of libavfilter and can be changed and
     * removed at will.
     * New public fields should be added right above.
     *****************************************************************
     */

    /**
     * Filter initialization function.
     *
     * This callback will be called only once during the filter lifetime, after
     * all the options have been set, but before links between filters are
     * established and format negotiation is done.
     *
     * Basic filter initialization should be done here. Filters with dynamic
     * inputs and/or outputs should create those inputs/outputs here based on
     * provided options. No more changes to this filter's inputs/outputs can be
     * done after this callback.
     *
     * This callback must not assume that the filter links exist or frame
     * parameters are known.
     *
     * @ref AVFilter.uninit "uninit" is guaranteed to be called even if
     * initialization fails, so this callback does not have to clean up on
     * failure.
     *
     * @return 0 on success, a negative AVERROR on failure
     */
    int (*init)(AVFilterContext *ctx);
     
    /**
     * Should be set instead of @ref AVFilter.init "init" by the filters that
     * want to pass a dictionary of AVOptions to nested contexts that are
     * allocated during init.
     *
     * On return, the options dict should be freed and replaced with one that
     * contains all the options which could not be processed by this filter (or
     * with NULL if all the options were processed).
     *
     * Otherwise the semantics is the same as for @ref AVFilter.init "init".
     */
    int (*init_dict)(AVFilterContext *ctx, AVDictionary **options);

    /**
     * Filter uninitialization function.
     *
     * Called only once right before the filter is freed. Should deallocate any
     * memory held by the filter, release any buffer references, etc. It does
     * not need to deallocate the AVFilterContext.priv memory itself.
     *
     * This callback may be called even if @ref AVFilter.init "init" was not
     * called or failed, so it must be prepared to handle such a situation.
     */
    void (*uninit)(AVFilterContext *ctx);

    /**
     * Query formats supported by the filter on its inputs and outputs.
     *
     * This callback is called after the filter is initialized (so the inputs
     * and outputs are fixed), shortly before the format negotiation. This
     * callback may be called more than once.
     *
     * This callback must set AVFilterLink.out_formats on every input link and
     * AVFilterLink.in_formats on every output link to a list of pixel/sample
     * formats that the filter supports on that link. For audio links, this
     * filter must also set @ref AVFilterLink.in_samplerates "in_samplerates" /
     * @ref AVFilterLink.out_samplerates "out_samplerates" and
     * @ref AVFilterLink.in_channel_layouts "in_channel_layouts" /
     * @ref AVFilterLink.out_channel_layouts "out_channel_layouts" analogously.
     *
     * This callback may be NULL for filters with one input, in which case
     * libavfilter assumes that it supports all input formats and preserves
     * them on output.
     *
     * @return zero on success, a negative value corresponding to an
     * AVERROR code otherwise
     */
    int (*query_formats)(AVFilterContext *);
     
    int priv_size;      ///< size of private data to allocate for the filter

    /**
     * Used by the filter registration system. Must not be touched by any other
     * code.
     */
    struct AVFilter *next;

    /**
     * Make the filter instance process a command.
     *
     * @param cmd    the command to process, for handling simplicity all commands must be alphanumeric only
     * @param arg    the argument for the command
     * @param res    a buffer with size res_size where the filter(s) can return a response. This must not change when the command is  not supported.
     * @param flags  if AVFILTER_CMD_FLAG_FAST is set and the command would be
     *               time consuming then a filter should treat it like an unsupported command
     *
     * @returns >=0 on success otherwise an error code.
     *          AVERROR(ENOSYS) on unsupported commands
     */
    int (*process_command)(AVFilterContext *, const char *cmd, const char *arg, char *res, int res_len, int flags);

    /**
     * Filter initialization function, alternative to the init()
     * callback. Args contains the user-supplied parameters, opaque is
     * used for providing binary data.
     */
    int (*init_opaque)(AVFilterContext *ctx, void *opaque);

    /**
     * Filter activation function.
     *
     * Called when any processing is needed from the filter, instead of any
     * filter_frame and request_frame on pads.
     *
     * The function must examine inlinks and outlinks and perform a single
     * step of processing. If there is nothing to do, the function must do
     * nothing and not return an error. If more steps are or may be
     * possible, it must use ff_filter_set_ready() to schedule another
     * activation.
     */
    int (*activate)(AVFilterContext *ctx);
} AVFilter;
```