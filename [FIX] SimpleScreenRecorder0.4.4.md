
# SimpleScreenRecorder 0.4.4 – Fix for MP3 Encoding on Debian 13 (FFmpeg 7.x)

###  Bug: MP3 encoder fails with  
**`libmp3lame` expects “stereo” instead of “2 channels”**  
Bug reference: https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1093281

---

##  Issue Summary
![alt text](https://github.com/flpmarti25/dev-code/blob/main/simplescreenrecorder/img2.png?raw=true)

![alt text](https://github.com/flpmarti25/dev-code/blob/main/simplescreenrecorder/img1.png?raw=true)



After upgrading to **Debian 13 (Trixie)**, the system FFmpeg version is upgraded from **5.1** to **7.x**.  
FFmpeg 7 introduced multiple breaking changes, including the **removal of legacy channel layout macros**:

- `AV_CH_LAYOUT_MONO`
- `AV_CH_LAYOUT_STEREO`
- and other old bitmask-based layouts

These macros were previously deprecated and have now been **removed from the public API**.

As a result, any code in SimpleScreenRecorder that still uses:

```cpp
ctx->channel_layout = AV_CH_LAYOUT_STEREO
```
or similar legacy APIs **fails to compile** or generates incorrect `AVChannelLayout` structures.

###  Runtime Failure

When attempting to use MP3 encoding (`libmp3lame`), FFmpeg logs:

```cpp
Specified channel layout '2 channels' is not supported by the libmp3lame encoder
Supported channel layouts:
  mono
  stereo
  ```
  This happens because FFmpeg 7 interprets the old 2-channel mask as a **generic “2 channels” layout**, not as **stereo**, and `libmp3lame` only supports:

-   `mono`
-   `stereo`

###  Temporary Solution (Manual Build)
#### 1. Install build dependencies

sudo apt update
sudo apt build-dep simplescreenrecorder
#### 2. Download and build sources

cd simplescreenrecorder-0.4.4
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug
make -j$(nproc)

If successful, you’ll see:

[100%] Built target simplescreenrecorder
![alt text](https://github.com/flpmarti25/dev-code/blob/main/simplescreenrecorder/img4.png?raw=true)

 ##### Qt Error (Ignore or Patch)
```cpp
Found unsuitable Qt version "5.15.15" ... requires Qt 4.x
```
This comes from the upstream CMake configuration assuming **Qt4**, which no longer exists on modern systems.


![alt text](https://github.com/flpmarti25/dev-code/blob/main/simplescreenrecorder/img3.png?raw=true)

 ##### File to Patch (FFmpeg 7 Fix)
Edit:

src/AV/Output/BaseEncoder.cpp

Replace any legacy channel layout usage with:
```cpp
av_channel_layout_default(&m_codec_context->ch_layout,  2);
```
![alt text](https://github.com/flpmarti25/dev-code/blob/main/simplescreenrecorder/img5.png?raw=true)

After applying this patch and rebuilding SimpleScreenRecorder, MP3 encoding works again on **Debian 13 / FFmpeg 7.x** with no runtime errors.




