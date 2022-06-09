# ffmpeg tips by Frodo<br>
## Escape filename in video filters (Wndows) example
````
-vf colormatrix=bt709:bt601,subtitles="C\\:\\\\Users\\\\frodo\\\\Downloads\\\\\[SubsPlease\] Hakozume - Kouban Joshi no Gyakushuu - 06 \(720p\) \[2908017A\].mkv":si=0
````
## Lossless screen recording & forced keyframes every 3 seconds (Linux) example:
Keyframes are needed for smooth video rewind, they help to cut a video with the exact time you need
````
ffmpeg -video_size 1920x1080 -framerate 30 -f x11grab -i :0.0 -force_key_frames "expr:gte(t,n_forced*3)" -c:v libx264rgb -crf 0 -preset ultrafast -color_range 2 output.mkv
````
