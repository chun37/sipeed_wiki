---
title: 音频操作
keywords: 音频操作, MaixPy3, Python, Python3
desc: maixpy doc: 音频操作
---

MaixPy3 关于音频相关操作采用的是 库，PyAudio 库，PyAudio 为跨平台音频 I/O 库 PortAudio 提供了 Python 绑定，帮助用户轻松地在各种平台上播放和录制音频。

[pyaudio 官方文档](http://people.csail.mit.edu/hubert/pyaudio/docs/)

## 录音操作

```python
import pyaudio
import wave

CHUNK = 1024
FORMAT = pyaudio.paInt16
CHANNELS = 2
RATE = 44100
RECORD_SECONDS = 5
WAVE_OUTPUT_FILENAME = "output.wav"

p = pyaudio.PyAudio()

stream = p.open(format=FORMAT,
                channels=CHANNELS,
                rate=RATE,
                input=True,
                frames_per_buffer=CHUNK)

print("* recording")

frames = []

for i in range(0, int(RATE / CHUNK * RECORD_SECONDS)):
    data = stream.read(CHUNK)
    frames.append(data)

print("* done recording")

stream.stop_stream()
stream.close()
p.terminate()

wf = wave.open(WAVE_OUTPUT_FILENAME, 'wb')
wf.setnchannels(CHANNELS)
wf.setsampwidth(p.get_sample_size(FORMAT))
wf.setframerate(RATE)
wf.writeframes(b''.join(frames))
wf.close()
```

## 播放音频

```python

import pyaudio
import wave
import sys

# 定义数据流块
CHUNK = 1024

if len(sys.argv) < 2:
    print("Plays a wave file.\n\nUsage: %s filename.wav" % sys.argv[0])
    sys.exit(-1)

# 只读方式打开wav文件
wf = wave.open(r'test.wav', 'rb')#(sys.argv[1], 'rb')

p = pyaudio.PyAudio()

# 打开数据流
stream = p.open(format=p.get_format_from_width(wf.getsampwidth()),
                channels=wf.getnchannels(),
                rate=wf.getframerate(),
                output=True)

# 读取数据
data = wf.readframes(CHUNK)

# 播放
while data != '':
    stream.write(data)
    data = wf.readframes(CHUNK)

# 停止数据流
stream.stop_stream()
stream.close()

# 关闭 PyAudio
p.terminate()
```
