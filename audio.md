
# Creating Audio CDs on Ubuntu Using the Terminal

This guide explains how to create a **standard music CD** (playable in most CD players) using command-line tools on Ubuntu. It covers installation, conversion, normalization, device detection, burning, and troubleshooting.

---

## Prerequisites

Install the required packages:

```bash
sudo apt-get update
sudo apt-get install ffmpeg normalize-audio wodim
```

- **ffmpeg** → Converts audio files (MP3, FLAC, etc.) to WAV format.  
- **normalize-audio** → Adjusts volume levels across tracks.  
- **wodim** → Burns audio tracks to CD.  

---

## Step 1: Convert Audio Files to WAV

Audio CDs require **44.1 kHz, 16-bit, stereo WAV files**.

Navigate to your music folder:

```bash
cd ~/Music/MyAlbum
```

Convert all MP3 files to WAV:

```bash
for f in *.mp3; do ffmpeg -i "$f" -ar 44100 -ac 2 -sample_fmt s16 "${f%.mp3}.wav"; done
```

- `-ar 44100` → Sample rate 44.1 kHz  
- `-ac 2` → Stereo  
- `-sample_fmt s16` → 16-bit PCM  

---

## Step 2: Normalize Audio Levels (Optional)

To ensure consistent volume across tracks:

```bash
normalize-audio -m *.wav
```

---

## Step 3: Identify Your CD Burner Device

Check which device corresponds to your CD/DVD drive:

```bash
wodim dev=/dev/sr0 -checkdrive
```

Typical device path: `/dev/sr0`.

---

## Step 4: Burn WAV Files to CD

Insert a blank **CD-R** (recommended for compatibility).

Burn the tracks:

```bash
sudo wodim -v -eject dev=/dev/sr0 speed=8 -sao -audio -pad *.wav
```

### Explanation of options:
- `-v` → Verbose output  
- `-eject` → Eject disc after burning  
- `dev=/dev/sr0` → Device path for burner  
- `speed=8` → Burn speed (lower speeds = safer burns)  
- `-sao` → Session At Once mode (better for audio CDs)  
- `-audio` → Burn as audio CD tracks  
- `-pad` → Add padding for sector alignment  

---

## Step 5: Verify the Burned CD

After burning, check the disc contents:

```bash
wodim -toc dev=/dev/sr0
```

This displays the **Table of Contents** (tracks and lengths).

---

## Notes & Tips

- Use **CD-R** instead of CD-RW for maximum compatibility.  
- Track order depends on **file order in the command**:
  - `*.wav` expands alphabetically.  
  - Rename files with numbers (`01-Track.wav`, `02-Track.wav`) or list them manually.  
- Padding warnings (`WARNING: padding up to secsize`) are normal and harmless.  
- DVDs cannot be used for audio CDs. Burning WAVs to a DVD creates a **data DVD**, not a playable audio CD.  

---

## Troubleshooting

- **Drive not detected** → Use `lsblk` or `dmesg | grep -i cd` to confirm hardware.  
- **Permission issues** → Run with `sudo` or add user to `cdrom` group:  
  ```bash
  sudo usermod -aG cdrom $USER
  ```
  Then log out and back in.  
- **Buffer underruns** → Lower burn speed (`speed=4`).  
- **Order problems** → Explicitly list files in the desired order instead of relying on `*.wav`.  

