
```bash
# DWHISPER_METAL=NO 是明确指定启用 Metal 支持, 用 Metal 后端进行 GPU 加速
# -DWHISPER_COREML=1 是启用了 CoreML 后端.
# CoreML 和 Metal 是 macOS 上不同的 GPU 加速框架。
# 
# 希望在 Mac M1 上使用 GPU 加速，更应该使用 Metal 后端。 
# CoreML 通常用于苹果设备上的机器学习模型部署，但对于 whisper.cpp 来说，Metal 后端通常是更直接、更高效的选择，尤其是在 Mac M1 上。

cmake -B build -DWHISPER_METAL=ON
cmake --build build -j --config Release

# ffmpeg 转换音视频为 wav 音频文件
ffmpeg -i xxx.mp3/mp4 output.wav

# 识别中文, 指定生成 subtitles.srt
# 3个小时40分钟的，需要花费 15 分钟
./build/bin/whisper-cli -m models/ggml-base.bin -f output.wav -l zh -osrt -of subtitles -t 8

# 3个小时40分钟的，需要花费 19 分钟
build/bin/whisper-cli -m models/whisper/ggml-base.bin -f output.wav -bs 5 -l zh -oj

```


# 测试对比


原始视频
> 15分钟

```bash
ffmpeg -i test.mp4 
ffmpeg version 7.1.1 Copyright (c) 2000-2025 the FFmpeg developers
  built with Apple clang version 15.0.0 (clang-1500.1.0.2.5)
  configuration: --prefix=/opt/homebrew/Cellar/ffmpeg/7.1.1_1 --enable-shared --enable-pthreads --enable-version3 --cc=clang --host-cflags= --host-ldflags='-Wl,-ld_classic' --enable-ffplay --enable-gnutls --enable-gpl --enable-libaom --enable-libaribb24 --enable-libbluray --enable-libdav1d --enable-libharfbuzz --enable-libjxl --enable-libmp3lame --enable-libopus --enable-librav1e --enable-librist --enable-librubberband --enable-libsnappy --enable-libsrt --enable-libssh --enable-libsvtav1 --enable-libtesseract --enable-libtheora --enable-libvidstab --enable-libvmaf --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxml2 --enable-libxvid --enable-lzma --enable-libfontconfig --enable-libfreetype --enable-frei0r --enable-libass --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-libspeex --enable-libsoxr --enable-libzmq --enable-libzimg --disable-libjack --disable-indev=jack --enable-videotoolbox --enable-audiotoolbox --enable-neon
  libavutil      59. 39.100 / 59. 39.100
  libavcodec     61. 19.101 / 61. 19.101
  libavformat    61.  7.100 / 61.  7.100
  libavdevice    61.  3.100 / 61.  3.100
  libavfilter    10.  4.100 / 10.  4.100
  libswscale      8.  3.100 /  8.  3.100
  libswresample   5.  3.100 /  5.  3.100
  libpostproc    58.  3.100 / 58.  3.100
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'test.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    creation_time   : 1970-01-01T00:00:00.000000Z
    encoder         : Lavf52.31.0
  Duration: 00:14:53.54, start: 0.000000, bitrate: 1099 kb/s
  Stream #0:0[0x1](und): Video: h264 (Main) (avc1 / 0x31637661), yuv420p(progressive), 1280x720 [SAR 1:1 DAR 16:9], 1048 kb/s, 24 fps, 24 tbr, 24 tbn (default)
      Metadata:
        creation_time   : 1970-01-01T00:00:00.000000Z
        handler_name    : VideoHandler
        vendor_id       : [0][0][0][0]
  Stream #0:1[0x2](eng): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 45 kb/s (default)
      Metadata:
        creation_time   : 1970-01-01T00:00:00.000000Z
        handler_name    : SoundHandler
        vendor_id       : [0][0][0][0]
At least one output file must be specified
```

转为 wav 音频
```bash
ffmpeg -i test.mp4 -ac 1 -ar 16000 -acodec pcm_s16le test.wav
```

使用 base 模型
```bash
build/bin/whisper-cli -m models/whisper/ggml-base.bin -f test.wav -l zh -osrt -of test-base -t 8
```

base 模型耗时 40秒

> 基本上很多识别错误

```
output_srt: saving output to 'test-base.srt'

whisper_print_timings:     load time =   190.72 ms
whisper_print_timings:     fallbacks =   1 p /   9 h
whisper_print_timings:      mel time =   339.01 ms
whisper_print_timings:   sample time = 11212.16 ms / 14171 runs (    0.79 ms per run)
whisper_print_timings:   encode time =  5286.01 ms /    31 runs (  170.52 ms per run)
whisper_print_timings:   decode time =  1518.36 ms /   208 runs (    7.30 ms per run)
whisper_print_timings:   batchd time = 20336.28 ms / 13811 runs (    1.47 ms per run)
whisper_print_timings:   prompt time =  1175.15 ms /  7012 runs (    0.17 ms per run)
whisper_print_timings:    total time = 40899.04 ms
ggml_metal_free: deallocating
```

使用 large-v3-turbo 模型
```bash
build/bin/whisper-cli -m models/whisper/ggml-large-v3-turbo.bin -f test.wav -l zh -osrt -of test-large-v3-turbo -t 8
```

large-v3-turbo 模型耗时 175秒

> 识别精度还可以，但是耗时太长了

```
output_srt: saving output to 'test-large-v3-turbo.srt'

whisper_print_timings:     load time =  1201.38 ms
whisper_print_timings:     fallbacks =  13 p /  14 h
whisper_print_timings:      mel time =   470.62 ms
whisper_print_timings:   sample time = 20657.55 ms / 23869 runs (    0.87 ms per run)
whisper_print_timings:   encode time = 97276.96 ms /    38 runs ( 2559.92 ms per run)
whisper_print_timings:   decode time =  2114.78 ms /   226 runs (    9.36 ms per run)
whisper_print_timings:   batchd time = 47922.75 ms / 23428 runs (    2.05 ms per run)
whisper_print_timings:   prompt time =  3708.90 ms /  8681 runs (    0.43 ms per run)
whisper_print_timings:    total time = 175444.44 ms
ggml_metal_free: deallocating
```

使用 large-v3-turbo 模型，并限制线程数为 4
```bash
build/bin/whisper-cli -m models/whisper/ggml-large-v3-turbo.bin -f test.wav -l zh -osrt -of test-large-v3-turbo-t4 -t 4
```

large-v3-turbo 模型耗时 219秒

> 识别精度还可以， 和 t8 完全一样，但是耗时太长了

```
output_srt: saving output to 'test-large-v3-turbo-t4.srt'

whisper_print_timings:     load time =  1899.38 ms
whisper_print_timings:     fallbacks =  13 p /  14 h
whisper_print_timings:      mel time =   539.23 ms
whisper_print_timings:   sample time = 23193.41 ms / 23869 runs (    0.97 ms per run)
whisper_print_timings:   encode time = 128409.49 ms /    38 runs ( 3379.20 ms per run)
whisper_print_timings:   decode time =  2318.95 ms /   226 runs (   10.26 ms per run)
whisper_print_timings:   batchd time = 50477.16 ms / 23428 runs (    2.15 ms per run)
whisper_print_timings:   prompt time =  8480.07 ms /  8681 runs (    0.98 ms per run)
whisper_print_timings:    total time = 218927.97 ms
ggml_metal_free: deallocating
```

> 结论是还不如用剪映的本地语音识别，这个音频剪映识别的准确率更高，耗时更短.
> 同样的 15分钟 wav, 在 mac M1 上剪映才花费 30s, 而 whisper 却需要 190-220s

