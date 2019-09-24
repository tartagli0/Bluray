# Extract Movie from Blu-ray Disk
[MakeMKV](https://www.makemkv.com) is able to extract titles from a Blu-ray disk
via a GUI or through its command-line tool [makemkvcon](https://www.makemkv.com/developers/usage.txt).

## Show titles
To command below displays information about */dev/sr0* (*disc:1*):
```bash
makemkvcon -r --decrypt --cache=1024 info disc:1
```

## Extract title
*makemkvcon* uses a zero index for title numbers (e.g., first title == 0).
The command below will extract the first title into the *movies* folder:
```bash
makemkvcon -r --decrypt --cache=1024 mkv disc:1 0 movies
```

# Convert Movie with HandBrake
Use [HandBrake](https://handbrake.fr) to convert the extracted video to *H.264*
format, which will compress the video without loss of Blu-ray quality.

## Show file info
To display information about the video to be compressed, use the command below:
```bash
HandBrakeCLI -i My_Movie.mkv -t 1 --scan
```

## Convert video
The shell script below will convert the file *Aladdin.mkv*, with audio tracks 2 and 5,
to the folder */data/Movies*:
```bash
#! /bin/bash
movie=Aladdin_2019.mkv
output=/data/Kids_Movies/${movie}
audio="2,5"
preset="H.264 MKV 1080p30"

HandBrakeCLI --preset "${preset}" -i ${input} -t 1 -o ${output} -m -a "${audio}"
```
