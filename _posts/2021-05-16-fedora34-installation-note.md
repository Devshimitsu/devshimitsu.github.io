---
published: true
layout: post
title: Fedora 34 Installation
subtitle: With Acer Aspire V Nitro
cover-img:
thumbnail-img:
share-img:
tags: [linux, fedora, gnome]
---

# Fedora 34 Installation
Device: Acer Aspire V Nitro  

## 1. Touchpad (Device specific)
The touchpad's physical button doesn't work well.  
Most of the time, it requires 3 clicks to register a single click.  
The secondary button simply doesn't register.  
Enable `Tap to Click` at setting to workaround this issue.  
```
gsettings set org.gnome.desktop.peripherals.touchpad tap-to-click 'true'
# Do this with sudo as well, to enable tap-to-click for GDM
sudo gsettings set org.gnome.desktop.peripherals.touchpad tap-to-click 'true'
```
(Seriously, why it isn't enabled by default?)  

## 2. Sound (Device specific)
Sound output doesn't work by default.  
This is the common issue of this paticular device.  
```
echo "options snd-hda-intel dmic_detect=0" | sudo tee -a /etc/modprobe.d/alsa-base.conf
echo "blacklist snd_soc_skl" | sudo tee -a /etc/modprobe.d/blacklist.conf
```
Reboot.  

## 3. RPM Fusion
Enable RPM Fusion to get access to multimedia codecs, NVDIA drivers, etc.  
```
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

## 4. Flathub
Add flathub repo to flatpak.  
```
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

## 5. Multimedia
```
sudo dnf groupupdate Multimedia
# Video acceleration Intel (Optional)
sudo dnf install libva-intel-driver libva-intel-hybrid-driver
# Video acceleration AMD/NVIDIA (Optional)
sudo dnf install libvdpau-va-gl
```

## 6. NVIDIA driver (Device specific)
Make sure you disabled the Secure Boot  
```
sudo dnf install xorg-x11-drv-nvidia akmod-nvidia
# Optional, for cuda/nvdec/nvenc support
sudo dnf install xorg-x11-drv-nvidia-cuda
```
Reboot.  

## 7. Daily (Optional)
Get only what you need! ;)  
```
flatpak install flathub com.bitwarden.desktop com.discordapp.Discord com.github.tchx84.Flatseal com.microsoft.Teams com.obsproject.Studio com.skype.Client com.slack.Slack com.spotify.Client com.wps.Office fr.handbrake.ghb org.kde.kdenlive us.zoom.Zoom
sudo dnf install conky geary gimp gnome-tweaks lollypop lutris pulseeffects steam tilix tilix-nautilus uget wine winetricks zsh
```

### Anydesk (Flathub)  
```
flatpak install flathub com.anydesk.Anydesk
```

### Anydesk (Official Repo)  
Note that the package is broken at the time of writing.  
Run after `sudo -i`  
```
cat > /etc/yum.repos.d/AnyDesk-Fedora.repo << "EOF" 
[anydesk]
name=AnyDesk Fedora - stable
baseurl=http://rpm.anydesk.com/fedora/$basearch/
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://keys.anydesk.com/repos/RPM-GPG-KEY
EOF
dnf install anydesk
```

### ProtonVPN (Official Repo)  
Run after `sudo -i`  
```
cat > /etc/yum.repos.d/fedora-33-unstable.repo << "EOF" 
[proton-fedora-33-unstable]
name="fedora-33-unstable"
baseurl=https://repo.protonvpn.com/fedora-33-unstable/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://repo.protonvpn.com/fedora-33-unstable/public_key.asc
EOF
dnf install protonvpn
```

### MEGA  
Note that it doesn't work at the time of writing.  
```
sudo dnf install megasync nautilus-megasync
```
If so, consider downloading from official website.
```
https://mega.io/sync
```

### Notion Enhanced  
```
wget https://notion.davidbailey.codes/notion-linux.repo
sudo mv notion-linux.repo /etc/yum.repos.d/notion-linux.repo
sudo dnf install notion-enhanced
```

### PulseEffects Preset  
Launch PulseEffects at least once before execute.  
```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/JackHack96/PulseEffects-Presets/master/install.sh)"
```

### X11 Gestures  
GNOME 40 gestures won't work under X11 out of the box  
```
https://github.com/JoseExposito/gnome-shell-extension-x11gestures
```

### Uget browser intergration  
```
wget https://raw.githubusercontent.com/ugetdm/uget-integrator/master/install/linux/install_uget_integrator.sh
chmod +x install_uget_integrator.sh
./install_uget_integrator.sh
```

### Zsh configuration  
```
https://github.com/sorin-ionescu/prezto
```

### IME  
```
# Chinese (Copr ver. with cloud-input)
sudo dnf remove ibus-libpinyin
sudo dnf copr enable pwu/cloud-input
sudo dnf install ibus-libpinyin
# Japanese
sudo dnf install ibus-mozc
# Emoji
sudo dnf install ibus-uniemoji
```

### Keymap for `ibus-mozc` (US keyboard)  
Save as text file, then import to Mozc's setting  
```
status	key	command
Composition	Backspace	Backspace
Composition	Ctrl a	MoveCursorToBeginning
Composition	Ctrl Backspace	Backspace
Composition	Ctrl d	MoveCursorRight
Composition	Ctrl Down	MoveCursorToEnd
Composition	Ctrl e	MoveCursorToBeginning
Composition	Ctrl Enter	Commit
Composition	Ctrl f	MoveCursorToEnd
Composition	Ctrl g	Delete
Composition	Ctrl h	Backspace
Composition	Ctrl i	ConvertToFullKatakana
Composition	Ctrl k	MoveCursorLeft
Composition	Ctrl l	MoveCursorRight
Composition	Ctrl Left	MoveCursorToBeginning
Composition	Ctrl m	Commit
Composition	Ctrl n	MoveCursorToEnd
Composition	Ctrl o	ConvertToHalfWidth
Composition	Ctrl p	ConvertToFullAlphanumeric
Composition	Ctrl Right	MoveCursorToEnd
Composition	Ctrl s	MoveCursorLeft
Composition	Ctrl Shift Space	InsertFullSpace
Composition	Ctrl Space	InsertHalfSpace
Composition	Ctrl t	ConvertToHalfAlphanumeric
Composition	Ctrl u	ConvertToHiragana
Composition	Ctrl Up	MoveCursorToBeginning
Composition	Ctrl x	MoveCursorToEnd
Composition	Ctrl z	Cancel
Composition	Delete	Delete
Composition	Down	MoveCursorToEnd
Composition	Eisu	ToggleAlphanumericMode
Composition	End	MoveCursorToEnd
Composition	Enter	Commit
Composition	ESC	Cancel
Composition	F10	ConvertToHalfAlphanumeric
Composition	F2	ConvertWithoutHistory
Composition	F6	ConvertToHiragana
Composition	F7	ConvertToFullKatakana
Composition	F8	ConvertToHalfWidth
Composition	F9	ConvertToFullAlphanumeric
Composition	Hankaku/Zenkaku	IMEOff
Composition	Henkan	Convert
Composition	Hiragana	InputModeHiragana
Composition	Home	MoveCursorToBeginning
Composition	Katakana	InputModeFullKatakana
Composition	Left	MoveCursorLeft
Composition	Muhenkan	SwitchKanaType
Composition	Right	MoveCursorRight
Composition	Shift Backspace	Backspace
Composition	Shift ESC	Cancel
Composition	Shift Left	MoveCursorLeft
Composition	Shift Muhenkan	ConvertToFullAlphanumeric
Composition	Shift Right	MoveCursorRight
Composition	Shift Space	Convert
Composition	Space	Convert
Composition	Tab	PredictAndConvert
Conversion	Backspace	Cancel
Conversion	Ctrl a	SegmentFocusFirst
Conversion	Ctrl Backspace	Cancel
Conversion	Ctrl d	SegmentFocusRight
Conversion	Ctrl Down	CommitOnlyFirstSegment
Conversion	Ctrl e	ConvertPrev
Conversion	Ctrl Enter	Commit
Conversion	Ctrl f	SegmentFocusLast
Conversion	Ctrl g	Cancel
Conversion	Ctrl h	Cancel
Conversion	Ctrl i	ConvertToFullKatakana
Conversion	Ctrl k	SegmentWidthShrink
Conversion	Ctrl l	SegmentWidthExpand
Conversion	Ctrl Left	SegmentFocusFirst
Conversion	Ctrl m	Commit
Conversion	Ctrl n	CommitOnlyFirstSegment
Conversion	Ctrl o	ConvertToHalfWidth
Conversion	Ctrl p	ConvertToFullAlphanumeric
Conversion	Ctrl Right	SegmentFocusLast
Conversion	Ctrl s	SegmentFocusLeft
Conversion	Ctrl Shift Space	InsertFullSpace
Conversion	Ctrl Space	InsertHalfSpace
Conversion	Ctrl t	ConvertToHalfAlphanumeric
Conversion	Ctrl u	ConvertToHiragana
Conversion	Ctrl Up	ConvertPrev
Conversion	Ctrl x	ConvertNext
Conversion	Ctrl z	Cancel
Conversion	Delete	Cancel
Conversion	Down	ConvertNext
Conversion	Eisu	ToggleAlphanumericMode
Conversion	End	SegmentFocusLast
Conversion	Enter	Commit
Conversion	ESC	Cancel
Conversion	F10	ConvertToHalfAlphanumeric
Conversion	F6	ConvertToHiragana
Conversion	F7	ConvertToFullKatakana
Conversion	F8	ConvertToHalfWidth
Conversion	F9	ConvertToFullAlphanumeric
Conversion	Hankaku/Zenkaku	IMEOff
Conversion	Henkan	ConvertNext
Conversion	Hiragana	InputModeHiragana
Conversion	Home	SegmentFocusFirst
Conversion	Katakana	InputModeFullKatakana
Conversion	Left	SegmentFocusLeft
Conversion	Muhenkan	SwitchKanaType
Conversion	PageDown	ConvertNextPage
Conversion	PageUp	ConvertPrevPage
Conversion	Right	SegmentFocusRight
Conversion	Shift Backspace	Cancel
Conversion	Shift Down	ConvertNextPage
Conversion	Shift ESC	Cancel
Conversion	Shift Henkan	ConvertPrev
Conversion	Shift Left	SegmentWidthShrink
Conversion	Shift Muhenkan	ConvertToFullAlphanumeric
Conversion	Shift Right	SegmentWidthExpand
Conversion	Shift Space	ConvertPrev
Conversion	Shift Tab	ConvertPrev
Conversion	Shift Up	ConvertPrevPage
Conversion	Space	ConvertNext
Conversion	Tab	PredictAndConvert
Conversion	Up	ConvertPrev
DirectInput	Eisu	IMEOn
DirectInput	F13	IMEOn
DirectInput	Hankaku/Zenkaku	IMEOn
DirectInput	Henkan	Reconvert
DirectInput	Hiragana	IMEOn
DirectInput	Katakana	IMEOn
Precomposition	Backspace	Revert
Precomposition	Ctrl Backspace	Undo
Precomposition	Ctrl Shift Space	InsertFullSpace
Precomposition	Eisu	ToggleAlphanumericMode
Precomposition	Hankaku/Zenkaku	IMEOff
Precomposition	Hiragana	InputModeHiragana
Precomposition	Henkan	Reconvert
Precomposition	Katakana	InputModeFullKatakana
Precomposition	Muhenkan	InputModeSwitchKanaType
Precomposition	Shift Muhenkan	ToggleAlphanumericMode
Precomposition	Shift Space	InsertAlternateSpace
Precomposition	Space	InsertSpace
Prediction	Ctrl Delete	DeleteSelectedCandidate
Suggestion	Down	PredictAndConvert
Suggestion	Shift Enter	CommitFirstSuggestion
DirectInput	Ctrl Space	IMEOn
Precomposition	Ctrl Space	IMEOff
Composition	ASCII	InsertCharacter
Composition	Kanji	IMEOff
Composition	OFF	IMEOff
Composition	ON	IMEOn
Conversion	Kanji	IMEOff
Conversion	OFF	IMEOff
Conversion	ON	IMEOn
DirectInput	Kanji	IMEOn
DirectInput	ON	IMEOn
Precomposition	ASCII	InsertCharacter
Precomposition	Kanji	IMEOff
Precomposition	OFF	IMEOff
Precomposition	ON	IMEOn
```

## 8. Developers (Optional)
### JetBrains  
```
https://www.jetbrains.com/toolbox-app/
```

### VSCode  
```
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
sudo dnf install code
```

### Machine Learning (CUDA Toolkit and ML repo)  
```
sudo dnf config-manager --add-repo http://developer.download.nvidia.com/compute/cuda/repos/fedora33/x86_64/cuda-fedora33.repo
# Optional, but safety first
echo "exclude=akmod-nvidia*,kmod-nvidia*,*nvidia*,nvidia-*,cuda-nvidia-kmod-common,dkms-nvidia,nvidia-libXNVCtrl" | sudo tee -a /etc/yum.repos.d/cuda-fedora33.repo
sudo dnf clean all
sudo dnf install cuda

sudo dnf install https://developer.download.nvidia.com/compute/machine-learning/repos/rhel8/x86_64/nvidia-machine-learning-repo-rhel8-1.0.0-1.x86_64.rpm
sudo dnf install libcudnn8 libcudnn8-devel libnccl libnccl-devel
```