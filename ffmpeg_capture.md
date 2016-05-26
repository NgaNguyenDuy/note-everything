Use the [x11grab](https://ffmpeg.org/ffmpeg-devices.html#x11grab) device:

`ffmpeg -video_size 1024x768 -framerate 25 -f x11grab -i :0.0+100,200 /tmp/output.mp4`
This will grab the image from desktop, starting with the upper-left corner at (x=100, y=200) with the width and height of 1024x768.

- If you need audio too you can use ALSA (see Capture/ALSA for more info):

    `ffmpeg -video_size 1024x768 -framerate 25 -f x11grab -i :0.0+100,200 -f alsa -ac 2 -i hw:0 /tmp/output.mkv`
- Or the pulse input device:

    `ffmpeg -video_size 1024x768 -framerate 25 -f x11grab -i :0.0+100,200 -f pulse -ac 2 -i default /tmp/output.mkv`

Todo:
- [ ] Grab the region to screencast.
- [ ] Allow select storage location.
