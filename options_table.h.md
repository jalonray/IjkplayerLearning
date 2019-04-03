# options\_table.h

```
static const AVOption avformat_options[] = {
{"avioflags", NULL, OFFSET(avio_flags), AV_OPT_TYPE_FLAGS, {.i64 = DEFAULT }, INT_MIN, INT_MAX, D|E, "avioflags"},
{"direct", "reduce buffering", 0, AV_OPT_TYPE_CONST, {.i64 = AVIO_FLAG_DIRECT }, INT_MIN, INT_MAX, D|E, "avioflags"},
{"probesize", "set probing size", OFFSET(probesize), AV_OPT_TYPE_INT64, {.i64 = 5000000 }, 32, INT64_MAX, D},
{"formatprobesize", "number of bytes to probe file format", OFFSET(format_probesize), AV_OPT_TYPE_INT, {.i64 = PROBE_BUF_MAX}, 0, INT_MAX-1, D},
{"packetsize", "set packet size", OFFSET(packet_size), AV_OPT_TYPE_INT, {.i64 = DEFAULT }, 0, INT_MAX, E},
{"fflags", NULL, OFFSET(flags), AV_OPT_TYPE_FLAGS, {.i64 = AVFMT_FLAG_AUTO_BSF }, INT_MIN, INT_MAX, D|E, "fflags"},
{"flush_packets", "reduce the latency by flushing out packets immediately", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_FLUSH_PACKETS }, INT_MIN, INT_MAX, E, "fflags"},
{"ignidx", "ignore index", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_IGNIDX }, INT_MIN, INT_MAX, D, "fflags"},
{"genpts", "generate pts", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_GENPTS }, INT_MIN, INT_MAX, D, "fflags"},
{"nofillin", "do not fill in missing values that can be exactly calculated", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_NOFILLIN }, INT_MIN, INT_MAX, D, "fflags"},
{"noparse", "disable AVParsers, this needs nofillin too", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_NOPARSE }, INT_MIN, INT_MAX, D, "fflags"},
{"igndts", "ignore dts", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_IGNDTS }, INT_MIN, INT_MAX, D, "fflags"},
{"discardcorrupt", "discard corrupted frames", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_DISCARD_CORRUPT }, INT_MIN, INT_MAX, D, "fflags"},
{"sortdts", "try to interleave outputted packets by dts", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_SORT_DTS }, INT_MIN, INT_MAX, D, "fflags"},
#if FF_API_LAVF_KEEPSIDE_FLAG
{"keepside", "don't merge side data", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_KEEP_SIDE_DATA }, INT_MIN, INT_MAX, D, "fflags"},
#endif
{"fastseek", "fast but inaccurate seeks", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_FAST_SEEK }, INT_MIN, INT_MAX, D, "fflags"},
{"latm", "enable RTP MP4A-LATM payload", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_MP4A_LATM }, INT_MIN, INT_MAX, E, "fflags"},
{"nobuffer", "reduce the latency introduced by optional buffering", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_FLAG_NOBUFFER }, 0, INT_MAX, D, "fflags"},
{"seek2any", "allow seeking to non-keyframes on demuxer level when supported", OFFSET(seek2any), AV_OPT_TYPE_BOOL, {.i64 = 0 }, 0, 1, D},
{"bitexact", "do not write random/volatile data", 0, AV_OPT_TYPE_CONST, { .i64 = AVFMT_FLAG_BITEXACT }, 0, 0, E, "fflags" },
{"shortest", "stop muxing with the shortest stream", 0, AV_OPT_TYPE_CONST, { .i64 = AVFMT_FLAG_SHORTEST }, 0, 0, E, "fflags" },
{"autobsf", "add needed bsfs automatically (delays header until each stream's first packet is written)", 0, AV_OPT_TYPE_CONST, { .i64 = AVFMT_FLAG_AUTO_BSF }, 0, 0, E, "fflags" },
{"analyzeduration", "specify how many microseconds are analyzed to probe the input", OFFSET(max_analyze_duration), AV_OPT_TYPE_INT64, {.i64 = 0 }, 0, INT64_MAX, D},
{"cryptokey", "decryption key", OFFSET(key), AV_OPT_TYPE_BINARY, {.dbl = 0}, 0, 0, D},
{"indexmem", "max memory used for timestamp index (per stream)", OFFSET(max_index_size), AV_OPT_TYPE_INT, {.i64 = 1<<20 }, 0, INT_MAX, D},
{"rtbufsize", "max memory used for buffering real-time frames", OFFSET(max_picture_buffer), AV_OPT_TYPE_INT, {.i64 = 3041280 }, 0, INT_MAX, D}, /* defaults to 1s of 15fps 352x288 YUYV422 video */
{"fdebug", "print specific debug info", OFFSET(debug), AV_OPT_TYPE_FLAGS, {.i64 = DEFAULT }, 0, INT_MAX, E|D, "fdebug"},
{"ts", NULL, 0, AV_OPT_TYPE_CONST, {.i64 = FF_FDEBUG_TS }, INT_MIN, INT_MAX, E|D, "fdebug"},
{"max_delay", "maximum muxing or demuxing delay in microseconds", OFFSET(max_delay), AV_OPT_TYPE_INT, {.i64 = -1 }, -1, INT_MAX, E|D},
{"start_time_realtime", "wall-clock time when stream begins (PTS==0)", OFFSET(start_time_realtime), AV_OPT_TYPE_INT64, {.i64 = AV_NOPTS_VALUE}, INT64_MIN, INT64_MAX, E},
{"fpsprobesize", "number of frames used to probe fps", OFFSET(fps_probe_size), AV_OPT_TYPE_INT, {.i64 = -1}, -1, INT_MAX-1, D},
{"audio_preload", "microseconds by which audio packets should be interleaved earlier", OFFSET(audio_preload), AV_OPT_TYPE_INT, {.i64 = 0}, 0, INT_MAX-1, E},
{"chunk_duration", "microseconds for each chunk", OFFSET(max_chunk_duration), AV_OPT_TYPE_INT, {.i64 = 0}, 0, INT_MAX-1, E},
{"chunk_size", "size in bytes for each chunk", OFFSET(max_chunk_size), AV_OPT_TYPE_INT, {.i64 = 0}, 0, INT_MAX-1, E},
/* this is a crutch for avconv, since it cannot deal with identically named options in different contexts.
 * to be removed when avconv is fixed */
{"f_err_detect", "set error detection flags (deprecated; use err_detect, save via avconv)", OFFSET(error_recognition), AV_OPT_TYPE_FLAGS, {.i64 = AV_EF_CRCCHECK }, INT_MIN, INT_MAX, D, "err_detect"},
{"err_detect", "set error detection flags", OFFSET(error_recognition), AV_OPT_TYPE_FLAGS, {.i64 = AV_EF_CRCCHECK }, INT_MIN, INT_MAX, D, "err_detect"},
{"crccheck", "verify embedded CRCs", 0, AV_OPT_TYPE_CONST, {.i64 = AV_EF_CRCCHECK }, INT_MIN, INT_MAX, D, "err_detect"},
{"bitstream", "detect bitstream specification deviations", 0, AV_OPT_TYPE_CONST, {.i64 = AV_EF_BITSTREAM }, INT_MIN, INT_MAX, D, "err_detect"},
{"buffer", "detect improper bitstream length", 0, AV_OPT_TYPE_CONST, {.i64 = AV_EF_BUFFER }, INT_MIN, INT_MAX, D, "err_detect"},
{"explode", "abort decoding on minor error detection", 0, AV_OPT_TYPE_CONST, {.i64 = AV_EF_EXPLODE }, INT_MIN, INT_MAX, D, "err_detect"},
{"ignore_err", "ignore errors", 0, AV_OPT_TYPE_CONST, {.i64 = AV_EF_IGNORE_ERR }, INT_MIN, INT_MAX, D, "err_detect"},
{"careful",    "consider things that violate the spec, are fast to check and have not been seen in the wild as errors", 0, AV_OPT_TYPE_CONST, {.i64 = AV_EF_CAREFUL }, INT_MIN, INT_MAX, D, "err_detect"},
{"compliant",  "consider all spec non compliancies as errors", 0, AV_OPT_TYPE_CONST, {.i64 = AV_EF_COMPLIANT }, INT_MIN, INT_MAX, D, "err_detect"},
{"aggressive", "consider things that a sane encoder shouldn't do as an error", 0, AV_OPT_TYPE_CONST, {.i64 = AV_EF_AGGRESSIVE }, INT_MIN, INT_MAX, D, "err_detect"},
{"use_wallclock_as_timestamps", "use wallclock as timestamps", OFFSET(use_wallclock_as_timestamps), AV_OPT_TYPE_BOOL, {.i64 = 0}, 0, 1, D},
{"skip_initial_bytes", "set number of bytes to skip before reading header and frames", OFFSET(skip_initial_bytes), AV_OPT_TYPE_INT64, {.i64 = 0}, 0, INT64_MAX-1, D},
{"correct_ts_overflow", "correct single timestamp overflows", OFFSET(correct_ts_overflow), AV_OPT_TYPE_BOOL, {.i64 = 1}, 0, 1, D},
{"flush_packets", "enable flushing of the I/O context after each packet", OFFSET(flush_packets), AV_OPT_TYPE_INT, {.i64 = -1}, -1, 1, E},
{"metadata_header_padding", "set number of bytes to be written as padding in a metadata header", OFFSET(metadata_header_padding), AV_OPT_TYPE_INT, {.i64 = -1}, -1, INT_MAX, E},
{"output_ts_offset", "set output timestamp offset", OFFSET(output_ts_offset), AV_OPT_TYPE_DURATION, {.i64 = 0}, -INT64_MAX, INT64_MAX, E},
{"max_interleave_delta", "maximum buffering duration for interleaving", OFFSET(max_interleave_delta), AV_OPT_TYPE_INT64, { .i64 = 10000000 }, 0, INT64_MAX, E },
{"f_strict", "how strictly to follow the standards (deprecated; use strict, save via avconv)", OFFSET(strict_std_compliance), AV_OPT_TYPE_INT, {.i64 = DEFAULT }, INT_MIN, INT_MAX, D|E, "strict"},
{"strict", "how strictly to follow the standards", OFFSET(strict_std_compliance), AV_OPT_TYPE_INT, {.i64 = DEFAULT }, INT_MIN, INT_MAX, D|E, "strict"},
{"very", "strictly conform to a older more strict version of the spec or reference software", 0, AV_OPT_TYPE_CONST, {.i64 = FF_COMPLIANCE_VERY_STRICT }, INT_MIN, INT_MAX, D|E, "strict"},
{"strict", "strictly conform to all the things in the spec no matter what the consequences", 0, AV_OPT_TYPE_CONST, {.i64 = FF_COMPLIANCE_STRICT }, INT_MIN, INT_MAX, D|E, "strict"},
{"normal", NULL, 0, AV_OPT_TYPE_CONST, {.i64 = FF_COMPLIANCE_NORMAL }, INT_MIN, INT_MAX, D|E, "strict"},
{"unofficial", "allow unofficial extensions", 0, AV_OPT_TYPE_CONST, {.i64 = FF_COMPLIANCE_UNOFFICIAL }, INT_MIN, INT_MAX, D|E, "strict"},
{"experimental", "allow non-standardized experimental variants", 0, AV_OPT_TYPE_CONST, {.i64 = FF_COMPLIANCE_EXPERIMENTAL }, INT_MIN, INT_MAX, D|E, "strict"},
{"max_ts_probe", "maximum number of packets to read while waiting for the first timestamp", OFFSET(max_ts_probe), AV_OPT_TYPE_INT, { .i64 = 50 }, 0, INT_MAX, D },
{"avoid_negative_ts", "shift timestamps so they start at 0", OFFSET(avoid_negative_ts), AV_OPT_TYPE_INT, {.i64 = -1}, -1, 2, E, "avoid_negative_ts"},
{"auto",              "enabled when required by target format",    0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_AVOID_NEG_TS_AUTO },              INT_MIN, INT_MAX, E, "avoid_negative_ts"},
{"disabled",          "do not change timestamps",                  0, AV_OPT_TYPE_CONST, {.i64 = 0 },                                    INT_MIN, INT_MAX, E, "avoid_negative_ts"},
{"make_non_negative", "shift timestamps so they are non negative", 0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_AVOID_NEG_TS_MAKE_NON_NEGATIVE }, INT_MIN, INT_MAX, E, "avoid_negative_ts"},
{"make_zero",         "shift timestamps so they start at 0",       0, AV_OPT_TYPE_CONST, {.i64 = AVFMT_AVOID_NEG_TS_MAKE_ZERO },         INT_MIN, INT_MAX, E, "avoid_negative_ts"},
{"dump_separator", "set information dump field separator", OFFSET(dump_separator), AV_OPT_TYPE_STRING, {.str = ", "}, CHAR_MIN, CHAR_MAX, D|E},
{"codec_whitelist", "List of decoders that are allowed to be used", OFFSET(codec_whitelist), AV_OPT_TYPE_STRING, { .str = NULL },  CHAR_MIN, CHAR_MAX, D },
{"format_whitelist", "List of demuxers that are allowed to be used", OFFSET(format_whitelist), AV_OPT_TYPE_STRING, { .str = NULL },  CHAR_MIN, CHAR_MAX, D },
{"protocol_whitelist", "List of protocols that are allowed to be used", OFFSET(protocol_whitelist), AV_OPT_TYPE_STRING, { .str = NULL },  CHAR_MIN, CHAR_MAX, D },
{"protocol_blacklist", "List of protocols that are not allowed to be used", OFFSET(protocol_blacklist), AV_OPT_TYPE_STRING, { .str = NULL },  CHAR_MIN, CHAR_MAX, D },
{"max_streams", "maximum number of streams", OFFSET(max_streams), AV_OPT_TYPE_INT, { .i64 = 1000 }, 0, INT_MAX, D },
{NULL},
};
```

avOption 的定义在 extra/ffmpeg/libavutil/opt.h 中：

```
/**
 * @defgroup avoptions AVOptions
 * @ingroup lavu_data
 * @{
 * AVOptions provide a generic system to declare options on arbitrary structs
 * ("objects"). An option can have a help text, a type and a range of possible
 * values. Options may then be enumerated, read and written to.
 *
 * @section avoptions_implement Implementing AVOptions
 * This section describes how to add AVOptions capabilities to a struct.
 *
 * All AVOptions-related information is stored in an AVClass. Therefore
 * the first member of the struct should be a pointer to an AVClass describing it.
 * The option field of the AVClass must be set to a NULL-terminated static array
 * of AVOptions. Each AVOption must have a non-empty name, a type, a default
 * value and for number-type AVOptions also a range of allowed values. It must
 * also declare an offset in bytes from the start of the struct, where the field
 * associated with this AVOption is located. Other fields in the AVOption struct
 * should also be set when applicable, but are not required.
 *
 * The following example illustrates an AVOptions-enabled struct:
 * @code
 * typedef struct test_struct {
 *     const AVClass *class;
 *     int      int_opt;
 *     char    *str_opt;
 *     uint8_t *bin_opt;
 *     int      bin_len;
 * } test_struct;
 *
 * static const AVOption test_options[] = {
 *   { "test_int", "This is a test option of int type.", offsetof(test_struct, int_opt),
 *     AV_OPT_TYPE_INT, { .i64 = -1 }, INT_MIN, INT_MAX },
 *   { "test_str", "This is a test option of string type.", offsetof(test_struct, str_opt),
 *     AV_OPT_TYPE_STRING },
 *   { "test_bin", "This is a test option of binary type.", offsetof(test_struct, bin_opt),
 *     AV_OPT_TYPE_BINARY },
 *   { NULL },
 * };
 *
 * static const AVClass test_class = {
 *     .class_name = "test class",
 *     .item_name  = av_default_item_name,
 *     .option     = test_options,
 *     .version    = LIBAVUTIL_VERSION_INT,
 * };
 * @endcode
 *
 * Next, when allocating your struct, you must ensure that the AVClass pointer
 * is set to the correct value. Then, av_opt_set_defaults() can be called to
 * initialize defaults. After that the struct is ready to be used with the
 * AVOptions API.
 *
 * When cleaning up, you may use the av_opt_free() function to automatically
 * free all the allocated string and binary options.
 *
 * Continuing with the above example:
 *
 * @code
 * test_struct *alloc_test_struct(void)
 * {
 *     test_struct *ret = av_mallocz(sizeof(*ret));
 *     ret->class = &test_class;
 *     av_opt_set_defaults(ret);
 *     return ret;
 * }
 * void free_test_struct(test_struct **foo)
 * {
 *     av_opt_free(*foo);
 *     av_freep(foo);
 * }
 * @endcode
 *
 * @subsection avoptions_implement_nesting Nesting
 *      It may happen that an AVOptions-enabled struct contains another
 *      AVOptions-enabled struct as a member (e.g. AVCodecContext in
 *      libavcodec exports generic options, while its priv_data field exports
 *      codec-specific options). In such a case, it is possible to set up the
 *      parent struct to export a child's options. To do that, simply
 *      implement AVClass.child_next() and AVClass.child_class_next() in the
 *      parent struct's AVClass.
 *      Assuming that the test_struct from above now also contains a
 *      child_struct field:
 *
 *      @code
 *      typedef struct child_struct {
 *          AVClass *class;
 *          int flags_opt;
 *      } child_struct;
 *      static const AVOption child_opts[] = {
 *          { "test_flags", "This is a test option of flags type.",
 *            offsetof(child_struct, flags_opt), AV_OPT_TYPE_FLAGS, { .i64 = 0 }, INT_MIN, INT_MAX },
 *          { NULL },
 *      };
 *      static const AVClass child_class = {
 *          .class_name = "child class",
 *          .item_name  = av_default_item_name,
 *          .option     = child_opts,
 *          .version    = LIBAVUTIL_VERSION_INT,
 *      };
 *
 *      void *child_next(void *obj, void *prev)
 *      {
 *          test_struct *t = obj;
 *          if (!prev && t->child_struct)
 *              return t->child_struct;
 *          return NULL
 *      }
 *      const AVClass child_class_next(const AVClass *prev)
 *      {
 *          return prev ? NULL : &child_class;
 *      }
 *      @endcode
 *      Putting child_next() and child_class_next() as defined above into
 *      test_class will now make child_struct's options accessible through
 *      test_struct (again, proper setup as described above needs to be done on
 *      child_struct right after it is created).
 *
 *      From the above example it might not be clear why both child_next()
 *      and child_class_next() are needed. The distinction is that child_next()
 *      iterates over actually existing objects, while child_class_next()
 *      iterates over all possible child classes. E.g. if an AVCodecContext
 *      was initialized to use a codec which has private options, then its
 *      child_next() will return AVCodecContext.priv_data and finish
 *      iterating. OTOH child_class_next() on AVCodecContext.av_class will
 *      iterate over all available codecs with private options.
 *
 * @subsection avoptions_implement_named_constants Named constants
 *      It is possible to create named constants for options. Simply set the unit
 *      field of the option the constants should apply to a string and
 *      create the constants themselves as options of type AV_OPT_TYPE_CONST
 *      with their unit field set to the same string.
 *      Their default_val field should contain the value of the named
 *      constant.
 *      For example, to add some named constants for the test_flags option
 *      above, put the following into the child_opts array:
 *      @code
 *      { "test_flags", "This is a test option of flags type.",
 *        offsetof(child_struct, flags_opt), AV_OPT_TYPE_FLAGS, { .i64 = 0 }, INT_MIN, INT_MAX, "test_unit" },
 *      { "flag1", "This is a flag with value 16", 0, AV_OPT_TYPE_CONST, { .i64 = 16 }, 0, 0, "test_unit" },
 *      @endcode
 *
 * @section avoptions_use Using AVOptions
 * This section deals with accessing options in an AVOptions-enabled struct.
 * Such structs in FFmpeg are e.g. AVCodecContext in libavcodec or
 * AVFormatContext in libavformat.
 *
 * @subsection avoptions_use_examine Examining AVOptions
 * The basic functions for examining options are av_opt_next(), which iterates
 * over all options defined for one object, and av_opt_find(), which searches
 * for an option with the given name.
 *
 * The situation is more complicated with nesting. An AVOptions-enabled struct
 * may have AVOptions-enabled children. Passing the AV_OPT_SEARCH_CHILDREN flag
 * to av_opt_find() will make the function search children recursively.
 *
 * For enumerating there are basically two cases. The first is when you want to
 * get all options that may potentially exist on the struct and its children
 * (e.g.  when constructing documentation). In that case you should call
 * av_opt_child_class_next() recursively on the parent struct's AVClass.  The
 * second case is when you have an already initialized struct with all its
 * children and you want to get all options that can be actually written or read
 * from it. In that case you should call av_opt_child_next() recursively (and
 * av_opt_next() on each result).
 *
 * @subsection avoptions_use_get_set Reading and writing AVOptions
 * When setting options, you often have a string read directly from the
 * user. In such a case, simply passing it to av_opt_set() is enough. For
 * non-string type options, av_opt_set() will parse the string according to the
 * option type.
 *
 * Similarly av_opt_get() will read any option type and convert it to a string
 * which will be returned. Do not forget that the string is allocated, so you
 * have to free it with av_free().
 *
 * In some cases it may be more convenient to put all options into an
 * AVDictionary and call av_opt_set_dict() on it. A specific case of this
 * are the format/codec open functions in lavf/lavc which take a dictionary
 * filled with option as a parameter. This makes it possible to set some options
 * that cannot be set otherwise, since e.g. the input file format is not known
 * before the file is actually opened.
 */

enum AVOptionType{
    AV_OPT_TYPE_FLAGS,
    AV_OPT_TYPE_INT,
    AV_OPT_TYPE_INT64,
    AV_OPT_TYPE_DOUBLE,
    AV_OPT_TYPE_FLOAT,
    AV_OPT_TYPE_STRING,
    AV_OPT_TYPE_RATIONAL,
    AV_OPT_TYPE_BINARY,  ///< offset must point to a pointer immediately followed by an int for the length
    AV_OPT_TYPE_DICT,
    AV_OPT_TYPE_UINT64,
    AV_OPT_TYPE_CONST = 128,
    AV_OPT_TYPE_IMAGE_SIZE = MKBETAG('S','I','Z','E'), ///< offset must point to two consecutive integers
    AV_OPT_TYPE_PIXEL_FMT  = MKBETAG('P','F','M','T'),
    AV_OPT_TYPE_SAMPLE_FMT = MKBETAG('S','F','M','T'),
    AV_OPT_TYPE_VIDEO_RATE = MKBETAG('V','R','A','T'), ///< offset must point to AVRational
    AV_OPT_TYPE_DURATION   = MKBETAG('D','U','R',' '),
    AV_OPT_TYPE_COLOR      = MKBETAG('C','O','L','R'),
    AV_OPT_TYPE_CHANNEL_LAYOUT = MKBETAG('C','H','L','A'),
    AV_OPT_TYPE_BOOL           = MKBETAG('B','O','O','L'),
};

**
 * AVOption
 */
typedef struct AVOption {
    const char *name;

    /**
     * short English help text
     * @todo What about other languages?
     */
    const char *help;

    /**
     * The offset relative to the context structure where the option
     * value is stored. It should be 0 for named constants.
     */
    int offset;
    enum AVOptionType type;

    /**
     * the default value for scalar options
     */
    union {
        int64_t i64;
        double dbl;
        const char *str;
        /* TODO those are unused now */
        AVRational q;
    } default_val;
    double min;                 ///< minimum valid value for the option
    double max;                 ///< maximum valid value for the option

    int flags;
#define AV_OPT_FLAG_ENCODING_PARAM  1   ///< a generic parameter which can be set by the user for muxing or encoding
#define AV_OPT_FLAG_DECODING_PARAM  2   ///< a generic parameter which can be set by the user for demuxing or decoding
#if FF_API_OPT_TYPE_METADATA
#define AV_OPT_FLAG_METADATA        4   ///< some data extracted or inserted into the file like title, comment, ...
#endif
#define AV_OPT_FLAG_AUDIO_PARAM     8
#define AV_OPT_FLAG_VIDEO_PARAM     16
#define AV_OPT_FLAG_SUBTITLE_PARAM  32
/**
 * The option is intended for exporting values to the caller.
 */
#define AV_OPT_FLAG_EXPORT          64
/**
 * The option may not be set through the AVOptions API, only read.
 * This flag only makes sense when AV_OPT_FLAG_EXPORT is also set.
 */
#define AV_OPT_FLAG_READONLY        128
#define AV_OPT_FLAG_FILTERING_PARAM (1<<16) ///< a generic parameter which can be set by the user for filtering
//FIXME think about enc-audio, ... style flags

    /**
     * The logical unit to which the option belongs. Non-constant
     * options and corresponding named constants share the same
     * unit. May be NULL.
     */
    const char *unit;
} AVOption;
```