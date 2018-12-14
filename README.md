# ffmpeg-ffnvcodec-explanation
how to get ffmpeg to export the needed symbols on (K)ubuntu cosmic 18.10 so OBS and MPV can use NVENC/NVDEC


```
In file included from nasmlib/crc64.c:35:
./include/nasmlib.h:194:1: error: ‘pure’ attribute on function returning ‘void’ [-Werror=attributes]
 void pure_func seg_init(void);
I used yasm and it worked too
```
get nvenc headers and build your ffmpeg distro package from source

**1. Install the nv-codec-headers package:**

```
sudo apt-get install make git
mkdir -p ~/devel/ && cd ~/devel/
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
cd nv-codec-headers
make && sudo make install
```

**2. Compile FFmpeg:**
```
sudo apt-get build-dep ffmpeg
mkdir -p ~/devel/ffmpeg
cd ~/devel/ffmpeg
sudo apt-get source ffmpeg
cd ffmpeg-4.0.2 #cd ffmpeg-x.x.x [x.x.x represents the version number]
debuild -us -uc -b
cd ..
rm libavcodec58*
sudo dpkg -i *.deb
sudo apt-mark hold ffmpeg libavcodec-dev libavcodec-extra58 libavfilter-dev libavformat58 libavresample-dev libavutil-dev libavutil56
```
- **Testing:**
```
$ ffmpeg -hide_banner -encoders  | grep nvenc
 V..... h264_nvenc           NVIDIA NVENC H.264 encoder (codec h264)
 V..... nvenc                NVIDIA NVENC H.264 encoder (codec h264)
 V..... nvenc_h264           NVIDIA NVENC H.264 encoder (codec h264)
 V..... nvenc_hevc           NVIDIA NVENC hevc encoder (codec hevc)
 V..... hevc_nvenc           NVIDIA NVENC hevc encoder (codec hevc)
$ ffmpeg -hide_banner -decoders | grep cuvid
 V..... h264_cuvid           Nvidia CUVID H264 decoder (codec h264)
 V..... hevc_cuvid           Nvidia CUVID HEVC decoder (codec hevc)
 V..... mjpeg_cuvid          Nvidia CUVID MJPEG decoder (codec mjpeg)
 V..... mpeg1_cuvid          Nvidia CUVID MPEG1VIDEO decoder (codec mpeg1video)
 V..... mpeg2_cuvid          Nvidia CUVID MPEG2VIDEO decoder (codec mpeg2video)
 V..... mpeg4_cuvid          Nvidia CUVID MPEG4 decoder (codec mpeg4)
 V..... vc1_cuvid            Nvidia CUVID VC1 decoder (codec vc1)
 V..... vp8_cuvid            Nvidia CUVID VP8 decoder (codec vp8)
 V..... vp9_cuvid            Nvidia CUVID VP9 decoder (codec vp9) 
```

**3. Install OBS (no need to compile)** 
```
sudo apt-get install ppa-purge #for safety
sudo add-apt-repository ppa:obsproject/obs-studio
sudo apt-get update
sudo apt-get install obs-studio
```


**4. Build MPV to use NVDEC for movie viewing**
```
mkdir -p ~/devel/mpv
cd ~/devel/mpv
sudo apt-get source mpv
cd mpv-0.29.0 #cd mpv-x.x.x [x.x.x represents version]
debuild -us -uc -b
cd ..
sudo dpkg -i mpv*.deb # dont need libmpv{-dev}
sudo apt-mark hold mpv
```
enJOY
