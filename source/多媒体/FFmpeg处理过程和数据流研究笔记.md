# FFmpeg处理过程和数据流研究笔记

***注：本文引用的ffmpeg相关代码基于4.4版本。***

## FFmpeg解码

ffmpeg的HEVC解码过程分为三个步骤：
1. 发送码流：将待解码码流数据AVPacket输入给解码器
2. 解码：解码器进行解码，将AVPacket数据解码重建为视频帧
3. 获取解码帧：从解码器中获取已完成解码的图像帧，即AVFrame

本文只分析和记录发送AVPacket和获取AVFrame两部分的主要函数调用过程和数据流转路径。

### 主要结构体关系

HEVC解码涉及的主要结构体关系如下：

```
AVCodecContext   *avctx = (HEVCContext*)s->avctx
AVCodecContext   *avctx = (PerThreadContext*)p->avctx;
AVCodecInternal   *avci = avctx->internal;
DecodeSimpleContext *ds = &avci->ds;
AVPacket           *pkt = ds->in_pkt;
AVPacket           *pkt = avci->buffer_pkt

// Thread Pool:
PerThreadContext *p      = avctx->internal->thread_ctx;
PerThreadContext *p      = &fctx->threads[fctx->next_decoding];
FrameThreadContext *fctx = avctx->internal->thread_ctx;
```

### 发送码流

ffmpeg解码的**主线程**主要涉及ffmpeg的解码框架和码流数据的通用处理，不涉及特定的视频编码标准和具体的解码器实现，其调用解码器和输入待解码码流数据的调用栈为：

```
#0  0x00007ffff3f20a35 in pthread_cond_wait@@GLIBC_2.3.2 () from /lib64/libpthread.so.0
#1  0x00007ffff58a8d03 in submit_packet (p=0x652c20, user_avctx=0x683680, avpkt=0x651d80) at libavcodec/pthread_frame.c:438
#2  0x00007ffff58a905c in ff_thread_decode_frame (avctx=0x683680, picture=0x651780, got_picture_ptr=0x7fffffffd57c, avpkt=0x651d80) at libavcodec/pthread_frame.c:523
#3  0x00007ffff533af58 in decode_simple_internal (avctx=0x683680, frame=0x651780, discarded_samples=0x7fffffffd630) at libavcodec/decode.c:325
#4  0x00007ffff533bbc9 in decode_simple_receive_frame (avctx=0x683680, frame=0x651780) at libavcodec/decode.c:526
#5  0x00007ffff533bcb5 in decode_receive_frame_internal (avctx=0x683680, frame=0x651780) at libavcodec/decode.c:546
#6  0x00007ffff533bf3c in avcodec_send_packet (avctx=0x683680, avpkt=0x8496c0) at libavcodec/decode.c:608
#7  0x0000000000430136 in decode (avctx=0x683680, frame=0x849780, got_frame=0x7fffffffd8d4, pkt=0x8496c0) at fftools/ffmpeg.c:2285
#8  0x0000000000430882 in decode_video (ist=0x65fb40, pkt=0x8496c0, got_output=0x7fffffffd8d4, duration_pts=0x7fffffffd8d8, eof=0, decode_failed=0x7fffffffd8d0) at fftools/ffmpeg.c:2425
#9  0x0000000000431875 in process_input_packet (ist=0x65fb40, pkt=0x65fac0, no_eof=0) at fftools/ffmpeg.c:2672
#10 0x0000000000438e4e in process_input (file_index=0) at fftools/ffmpeg.c:4655
#11 0x000000000043932a in transcode_step () at fftools/ffmpeg.c:4795
#12 0x0000000000439460 in transcode () at fftools/ffmpeg.c:4849
#13 0x0000000000439c93 in main (argc=9, argv=0x7fffffffe118) at fftools/ffmpeg.c:5054
```

其中，`transcode()`为音视频文件转码和处理的主循环函数，最重要的功能是循环调用`transcode_step()`，直到文件遇到EOF或用户按下`CTRL-C`中断程序。
```
    while (!received_sigterm) {
        ret = transcode_step();
        if (ret < 0 && ret != AVERROR_EOF) {
            break;
        }
    }

```

`transcode_step()`先选择OutputStream，配置filtergraph，然后调用`process_input()`和`reap_filters()`解码视频和应用filter进行滤镜处理

```
static int transcode_step(void)
{
    OutputStream *ost;

    ost = choose_output();

    if (ost->filter && !ost->filter->graph->graph) {
        if (ifilter_has_all_input_formats(ost->filter->graph)) {
            ret = configure_filtergraph(ost->filter->graph);
        }
    }

    ret = process_input(ist->file_index);
    if (ret == AVERROR(EAGAIN))
        return 0;

    if (ret < 0)
        return ret == AVERROR_EOF ? 0 : ret;

    return reap_filters(0);
}
```

`process_input()`从`InputStream`列表中根据`file_index`获取文件，从文件中读取一个packet，即一帧frame的码流数据，然后调用`process_input_packet()`进一步解码

```
static int process_input(int file_index)
{
    ret = get_input_packet(ifile, &pkt);

    process_input_packet(ist, pkt, 0);
}
```

`process_input_packet()`根据媒体类型分类处理，分别调用对应的decode或transcode函数

```
static int process_input_packet(InputStream *ist, const AVPacket *pkt, int no_eof)
{
    while (ist->decoding_needed) {
        switch (ist->dec_ctx->codec_type) {
        case AVMEDIA_TYPE_AUDIO:
            ret = decode_audio(ist, repeating ? NULL : avpkt, &got_output, &decode_failed);
            break;
        case AVMEDIA_TYPE_VIDEO:
            ret = decode_video(ist, repeating ? NULL : avpkt, &got_output, &duration_pts, !pkt, &decode_failed);
            break;
        case AVMEDIA_TYPE_SUBTITLE:
            ret = transcode_subtitles(ist, avpkt, &got_output, &decode_failed);
            break;
        default:
            return -1;
        }
    }

    if (!pkt && ist->decoding_needed && eof_reached && !no_eof) {
        int ret = send_filter_eof(ist);
    }
}
```

HEVC解码的**从线程**开始涉及具体的视频编码标准和解码器实现，其调用栈为：

```
#0  hevc_decode_frame (avctx=0x654100, data=0x654540, got_output=0x652bd0, avpkt=0x654800) at libavcodec/hevcdec.c:3360
#1  0x00007ffff58a83b7 in frame_worker_thread (arg=0x652ac0) at libavcodec/pthread_frame.c:219
#2  0x00007ffff3f1cea5 in start_thread () from /lib64/libpthread.so.0
#3  0x00007ffff3c459fd in clone () from /lib64/libc.so.6
```

`hevc_decode_frame()`从参数中获得外部传入的packet的指针，主要是调用`decode_nal_units()`函数，此时会将`pkt->data`和`pkt->size`变量传入函数。一般而言一个packet对应一个HEVC标准中定义的Access Unit，`decode_nal_units()`会将Access Unit拆分为多个NALU进行分别解码，其主要流程为：

```
static int decode_nal_units(HEVCContext *s, const uint8_t *buf, int length)
{
    // 将packet拆分为单独的NAL Unit
    ret = ff_h2645_packet_split(&s->pkt, buf, length, s->avctx, s->is_nalff,
                                s->nal_length_size, s->avctx->codec_id, 1, 0);

    for (i = 0; i < s->pkt.nb_nals; i++) {
        H2645NAL *nal = &s->pkt.nals[i];
        ret = decode_nal_unit(s, nal);
    }
}
```

`decode_nal_unit()`根据NALU的type进行分类处理：

```
static int decode_nal_unit(HEVCContext *s, const H2645NAL *nal)
{
    case HEVC_NAL_VPS:
        ret = ff_hevc_decode_nal_vps(gb, s->avctx, &s->ps);
    case HEVC_NAL_SPS:
        ret = ff_hevc_decode_nal_sps(gb, s->avctx, &s->ps, s->apply_defdispwin);
    case HEVC_NAL_PPS:
        ret = ff_hevc_decode_nal_pps(gb, s->avctx, &s->ps);
    case HEVC_NAL_SEI_PREFIX:
    case HEVC_NAL_SEI_SUFFIX:
        ret = ff_hevc_decode_nal_sei(gb, s->avctx, &s->sei, &s->ps, s->nal_unit_type);
    case HEVC_NAL_TRAIL_R:
    case HEVC_NAL_TRAIL_N:
    case HEVC_NAL_TSA_N:
    case HEVC_NAL_TSA_R:
    case HEVC_NAL_STSA_N:
    case HEVC_NAL_STSA_R:
    case HEVC_NAL_BLA_W_LP:
    case HEVC_NAL_BLA_W_RADL:
    case HEVC_NAL_BLA_N_LP:
    case HEVC_NAL_IDR_W_RADL:
    case HEVC_NAL_IDR_N_LP:
    case HEVC_NAL_CRA_NUT:
    case HEVC_NAL_RADL_N:
    case HEVC_NAL_RADL_R:
    case HEVC_NAL_RASL_N:
    case HEVC_NAL_RASL_R:
        ret = hls_slice_header(s);

        if (s->sh.first_slice_in_pic_flag)
            static int hevc_frame_start(HEVCContext *s)

        if (s->avctx->hwaccel) {
            ret = s->avctx->hwaccel->decode_slice(s->avctx, nal->raw_data, nal->raw_size);
        } else {
            if (s->threads_number > 1 && s->sh.num_entry_point_offsets > 0)
                ctb_addr_ts = hls_slice_data_wpp(s, nal);
            else
                ctb_addr_ts = hls_slice_data(s);
                    -> s->avctx->execute(s->avctx, hls_decode_entry, arg, ret , 1, sizeof(int));
        }
}
```

对于HEVC解码的场景，视频压缩数据经历了`文件->stream->packet`的结构转变和数据流转，具体流转路径按从先到后、从上层到下层的顺序总结归纳如下:

```
1. Input File
2. (InputFile*)ifile                                     : @ process_input > get_input_packet;
3. (InputStream*)ist->pkt                                : @ process_input_packet > decode_video > decode
4. (AVPacket*)avctx->internal->buffer_pkt                : @ avcodec_send_packet
5. (AVPacket*)avctx->internal->bsf->internal->buffer_pkt : @ avcodec_send_packet > av_bsf_send_packet | decode_receive_frame_internal > decode_simple_receive_frame
6. (AVPacket*)avctx->internal->ds.in_pkt                 : @ decode_simple_internal > ff_decode_get_packet > av_bsf_receive_packet > bsf_list_filter > ff_bsf_get_packet_ref
7. (AVPacket*)PerThreadContext->avpkt                    : @ submit_packet  // 这里提交packet后会unref avctx->internal->ds.in_pkt
8. (uint8_t*)avpkt->data                                 : @ frame_worker_thread > codec->decode=hevc_decode_frame > decode_nal_units > decode_nal_unit
```

其中，每一行的格式为`(数据类型) 变量存储位置: @发生数据流转的函数和被调用路径`，`@`右边的`>`表示左边的函数进一步调用了右边的函数，数据流转发生在最右边的函数内部，`=`表示左边为函数指针，实际指向右边的函数，`|`表示存在多种调用路径的可能性。

### 获取解码帧

ffmpeg获取解码后的视频帧AVFrame的调用栈与发送待解码码流数据AVPacket的调用栈基本相同：

```
#0  0x00007ffff58a905c in ff_thread_decode_frame (avctx=0x683680, picture=0x651780, got_picture_ptr=0x7fffffffd57c, avpkt=0x651d80) at libavcodec/pthread_frame.c:523
#1  0x00007ffff533af58 in decode_simple_internal (avctx=0x683680, frame=0x651780, discarded_samples=0x7fffffffd630) at libavcodec/decode.c:325
#2  0x00007ffff533bbc9 in decode_simple_receive_frame (avctx=0x683680, frame=0x651780) at libavcodec/decode.c:526
#3  0x00007ffff533bcb5 in decode_receive_frame_internal (avctx=0x674ec0, frame=0x655240) at libavcodec/decode.c:546
#4  0x00007ffff53519e9 in avcodec_send_packet (avctx=0x674ec0, avpkt=0x7a6580) at libavcodec/decode.c:608
#5  0x00000000004304c4 in decode (avctx=0x674ec0, frame=0x7a66c0, got_frame=0x7fffffffd634, pkt=0x7a6580) at fftools/ffmpeg.c:2285
#6  0x0000000000430c8d in decode_video (ist=0x673c40, pkt=0x7a6580, got_output=0x7fffffffd634, duration_pts=0x7fffffffd638, eof=0, decode_failed=0x7fffffffd630) at fftools/ffmpeg.c:2425
#7  0x0000000000431cc8 in process_input_packet (ist=0x673c40, pkt=0x898340, no_eof=0) at fftools/ffmpeg.c:2672
#8  0x00000000004394df in process_input (file_index=0) at fftools/ffmpeg.c:4655
#9  0x00000000004399b9 in transcode_step () at fftools/ffmpeg.c:4795
#10 0x0000000000439aec in transcode () at fftools/ffmpeg.c:4849
#11 0x000000000043a3da in main (argc=9, argv=0x7fffffffdd68) at fftools/ffmpeg.c:5054
```

对于HEVC解码的场景，解码后的视频帧具体流转路径按从上层到下层的顺序总结归纳如下:

```
1. (AVFrame*)Decoded Frame                                    : @ decode > avcodec_send_packet | avcodec_receive_frame
2. (AVFrame*)avctx->internal->buffer_frame                    : @ avcodec_send_packet > decode_receive_frame_internal > decode_simple_receive_frame > decode_simple_internal > ff_thread_decode_frame
3. (AVFrame*)PerThreadContext->frame                          : @ frame_worker_thread > codec->decode=hevc_decode_frame
4. (AVFrame*)PerThreadContext->frame                          : @ hevc_decode_frame > decode_nal_units > decode_nal_unit > hevc_frame_start
5. (AVFrame*)(((HEVCContext*)avctx->priv_data)->output_frame) : @ ff_hevc_output_frame
```

其中，每一行的格式为`(数据类型) 变量存储位置: @发生数据流转的函数和被调用路径`，`@`右边的`>`表示左边的函数进一步调用了右边的函数，数据流转发生在最右边的函数内部，`=`表示左边为函数指针，实际指向右边的函数，`|`表示存在多种调用路径的可能性。

第5层中，`ff_hevc_output_frame()`从DPB中获取可输出的AVFrame，存放在`((HEVCContext*)avctx->priv_data)->output_frame`中。

第4层中，`hevc_decode_frame()`将`s->output_frame`移动到函数形参`data`中，具体主要代码为：

```
    if (s->output_frame->buf[0]) {
        av_frame_move_ref(data, s->output_frame);
        *got_output = 1;
    }
```

第3层中，`frame_worker_thread()`为**从线程**入口函数，将`PerThreadContext->frame`设置为`hevc_decode_frame()`的`data`形参的实参。

第2层中，`ff_thread_decode_frame()`使用while循环检测解码完成状态，将`PerThreadContext->frame`移动到形参`picture`中，相关主要代码为：

```
    do {
        p = &fctx->threads[finished++];

        if (atomic_load(&p->state) != STATE_INPUT_READY) {
            pthread_mutex_lock(&p->progress_mutex);
            while (atomic_load_explicit(&p->state, memory_order_relaxed) != STATE_INPUT_READY)
                pthread_cond_wait(&p->output_cond, &p->progress_mutex);
            pthread_mutex_unlock(&p->progress_mutex);
        }

        av_frame_move_ref(picture, p->frame);
    } while (!avpkt->size && !*got_picture_ptr && err >= 0 && finished != fctx->next_finished);
```

第2层中的`avcodec_send_packet()`在调用`decode_receive_frame_internal()`时将`picture`形参设置为`avctx->internal->buffer_frame`实参。


### FFmpeg试解码

```
#0  decode_simple_internal (avctx=0x64f6c0, frame=0x650880, discarded_samples=0x7fffffffd140) at libavcodec/decode.c:299
#1  0x00007ffff5351673 in decode_simple_receive_frame (avctx=0x64f6c0, frame=0x650880) at libavcodec/decode.c:526
#2  0x00007ffff535175f in decode_receive_frame_internal (avctx=0x64f6c0, frame=0x650880) at libavcodec/decode.c:546
#3  0x00007ffff53519e9 in avcodec_send_packet (avctx=0x64f6c0, avpkt=0x7fffffffd200) at libavcodec/decode.c:608
#4  0x00007ffff6c031ba in try_decode_frame (s=0x64dd00, st=0x64ebc0, avpkt=0x66ed80, options=0x64e6c0) at libavformat/utils.c:3087
#5  0x00007ffff6c0691c in avformat_find_stream_info (ic=0x64dd00, options=0x64e6c0) at libavformat/utils.c:3954
#6  0x000000000040dd21 in open_input_file (o=0x7fffffffd6d0, filename=0x7fffffffe093 "WenXin01.c00471ecguf.322150.1_track1.hvc") at fftools/ffmpeg_opt.c:1196
#7  0x000000000041c065 in open_files (l=0x64d958, inout=0x43d097 "input", open_file=0x40d274 <open_input_file>) at fftools/ffmpeg_opt.c:3338
#8  0x000000000041c1db in ffmpeg_parse_options (argc=9, argv=0x7fffffffdd68) at fftools/ffmpeg_opt.c:3378
#9  0x000000000043a2b9 in main (argc=9, argv=0x7fffffffdd68) at fftools/ffmpeg.c:5032
```

## FFmpeg编码

FFmpeg的编码调用栈为：

```
#0  libt265_encode_frame (avctx=0x655640, pkt=0x7d5400, pic=0x7d54c0, got_packet=0x7fffffffd65c) at libavcodec/libt265.c:322
#1  0x00007ffff53bb62d in encode_simple_internal (avctx=0x655640, avpkt=0x7d5400) at libavcodec/encode.c :214
#2  0x00007ffff53bb90b in encode_simple_receive_packet (avctx=0x655640, avpkt=0x7d5400) at libavcodec/encode.c:275
#3  0x00007ffff53bbaf2 in encode_receive_packet_internal (avctx=0x655640, avpkt=0x7d5400) at libavcodec/encode.c:309
#4  0x00007ffff53bbdff in avcodec_send_frame (avctx=0x655640, frame=0x7d46c0) at libavcodec/encode.c:387
#5  0x000000000042c9df in do_video_out (of=0x670000, ost=0x650c80, next_picture=0x7d46c0) at fftools/ffmpeg.c:1367
#6  0x000000000042d62b in reap_filters (flush=0) at fftools/ffmpeg.c:1562
#7  0x0000000000439a1f in transcode_step () at fftools/ffmpeg.c:4805
#8  0x0000000000439aec in transcode () at fftools/ffmpeg.c:4849
#9  0x000000000043a3da in main (argc=15, argv=0x7fffffffdca8) at fftools/ffmpeg.c:5054
```

FFmpeg的编码器输入帧的原始来源为解码器或filter的输出，与编码器之间通过AVLink链接，获取输入的待编码帧的调用栈为：

```
#2  0x00007ffff7014bc4 in get_frame_internal (ctx=0x7d3e00, frame=0x7d46c0, flags=2, samples=0) at libavfilter/buffersink.c:125
#3  0x00007ffff7014c77 in av_buffersink_get_frame_flags (ctx=0x7d3e00, frame=0x7d46c0, flags=2) at libavfilter/buffersink.c:142
#4  0x000000000042d4d5 in reap_filters (flush=0) at fftools/ffmpeg.c:1540
#5  0x0000000000439a1f in transcode_step () at fftools/ffmpeg.c:4805
#6  0x0000000000439aec in transcode () at fftools/ffmpeg.c:4849
#7  0x000000000043a3da in main (argc=15, argv=0x7fffffffdca8) at fftools/ffmpeg.c:5054
```

对于HEVC编码的场景，具体流转路径按从先到后、从上层到下层的顺序总结归纳如下:

```
. (AVFrame*)cur_frame                                   : @get_frame_internal > ff_inlink_consume_frame > ff_framequeue_take
. (AVFrame*)frame                                       : @get_frame_internal > return_or_keep_frame > av_frame_move_ref
. (AVFrame*)frame                                       : @av_buffersink_get_frame_flags
. (AVFrame*)filtered_frame                              : @reap_filters > av_buffersink_get_frame_flags
. (AVFrame*)filtered_frame                              : @reap_filters > do_video_out
. (AVFrame*)in_picture                                  : @do_video_out
. (AVFrame*)in_picture                                  : @do_video_out > avcodec_send_frame
. (AVFrame*)frame                                       : @avcodec_send_frame > encode_send_frame_internal
. (AVFrame*)avctx->internal->buffer_frame               : @encode_send_frame_internal
. (AVFrame*)avctx->internal->es.in_frame                : @avcodec_send_frame > encode_receive_packet_internal > encode_simple_receive_packet > encode_simple_internal > ff_encode_get_frame
. (AVFrame*)pic                                         : @encode_simple_internal > avctx->codec->encode2 = libt265_encode_frame
```

在`get_frame_internal()`中，先通过`ff_inlink_consume_frame > ff_framequeue_take`调用路径获取当前帧存储到局部变量`cur_frame`，再通过`return_or_keep_frame > av_frame_move_ref`调用路径将`cur_frame`移动到形参`frame`中。

## FFmpeg Filter

Filter的activate函数的调用栈为：

```
#0  0x00007ffff7290ec4 in activate (ctx=0x18976c0) at libavfilter/vf_sdr2hdr.c:256
#1  0x00007ffff7005bf5 in ff_filter_activate (filter=0x18976c0) at libavfilter/avfilter.c:1440
#2  0x00007ffff700a6d9 in ff_filter_graph_run_once (graph=0x1894600) at libavfilter/avfiltergraph.c:1403
#3  0x00007ffff700c354 in push_frame (graph=0x1894600) at libavfilter/buffersrc.c:157
#4  0x00007ffff700c894 in av_buffersrc_add_frame_flags (ctx=0x1891300, frame=0x849780, flags=4) at libavfilter/buffersrc.c:225
#5  0x000000000042ff9b in ifilter_send_frame (ifilter=0x682a80, frame=0x849780) at fftools/ffmpeg.c:2241
#6  0x000000000043021e in send_frame_to_filters (ist=0x65fb40, decoded_frame=0x849780) at fftools/ffmpeg.c:2315
#7  0x0000000000430ef2 in decode_video (ist=0x65fb40, pkt=0x8496c0, got_output=0x7fffffffd8d4, duration_pts=0x7fffffffd8d8, eof=0, decode_failed=0x7fffffffd8d0) at fftools/ffmpeg.c:2512
#8  0x0000000000431875 in process_input_packet (ist=0x65fb40, pkt=0x65fac0, no_eof=0) at fftools/ffmpeg.c:2672
#9  0x0000000000438e4e in process_input (file_index=0) at fftools/ffmpeg.c:4655
#10 0x000000000043932a in transcode_step () at fftools/ffmpeg.c:4795
#11 0x0000000000439460 in transcode () at fftools/ffmpeg.c:4849
#12 0x0000000000439c93 in main (argc=9, argv=0x7fffffffe118) at fftools/ffmpeg.c:5054
```

Filter将AVFrame推送到下一级filter或encoder的调用栈为：

```
#0  ff_framequeue_add (fq=0x7d43b0, frame=0x6c3a40) at libavfilter/framequeue.c :67
#1  0x00007ffff700ebcb in ff_filter_frame (link=0x7d42c0, frame=0x6c3a40) at libavfilter/avfilter.c:1132
#2  0x00007ffff700e844 in default_filter_frame (link=0x7d3cc0, frame=0x6c3a40) at libavfilter/avfilter.c :1060
#3  0x00007ffff700e936 in ff_filter_frame_framed (link=0x7d3cc0, frame=0x6c3a40) at libavfilter/avfilter.c:1085
#4  0x00007ffff700efea in ff_filter_frame_to_filter (link=0x7d3cc0) at libavfilter/avfilter.c:1233
#5  0x00007ffff700f1f2 in ff_filter_activate_default (filter=0x7d2d00) at libavfilter/avfilter.c:1282
#6  0x00007ffff700f358 in ff_filter_activate (filter=0x7d2d00) at libavfilter/avfilter.c:1440
#7  0x00007ffff7013ee9 in ff_filter_graph_run_once (graph=0x7cfc80) at libavfilter/avfiltergraph.c:1403
#8  0x00007ffff7015b10 in push_frame (graph=0x7cfc80) at libavfilter/buffersrc.c:157
#9  0x00007ffff7016036 in av_buffersrc_add_frame_flags (ctx=0x7d3280, frame=0x694e00, flags=4) at libavfilter/buffersrc.c:225
#10 0x00000000004302fc in ifilter_send_frame (ifilter=0x655900, frame=0x694e00) at fftools/ffmpeg.c:2241
#11 0x00000000004305ac in send_frame_to_filters (ist=0x6505c0, decoded_frame=0x694e00) at fftools/ffmpeg.c:2315
#12 0x00000000004312f7 in decode_video (ist=0x6505c0, pkt=0x668b40, got_output=0x7fffffffd544, duration_pts=0x7fffffffd548, eof=0, decode_failed=0x7fffffffd540) at fftools/ffmpeg.c:2512
#13 0x0000000000431cc8 in process_input_packet (ist=0x6505c0, pkt=0x898280, no_eof=0) at fftools/ffmpeg.c:2672
#14 0x00000000004394df in process_input (file_index=0) at fftools/ffmpeg.c:4655
#15 0x00000000004399b9 in transcode_step () at fftools/ffmpeg.c:4795
#16 0x0000000000439aec in transcode () at fftools/ffmpeg.c:4849
#17 0x000000000043a3da in main (argc=17, argv=0x7fffffffdc78) at fftools/ffmpeg.c:5054
```

## FFmpeg Bitstream Filter

以`h265_metadata_bsf`为例，初始化时的调用栈为：

```
#0  h265_metadata_update_fragment (bsf=0x64fa40, pkt=0x0, au=0x660de0) at libavcodec/h265_metadata_bsf.c:334
#1  0x00007ffff5269827 in ff_cbs_bsf_generic_init (bsf=0x64fa40, type=0x7ffff63490c0 <h265_metadata_type>) at libavcodec/cbs_bsf.c:135
#2  0x00007ffff55b9095 in h265_metadata_init (bsf=0x64fa40) at libavcodec/h265_metadata_bsf.c:408
#3  0x00007ffff52325e7 in av_bsf_init (ctx=0x64fa40) at libavcodec/bsf.c:181
#4  0x0000000000433352 in init_output_bsfs (ost=0x6601c0) at fftools/ffmpeg.c:3088
#5  0x00000000004357d2 in init_output_stream (ost=0x6601c0, frame=0x0, error=0x7fffffffd1e0 "", error_len=1024) at fftools/ffmpeg.c:3745
#6  0x000000000042b0a5 in init_output_stream_wrapper (ost=0x6601c0, frame=0x0, fatal=0) at fftools/ffmpeg.c:992
#7  0x0000000000435c89 in transcode_init () at fftools/ffmpeg.c:3829
#8  0x0000000000439a20 in transcode () at fftools/ffmpeg.c:4820
#9  0x000000000043a3bb in main (argc=11, argv=0x7fffffffdd38) at fftools/ffmpeg.c:5054
```

正常处理时的调用栈为：

```
#0  h265_metadata_update_fragment (bsf=0x64fa40, pkt=0x661d40, au=0x660de0) at libavcodec/h265_metadata_bsf.c:334
#1  0x00007ffff5269692 in ff_cbs_bsf_generic_filter (bsf=0x64fa40, pkt=0x661d40) at libavcodec/cbs_bsf.c:91
#2  0x00007ffff5232770 in av_bsf_receive_packet (ctx=0x64fa40, pkt=0x661d40) at libavcodec/bsf.c:229
#3  0x000000000042aad7 in output_packet (of=0x684e80, pkt=0x661d40, ost=0x6601c0, eof=0) at fftools/ffmpeg.c:907
#4  0x000000000042fbfb in do_streamcopy (ist=0x660ac0, ost=0x6601c0, pkt=0x69c300) at fftools/ffmpeg.c:2121
#5  0x0000000000432587 in process_input_packet (ist=0x660ac0, pkt=0x69c300, no_eof=0) at fftools/ffmpeg.c:2799
#6  0x00000000004394c0 in process_input (file_index=0) at fftools/ffmpeg.c:4655
#7  0x000000000043999a in transcode_step () at fftools/ffmpeg.c:4795
#8  0x0000000000439acd in transcode () at fftools/ffmpeg.c:4849
#9  0x000000000043a3bb in main (argc=11, argv=0x7fffffffdd38) at fftools/ffmpeg.c:5054
```

相关的主要结构体和变量的流转路径如下：

```
(OutputStream**) output_streams                                         : @ global variables of ffmpeg.c
(OutputStream*) ost                                                     : @ process_input_packet
(AVBSFContext*) ost->bsf_ctx                                            : @ do_streamcopy > output_packet
(CBSBSFContext*) ost->bsf_ctx->priv_data                                : @ av_bsf_receive_packet > ff_cbs_bsf_generic_filter
(CodedBitstreamContext*) ost->bsf_ctx->priv_data->input                 : @ ff_cbs_bsf_generic_filter
(CodedBitstreamH2645Context*) ost->bsf_ctx->priv_data->input->priv_data : @ ff_cbs_read_packet > cbs_read_data > cbs_h2645_split_fragment
```

FFmpeg初始化`output_stream`和`output_stream[0]`等全局变量的调用栈：

```
#0  new_output_stream (o=0x7fffffffd6a0, oc=0x686e80, type=AVMEDIA_TYPE_VIDEO, source_index=0) at fftools/ffmpeg_opt.c:1443
#1  0x0000000000411ccf in new_video_stream (o=0x7fffffffd6a0, oc=0x686e80, source_index=0) at fftools/ffmpeg_opt.c:1697
#2  0x0000000000417ba0 in open_output_file (o=0x7fffffffd6a0, filename=0x43b51d "pipe:") at fftools/ffmpeg_opt.c:2264
#3  0x000000000041c065 in open_files (l=0x64d940, inout=0x43d0c5 "output", open_file=0x417447 <open_output_file>) at fftools/ffmpeg_opt.c:3338
#4  0x000000000041c246 in ffmpeg_parse_options (argc=11, argv=0x7fffffffdd38) at fftools/ffmpeg_opt.c:3392
#5  0x000000000043a29a in main (argc=11, argv=0x7fffffffdd38) at fftools/ffmpeg.c:5032
```

初始化`AVBSFContext`结构体调用栈：

```
#0  av_bsf_list_finalize (lst=0x7fffffffc958, bsf=0x660210) at libavcodec/bsf.c:493
#1  0x00007ffff5233176 in av_bsf_list_parse_str (str=0x64eb20 "hevc_metadata=colour_primaries=1:transfer_characteristics=1:matrix_coefficients=1", bsf_lst=0x660210) at libavcodec/bsf.c:549
#2  0x0000000000410703 in new_output_stream (o=0x7fffffffd6a0, oc=0x686e80, type=AVMEDIA_TYPE_VIDEO, source_index=0) at fftools/ffmpeg_opt.c:1550
#3  0x0000000000411ccf in new_video_stream (o=0x7fffffffd6a0, oc=0x686e80, source_index=0) at fftools/ffmpeg_opt.c:1697
#4  0x0000000000417ba0 in open_output_file (o=0x7fffffffd6a0, filename=0x43b51d "pipe:") at fftools/ffmpeg_opt.c:2264
#5  0x000000000041c065 in open_files (l=0x64d940, inout=0x43d0c5 "output", open_file=0x417447 <open_output_file>) at fftools/ffmpeg_opt.c:3338
#6  0x000000000041c246 in ffmpeg_parse_options (argc=11, argv=0x7fffffffdd38) at fftools/ffmpeg_opt.c:3392
#7  0x000000000043a29a in main (argc=11, argv=0x7fffffffdd38) at fftools/ffmpeg.c:5032
```

初始化`CodedBitstreamContext`结构体调用栈：

```
#0  ff_cbs_init (ctx_ptr=0x660dd0, codec_id=AV_CODEC_ID_HEVC, log_ctx=0x64fa40) at libavcodec/cbs.c:117
#1  0x00007ffff5269773 in ff_cbs_bsf_generic_init (bsf=0x64fa40, type=0x7ffff63490c0 <h265_metadata_type>) at libavcodec/cbs_bsf.c:120
#2  0x00007ffff55b9095 in h265_metadata_init (bsf=0x64fa40) at libavcodec/h265_metadata_bsf.c:408
#3  0x00007ffff52325e7 in av_bsf_init (ctx=0x64fa40) at libavcodec/bsf.c:181
#4  0x0000000000433352 in init_output_bsfs (ost=0x6601c0) at fftools/ffmpeg.c:3088
#5  0x00000000004357d2 in init_output_stream (ost=0x6601c0, frame=0x0, error=0x7fffffffd1e0 "", error_len=1024) at fftools/ffmpeg.c:3745
#6  0x000000000042b0a5 in init_output_stream_wrapper (ost=0x6601c0, frame=0x0, fatal=0) at fftools/ffmpeg.c:992
#7  0x0000000000435c89 in transcode_init () at fftools/ffmpeg.c:3829
#8  0x0000000000439a20 in transcode () at fftools/ffmpeg.c:4820
#9  0x000000000043a3bb in main (argc=11, argv=0x7fffffffdd38) at fftools/ffmpeg.c:5054
```

初始化`(CodedBitstreamH2645Context*)->mp4`变量的调用栈：

```
#0  cbs_h2645_split_fragment (ctx=0x660100, frag=0x660de0, header=1) at libavcodec/cbs_h2645.c:600
#1  0x00007ffff524cdd4 in cbs_read_data (ctx=0x660100, frag=0x660de0, buf=0x0, data=0x653ac0 "\001\002 ", size=147, header=1) at libavcodec/cbs.c:263
#2  0x00007ffff524ce44 in ff_cbs_read_extradata (ctx=0x660100, frag=0x660de0, par=0x65ec00) at libavcodec/cbs.c:274
#3  0x00007ffff52697e5 in ff_cbs_bsf_generic_init (bsf=0x64fa40, type=0x7ffff63490c0 <h265_metadata_type>) at libavcodec/cbs_bsf.c:129
#4  0x00007ffff55b9095 in h265_metadata_init (bsf=0x64fa40) at libavcodec/h265_metadata_bsf.c:408
#5  0x00007ffff52325e7 in av_bsf_init (ctx=0x64fa40) at libavcodec/bsf.c:181
#6  0x0000000000433352 in init_output_bsfs (ost=0x6601c0) at fftools/ffmpeg.c:3088
#7  0x00000000004357d2 in init_output_stream (ost=0x6601c0, frame=0x0, error=0x7fffffffd1e0 "", error_len=1024) at fftools/ffmpeg.c:3745
#8  0x000000000042b0a5 in init_output_stream_wrapper (ost=0x6601c0, frame=0x0, fatal=0) at fftools/ffmpeg.c:992
#9  0x0000000000435c89 in transcode_init () at fftools/ffmpeg.c:3829
#10 0x0000000000439a20 in transcode () at fftools/ffmpeg.c:4820
#11 0x000000000043a3bb in main (argc=11, argv=0x7fffffffdd38) at fftools/ffmpeg.c:5054
```

## FFmpeg开发常见问题

### ffmpeg filter release版本处理首帧后抛出内存不足错误，debug版本正常

问题描述：

报错信息为
```
Error while filtering: Cannot allocate memory
Failed to inject frame into filter network: Cannot allocate memory
```

原因：

filter_frame函数的返回值未初始化，debug版本默认初始化为0，ffmpeg视为正常返回值，release版本默认不初始化，为随机值，ffmpeg视为异常返回值，经过错误处理程序后被当作AV_ERROR(ENOMEM)错误为处理。

解决办法：

初始化filter_frame返回值，根据需要返回正确的返回值。

### ffmpeg filter多输出一些图像帧

问题描述：

ffmpeg处理时，理论上输出帧数应当等于输入帧数，但实际上输出帧数为17，大于输入帧数10，且从以下ffmpeg日志中的`dup`字符可看出，多输出的帧为最后一帧的重复帧
```
*** 7 dup!
[out_0_0 @ 0x126a280] EOF on sink link out_0_0:default.
0.28 bitrate=8874664.3kbits/s dup=7 drop=0 speed=0.00146x    
No more output streams to write to, finishing.
frame= 17 fps=0.1 q=-0.0 Lsize=  307148kB time=00:00:00.28 bitrate=8880548.0kbits/s dup=7 drop=0 speed=0.00146x    
Input file #0 (i_YongYeXingHe05.4K.60fps.hdrvivid.f10.mp4):
  Input stream #0:0 (video): 10 packets read (877049 bytes); 10 frames decoded; 
  Total: 10 packets (877049 bytes) demuxed
Output file #0 (o_YongYeXingHe05.4K.60fps.hdrvivid.f10.ffm.yuv):
  Output stream #0:0 (video): 17 frames encoded; 17 packets muxed (314519040 bytes); 
  Total: 17 packets (314519040 bytes) muxed
```

原因：

输入mp4等文件中包含有特殊的pts时间戳信息

解决方法：

在ffmepg命令行中增加以下相关参数：
```
-vsync 0
```
