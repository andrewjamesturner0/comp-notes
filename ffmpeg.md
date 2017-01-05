---
title: ffmpeg
author: Andrew Turner
date: 2015-08-21
---

## Join videos together

Create a text file containing a list of the file to be joined together
using the following format:

    file path/to/file0.mkv
    file path/to/file1.mp4

Pass this file to ffmpeg with the `concat` option:

    ffmpeg -f concat -i mylist.txt output.mp4


## Re-encode video

    ffmpeg -i "input.mp4" -b:v 3000k -b:a 192k -strict -2 "output.mp4"

*  -b:v -- set the video bitrate
*  -b:a -- set the audio bitrate

