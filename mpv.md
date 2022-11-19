# MPV tips
### Update youtube-dl on stable distro to keep MPV functional
If you are using MPV + youtube-dl to play video/audio from a website and getting this error:
````
$ mpv link_to_youtube_vid --no-video
> [ffmpeg] https: HTTP error 403 Forbidden
  Failed to open https://rr15---sn-axq7sn76.googlevideo.com/videoplayback?expire=...
  ...
````
Update youtube-dl to the recent version to make it work:
````
$ sudo wget https://yt-dl.org/downloads/latest/youtube-dl -O /usr/local/bin/youtube-dl
$ sudo chmod a+rx /usr/local/bin/youtube-dl
````
