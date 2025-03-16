
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

```
