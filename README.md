# Background
This repo contains detailed instructions and examples on the process of ripping and converting
Blu-ray movies for eventual use in applications such as a *Plex* media server.

The steps have been performed in *Fedora Server* (currently release 33). While the steps can
be accomplished in other *linux*-based distros, these rely on *Snaps* or *Flatpacks*, which I have
so far found to be unreliable in this context.

This guide will make use of a custom repo for *Fedora*, which is the easiest and most reliable way
to obtain the necessary tools.

# Install Required Tools
The Multimedia repo available at [https://negativo17.org](https://negativo17.org) contains
both *MakeMKV* and *HandBrake*, the two tools used in this guide, pre-compiled for *Fedora*.

Add the Multimedia repo with the command:
```bash
dnf config-manager --add-repo=https://negativo17.org/repos/fedora-multimedia.repo
```

## MakeMKV
[MakeMKV](https://www.makemkv.com/) is currently the best tool that exists for ripping Blu-ray
movies. It has can be used for free as a beta with a temporary code, or a license can be purchased
for $50. This guide assumes that a license has been purchased (recommended, as the developer has
made an incredible tool and made it freely available).

Install *MakeMKV* with the command:
```bash
dnf install makemkv
```

When running a headless server, as is the case in this guide, add the purchased license code via
the following steps:
1. Create the folder *.MakeMKV* in your *home* directory if it doesn't already exist, e.g.,
   `mkdir ~/.MakeMKV`
2. Create the file *settings.conf* with the single line:
   ```bash
   app_Key="my-purchased-key-123"
   ```

## HandBrake
[HandBrake](https://handbrake.fr) is an open-source video transcoder available for multiple platforms.
It is currently the best and most feature rich of all video transcoder currently available.

*HandBrake* will be used to convert the extracted video to *H.264* format, which will compress the
video to about 10% of its original size without any loss of Blu-ray quality.

*HandBrake* includes both a GUI and a CLI version. As the operating system used in this case is a
headless server, only the CLI version will be installed.

Install the CLI tool with the command:
```bash
dnf install HandBrake-cli
```

*Fedora* does not give a user accounts access to optical disk drives (i.e., Blu-ray, DVD, CD) by
default. To fix this, add your user to the *cdrom* group with the command:
```bash
usermod -a -G cdrom username
```

# Extract Movie from Blu-ray Disk
The CLI tool used to rip movies with *MakeMKV* is named *makemkvcon*.
The command below can be used to display all available titles in a disk:
```bash
makemkvcon -r --decrypt --cache=1024 info disc:1
```
If the above command doesn't return any results, try re-running it replacing *disc:1* with
*disc:0*.

*makemkvcon* uses a zero index for title numbers (e.g., first title == 0).
The command below will extract the first title into the *movies* folder:
```bash
makemkvcon -r --decrypt --cache=1024 mkv disc:1 0 movies
```

The above commands will display too much information, which makes it difficult to locate the
titles. Alternatively, a *Bash* script such as the one below can be used to disply only relevant lines and
select the correct title:
```bash
#! /bin/bash

makemkvcon -r --decrypt --cache=1024 info disc:1 | \
  grep -E 'TINFO:[0-9]+,30.*chapter'

read -p 'TRACK NUMBER TO EXTRACT: ' track

makemkvcon -r --decrypt --cache=1024 mkv disc:1 $track .
```

# Transcode movie
To display information about the video to be compressed, use the command below:
```bash
HandBrakeCLI -i My_Movie.mkv -t 1 --scan
```

## Convert video
The shell script below will convert the file *Wonder_Woman_1984.mkv*, using audio tracks 1 and 6,
subtitle tracks 1 and 3, and save the file to the folder */data/Movies*:
```bash
#! /bin/bash
movie=Wonder_Woman_1984.mkv
audio="1,6"
subt="1,3"
movdir=/data/Movies
preset="Matroska/H.264 MKV 1080p30"

input=/home/abe/Bluray/${movie}
output=/home/abe/${movdir}/${movie}
HandBrakeCLI --preset "${preset}" -i ${input} -t 1 -o ${output} -m \
  -a "${audio}" -s "${subt}"
```
