# scrcpy

## Instalasi

```
sudo apt-get update
sudo apt-get install ffmpeg libsdl2-2.0-0 adb wget gcc git pkg-config meson ninja-build libsdl2-dev libavcodec-dev libavdevice-dev libavformat-dev libavutil-dev libswresample-dev libusb-1.0-0 libusb-1.0-0-dev
git clone https://github.com/Genymobile/scrcpy
cd scrcpy
./install_release.sh
```

## Penggunaan

### Mode USB

```
scrcpy
```

### Mode Wireless

```
adb tcpip [port]
adb connect [ip_address]:[port]
scrcpy
```
