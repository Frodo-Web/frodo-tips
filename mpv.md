# MPV tips
### Update youtube-dl on stable distro to keep MPV functional
If you are using MPV + youtube-dl to play video/audio from a website and getting this error:
````
$ mpv link_to_youtube_vid --no-video
> [ffmpeg] https: HTTP error 403 Forbidden
  Failed to open https://rr15---sn-axq7sn76.googlevideo.com/videoplayback?expire=...
  ...
````
Figure out youtube-dl location and update it to the recent version to make it work:
````
$ whereis youtube-dl
> youtube-dl: /usr/local/bin/youtube-dl

$ sudo wget https://yt-dl.org/downloads/latest/youtube-dl -O /usr/local/bin/youtube-dl  // Use your own path here
$ sudo chmod a+rx /usr/local/bin/youtube-dl
````
