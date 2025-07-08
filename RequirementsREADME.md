# Havoc 4.0 - "The Rise of ShakaZulu" - System Requirements
![Shakazulu1](https://github.com/user-attachments/assets/ba57a1ce-dc2d-4818-9310-2af69a24e14b)

This document outlines the system requirements and dependencies needed to build and run Havoc 4.0 with the enhanced beacon notification system.

## Overview

Havoc 4.0 introduces several new features including:
- 🔊 **Audio Beacon Notifications** - Play custom sounds on beacon callbacks
- 👀 **Visual Beacon Notifications** - Display GIF/image popups for beacon activity  
- ⚙️ **Notification Settings UI** - Configure notifications via Preferences menu
- 🎨 **ShakaZulu Theme** - Custom ASCII banner and branding
- 🛡️ **Enhanced HTTP Filtering** - Custom header filtering capabilities

## System Requirements

### Operating System
- **Linux**: Ubuntu 20.04+ / Debian 11+ / Kali Linux 2023.1+
- **macOS**: macOS 11.0+ (Big Sur)
- **Windows**: Windows 10+ (with WSL2 recommended for development)

### Minimum Hardware
- **CPU**: x86_64 architecture (Intel/AMD 64-bit)
- **RAM**: 4GB minimum, 8GB recommended
- **Storage**: 2GB free space for build dependencies and compilation
- **Audio**: Sound card/audio device (for audio notifications)

## Build Dependencies

### Core Build Tools
```bash
# Ubuntu/Debian/Kali
sudo apt update && sudo apt install -y \
    build-essential \
    cmake \
    make \
    git \
    pkg-config
```

### Qt5 Framework (Required)
```bash
# Ubuntu/Debian/Kali
sudo apt install -y \
    qt5-default \
    qtbase5-dev \
    qttools5-dev \
    qttools5-dev-tools \
    libqt5gui5 \
    libqt5core5a \
    libqt5widgets5 \
    libqt5network5 \
    libqt5websockets5-dev \
    libqt5sql5 \
    libqt5sql5-sqlite
```

### Qt5 Multimedia (NEW - Required for Beacon Notifications)
```bash
# Ubuntu/Debian/Kali
sudo apt install -y \
    qtmultimedia5-dev \
    libqt5multimedia5 \
    libqt5multimedia5-plugins \
    libqt5multimediawidgets5 \
    libqt5multimediagsttools5 \
    libqt5multimediaquick5
```

### Audio System Dependencies
```bash
# Ubuntu/Debian/Kali
sudo apt install -y \
    pulseaudio \
    alsa-utils \
    gstreamer1.0-plugins-base \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-libav
```

### Python Development (Required for Scripts)
```bash
# Ubuntu/Debian/Kali
sudo apt install -y \
    python3-dev \
    python3-pip \
    libpython3-dev
```

### Additional Libraries
```bash
# Ubuntu/Debian/Kali
sudo apt install -y \
    libssl-dev \
    libffi-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    libncurses5-dev \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libxml2-dev \
    libxmlsec1-dev \
    libffi-dev \
    liblzma-dev
```

## macOS Dependencies

### Homebrew Installation
```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install \
    cmake \
    make \
    qt@5 \
    python@3.10 \
    pkg-config

# Set Qt5 path
export PATH="/opt/homebrew/opt/qt@5/bin:$PATH"
export CMAKE_PREFIX_PATH="/opt/homebrew/opt/qt@5"
```

## Runtime Requirements

### Audio Support
- **PulseAudio** or **ALSA** for Linux audio output
- **CoreAudio** for macOS
- **DirectSound/WASAPI** for Windows

### Supported Audio Formats
- **MP3** - MPEG-1 Audio Layer 3
- **WAV** - Waveform Audio File Format
- **OGG** - Ogg Vorbis
- **M4A** - MPEG-4 Audio

### Supported Visual Formats
- **GIF** - Graphics Interchange Format (animated)
- **PNG** - Portable Network Graphics
- **JPG/JPEG** - Joint Photographic Experts Group
- **BMP** - Bitmap Image File
- **WEBP** - WebP Image Format

## Build Instructions

### 1. Clone Repository
```bash
git clone https://github.com/HavocFramework/Havoc.git
cd Havoc
```

### 2. Build Client
```bash
cd client
make clean
make
```

### 3. Build Teamserver
```bash
cd ../teamserver
make clean
make
```

## Configuration

### Audio Setup
1. Ensure audio system is working: `speaker-test -t wav -c 2`
2. Test audio playback: `aplay /usr/share/sounds/alsa/Front_Left.wav`
3. Configure PulseAudio if needed: `pulseaudio --start`

### Notification Media Files
- Place notification audio files in accessible directory
- Recommended format: WAV (44.1kHz, 16-bit) for best compatibility
- Keep file sizes reasonable (< 5MB for responsiveness)
- Test files with system media player before configuring

## Troubleshooting

### Common Build Issues

#### Qt5Multimedia Not Found
```bash
# Error: Could not find a package configuration file provided by "Qt5Multimedia"
sudo apt install qtmultimedia5-dev libqt5multimedia5-plugins
```

#### Python Development Headers Missing
```bash
# Error: Python.h not found
sudo apt install python3-dev libpython3-dev
```

#### Audio Playback Issues
```bash
# Test audio system
pulseaudio --check
aplay -l  # List audio devices
speaker-test -t wav -c 2  # Test speakers
```

#### GStreamer Plugins Missing
```bash
# For multimedia codec support
sudo apt install \
    gstreamer1.0-plugins-base \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-libav
```

### Runtime Issues

#### No Audio Output
1. Check system volume and mute status
2. Verify audio device selection in system settings
3. Test with: `pactl info` and `pactl list sinks`
4. Restart audio service: `sudo systemctl restart pulseaudio`

#### Visual Notifications Not Displaying
1. Verify file format is supported
2. Check file permissions and accessibility
3. Ensure file size is reasonable (< 10MB)
4. Test file with system image viewer

#### Settings Not Persisting
1. Check home directory write permissions
2. Verify QSettings can create config files
3. Check: `~/.config/Havoc/` directory permissions

## Feature Usage

### Beacon Notifications Setup
1. Launch Havoc 4.0
2. Go to **Havoc** → **Preferences** (or press Ctrl+Alt+S)
3. Enable audio/visual notifications as desired
4. Select notification media files using Browse buttons
5. Configure notification duration (1-30 seconds)
6. Click **Save** to apply settings

### Supported Notification Events
- **New Beacon Connection** - Triggered when a new agent connects
- **Beacon Callback** - Triggered when an existing beacon sends output

## Security Considerations

- Notification media files should be from trusted sources
- Audio files are played through system audio - ensure appropriate volume
- Visual notifications appear on screen - consider operational security
- Settings are stored locally in user configuration directory

## Version Information

- **Havoc Version**: 4.0
- **Codename**: "The Rise of ShakaZulu"
- **Qt Version**: 5.15+ required
- **Python Version**: 3.8+ required  
- **CMake Version**: 3.15+ required

## Credits

Original Havoc Framework by **C5pider** (@C5pider)  
Modifications and enhancements by **BushidoKarat3** (@bushidokarat3)

## Support

For issues specific to the notification system or Havoc 4.0 modifications:
- GitHub: https://github.com/bushidokarat3
- Issues: Report via GitHub Issues

For general Havoc Framework support:
- Documentation: https://havocframework.com/docs
- GitHub: https://github.com/HavocFramework/Havoc
