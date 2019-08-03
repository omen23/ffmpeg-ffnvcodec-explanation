# FFmpeg-ffnvcodec-explanation
 **\*\*last updated 03-08-2019\*\***
 – updated own distro to **19.04 disco** and updated the manual 

How to get FFmpeg to export the needed symbols on (K)ubuntu cosmic/disco (and similar distros) so OBS and MPV can use NVENC and NVDEC (formerly called CUVID) on Fermi, Maxwell, Kepler, Pascal, Volta and Turing architectures and how to use hardware-acceleration in Chromium  © *2018 - 2019 oMeN23*.
**This guide aims at the more modern cards compatible with NVENC and NVDEC – but there's also some instructions for cards using legacy drivers. (378, 390 and 396)**

Supported cards: https://developer.nvidia.com/video-encode-decode-gpu-support-matrix

### 0. Get Nvidia's proprietary driver:
```
sudo apt install ppa-purge # for safety
sudo add-apt-repository ppa:graphics-drivers/ppa
# please check the support plan for your GPU – nvidia-driver-{390,396,410,415,418,430} available so your display server doesn't fail!
# we will use 430.40 
# maybe you need the 32-bit versions of some libraries add ":i386" to the package name (i.e.: libnvidia-gl-430:i386)
sudo apt install nvidia-driver-430 xserver-xorg-video-nvidia-430 nvidia-utils-430 nvidia-kernel-source-430 nvidia-kernel-common-430 nvidia-dkms-430 nvidia-compute-utils-430 libnvidia-ifr1-430 libnvidia-gl-430 libnvidia-fbc1-430 libnvidia-encode-430 libnvidia-decode-430 libnvidia-cfg1-430 libnvidia-compute-430
```

### 1. Install the nv-codec-headers package:
```
sudo apt install make git
mkdir ~/devel/ && cd ~/devel/
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
cd nv-codec-headers
# change to older SDK branch here if you use a legacy driver — one of: git checkout sdk/8.{0,1,2} 
make && sudo make install
```
Most recent version of the Video Codec SDK is 9.0.x. *but* **below** is a list of the older SDK branches for legacy drivers.
```
FFmpeg version of headers required to interface with Nvidias codec APIs.

Corresponds to Video Codec SDK version 9.0.18.

Minimum required driver versions:
Linux: 418.30 or newer
Windows: 418.81 or newer
```

- **List of git branches for the legacy drivers**

If you use legacy drivers, here are the branches for them – after you clone the git repository `cd` into it and enter one of the three following lines before installing *(entering `make && sudo make install`)*.
Curious what will happen if you use the newest SDK headers with a legacy driver?  The linker/loader will not find the entrypoint or function (pointer) or header and your application will fail.

*All of these three branches support optional CUDA 10 features starting with driver 410.48 on Linux and 411.31 on Windows.*
```
git checkout sdk/8.2 # for Linux 396.24 or newer | for Windows 397.93 or newer
git checkout sdk/8.1 # for Linux 390.25 or newer | for Windows 390.77 or newer
git checkout sdk/8.0 # for Linux 378.13 or newer | for Windows 378.66 or newer
```


### 2. Compile FFmpeg:

FFmpeg will automatically detect the ffnvcodec-headers — extract from `./configure --help`:
```
  The following libraries provide various hardware acceleration features:
  --disable-amf            disable AMF video encoding code [autodetect]
  --disable-audiotoolbox   disable Apple AudioToolbox code [autodetect]
  --enable-cuda-sdk        enable CUDA features that require the CUDA SDK [no]
  --disable-cuvid          disable Nvidia CUVID support [autodetect]
  --disable-d3d11va        disable Microsoft Direct3D 11 video acceleration code [autodetect]
  --disable-dxva2          disable Microsoft DirectX 9 video acceleration code [autodetect]
  --disable-ffnvcodec      disable dynamically linked Nvidia code [autodetect]
  --enable-libdrm          enable DRM code (Linux) [no]
  --enable-libmfx          enable Intel MediaSDK (AKA Quick Sync Video) code via libmfx [no]
  --enable-libnpp          enable Nvidia Performance Primitives-based code [no]
  --enable-mmal            enable Broadcom Multi-Media Abstraction Layer (Raspberry Pi) via MMAL [no]
  --disable-nvdec          disable Nvidia video decoding acceleration (via hwaccel) [autodetect]
  --disable-nvenc          disable Nvidia video encoding code [autodetect]
  --enable-omx             enable OpenMAX IL code [no]
  --enable-omx-rpi         enable OpenMAX IL code for Raspberry Pi [no]
  --enable-rkmpp           enable Rockchip Media Process Platform code [no]
  --disable-v4l2-m2m       disable V4L2 mem2mem code [autodetect]
  --disable-vaapi          disable Video Acceleration API (mainly Unix/Intel) code [autodetect]
  --disable-vdpau          disable Nvidia Video Decode and Presentation API for Unix code [autodetect]
  --disable-videotoolbox   disable VideoToolbox code [autodetect]
```


Should the standard compilation not fit your needs (you have the need to link in specific libraries/don't need some libraries or you want to enable/disable specific features) then you can change the build-rules in `~/devel/ffmpeg/ffmpeg-4.1.3/debian/rules`.
The headers installed in **1. Install the nv-codec-headers package** will enable `ffnvcodec vdpau nvenc nvdec cuda cuvid` (***NVDEC is just a rebranding of CUVID***). CUDA does not mean the CUDA-SDK code – just hardware-acceleration via CUDA.
```
sudo apt build-dep ffmpeg
mkdir -p ~/devel/ffmpeg
cd ~/devel/ffmpeg
sudo apt source ffmpeg
sudo chown -hR $USER: * # atleast my distro has problems when unpacking source – so we change ownership
cd ffmpeg-4.1.3 # cd ffmpeg-x.x.x [x.x.x represents the version number] 
debuild -b --no-sign --jobs-try=$((`nproc`/2))
cd ..
rm -f libavcodec-extra* libavfilter-extra*
sudo dpkg -i *.deb
sudo apt-mark hold ffmpeg ffmpeg-doc libavcodec-dev libavcodec58 libavfilter7 libavfilter-dev libavformat58 libavformat-dev libavresample4 libavresample-dev libavutil-dev libavutil56 libavdevice58 libavdevice-dev libswscale5 libswscale-dev libswresample3 libswresample-dev libpostproc55 libpostproc-dev
```

- **Testing:**
```
ffmpeg -hide_banner -encoders  | grep nvenc
 V..... h264_nvenc           NVIDIA NVENC H.264 encoder (codec h264)
 V..... nvenc                NVIDIA NVENC H.264 encoder (codec h264)
 V..... nvenc_h264           NVIDIA NVENC H.264 encoder (codec h264)
 V..... nvenc_hevc           NVIDIA NVENC hevc encoder (codec hevc)
 V..... hevc_nvenc           NVIDIA NVENC hevc encoder (codec hevc)
ffmpeg -hide_banner -decoders | grep cuvid
 V..... h264_cuvid           Nvidia CUVID H264 decoder (codec h264)
 V..... hevc_cuvid           Nvidia CUVID HEVC decoder (codec hevc)
 V..... mjpeg_cuvid          Nvidia CUVID MJPEG decoder (codec mjpeg)
 V..... mpeg1_cuvid          Nvidia CUVID MPEG1VIDEO decoder (codec mpeg1video)
 V..... mpeg2_cuvid          Nvidia CUVID MPEG2VIDEO decoder (codec mpeg2video)
 V..... mpeg4_cuvid          Nvidia CUVID MPEG4 decoder (codec mpeg4)
 V..... vc1_cuvid            Nvidia CUVID VC1 decoder (codec vc1)
 V..... vp8_cuvid            Nvidia CUVID VP8 decoder (codec vp8)
 V..... vp9_cuvid            Nvidia CUVID VP9 decoder (codec vp9) 
ffmpeg -hide_banner -hwaccels
Hardware acceleration methods:
vdpau
cuda
vaapi
drm
cuvid
```

### 3. Install OBS (no need to compile): 
OBS detects if FFmpeg (libavcodec) exports `h264_nvenc` dynamically at runtime startup if GPU-acceleration was not disabled at build-time. 
```
sudo add-apt-repository ppa:obsproject/obs-studio
sudo apt update
sudo apt install obs-studio
```
**NOW** you should have a fully functional OBS with hardware-acceleration!
The rest of this guide is ***optional*** (for people who want to get the most out of their GPU).

### 4. Build MPV to use NVDEC for video decoding:
```
sudo apt build-dep mpv
mkdir -p ~/devel/mpv
cd ~/devel/mpv
sudo apt source mpv
sudo chown -hR $USER: *
cd mpv-0.29.1 # cd mpv-x.x.x [x.x.x represents version]
debuild -b --no-sign --jobs-try=$((`nproc`/2))  
cd ..
sudo dpkg -i mpv*.deb # we dont need libmpv{-dev}
sudo apt-mark hold mpv
...
mpv --hwdec=nvdec <input> # --hwdec=yes or auto will work too – just tweak your configuration file
```
**Protip: use mpv to play youtube videos with NVDEC VP9 decoding.**

Add `youtube-dl` to your system:
```
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt update
sudo apt install youtube-dl
```
Now you can watch youtube videos and livestreams *(e.g. twitch.tv)* with hardware-acceleration too — use `mpv URL ...`
### 5. Use hardware-acceleration enabled chromium:

Thanks to [Saikrishna Arcot](https://github.com/saiarcot895) who patched chromium against VAAPI there is hardware-acceleration for Intel and Nvidia GPUs (you'll need the patched `vdpau-va-driver`).

https://www.linuxuprising.com/2018/08/how-to-enable-hardware-accelerated.html (The article is a little outdated – my manual has all the latest changes covered too)
```
sudo add-apt-repository ppa:saiarcot895/chromium-dev
sudo apt update
sudo apt install vdpau-va-driver chromium-browser vdpauinfo vainfo
```
Then in chromium type in the adressbar `chrome://flags/#enable-accelerated-video` and `chrome://flags/#ignore-gpu-blacklist` and enable it and maybe zero-copy too etc. 
Check out `chrome://gpu` or `about:gpu` to see what configuration works best for you (e.g. on my system out of process rasterization doesn't work well and GpuMemoryBuffers are only implemented in software as of now 09-03-2019).
Install the [h264ify extension](https://chrome.google.com/webstore/detail/h264ify/aleakchihdccplidncghkekgioiakgal)
(even you now have a system that **can** offload VP9 decoding to the video card – it is not implemented in any browser.)

- **Checking if chromium actually uses the video card:**

Go to `chrome://media-internals` or `about:media-internals` when **h264ify** is **enabled** *(the video is delivered in `avc1` not `vp9` – you find that info in the `Stats for nerds` box when right clicking on a video)*, play a YouTube video and click on the box that says `(PLAY)` at the bottom right and in the **Log** you can search for properties – search for the `video_decoder` field and `MojoVideoDecoder` should be its value and to make sure `is_platform_video_decoder` which should be `true`. (in `nvidia-settings` **Video Engine Utilization** will rise if you play a video if everything is working) If the value of the field `video_decoder` is `FFmpegVideoDecoder` or `VpxVideoDecoder` you have an error somewhere. *(check `vainfo` and `vdpauinfo` outputs for errors first...)*

`GpuVideoDecoder` is deprecated now! `MojoVideoDecoder` has much better performance in utilizing the graphics cards **Video Engine** than `GpuVideoDecoder` on more recent rigs! Welp, it's a complete rewrite of the video decoder…*


Tested on a GeForce GTX 1070 system. If you have \*\***ANY**\*\* questions regarding this manual or something doesn't work out of the box for you or you want a more detailed explanation of one of the topics/steps — just open a new issue.
#### Donations
If this manual helped you out in any way and you wanna support me by buying a cup of coffee or a little something [***donate here***](https://www.streamlabs.com/omen235). 

***Thank you in advance!***
