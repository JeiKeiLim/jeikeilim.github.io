---
title: "FFMPEG cheat sheets"
last_modified_at: 2022-03-07T13:47
categories:
  - Blog
tags:
  - ffmpeg
  - Video
  - CheatSheet
toc: true
toc_sticky: true
---

# 1. Video Conversion

## 1.1 Basic

### 1.1.1. MOV to MP4

```python
ffmpeg -i /source/path /output/path
```

### 1.1.2. Change FPS

```python
ffmpeg -i <input> -filter:v fps=fps=30 <output>
```

### 1.1.3. Scale Change

```python
ffmpeg -i <input> -vf scale=320:240 <output>

ffmpeg -i <input> -vf scale=320:-1 <output>
```

### 1.1.4. No Audio

```python
ffmpeg -i <input> -an <output>
```

### 1.1.5 Crop Video

```python
# ffmpeg -i in.mp4 -filter:v "crop=out_w:out_h:x:y" out.mp4

# Specific Location
ffmpeg -i <input> -filter:v "crop=200:100:300:100" <output>

# Crop Only Top of specific pixel
ffmpeg -i <input> -filter:v "crop=iw:ih-20:0:ih" <output>

```

## 1.2 Advanced

### 1.2.1 비디오 재생 속도 조절하기

```python
ffmpeg -i input.mkv -filter:v "setpts=0.5*PTS" output.mk
```
(0.5 - 조절하는 속도 (2배속))

* Audio 가 있는 경우
```python
ffmpeg -i input.mkv -filter_complex "[0:v]setpts=0.5*PTS[v];[0:a]atempo=2.0[a]" -map "[v]" -map "[a]" output.mkv
```

(0.5 - 영상 속도 (2배속))
(2.0 - 오디오 속도 (2배속))

### 1.2.2 Video stack vertically

```python
ffmpeg -i input0 -i input1 -filter_complex vstack=inputs=2 output
```

![Stack vertical](/assets/images/2022-03-07-ffmpeg-cheat-sheet/03_top_bottom.png){: width="25%"}

### 1.2.3 Video stack horizontally

```python
ffmpeg -i input0 -i input1 -filter_complex hstack=inputs=2 output
```

![Stack vertical](/assets/images/2022-03-07-ffmpeg-cheat-sheet/01_left_right.png){: width="50%"}

### 1.2.4 Video stack with a border

```python
ffmpeg -i input0 -i input1 -filter_complex "[0]pad=iw+5:color=black[left];[left][1]hstack=inputs=2" output
```

![Stack vertical](/assets/images/2022-03-07-ffmpeg-cheat-sheet/02_left_right_border.png){: width="50%"}

Further details can be found here [https://stackoverflow.com/questions/11552565/vertically-or-horizontally-stack-mosaic-several-videos-using-ffmpeg](https://stackoverflow.com/questions/11552565/vertically-or-horizontally-stack-mosaic-several-videos-using-ffmpeg){:target='_blank'}

### 1.2.5 Video 2x2 grid with text

```python
ffmpeg -i 2019_1231_face_test.mov -i test_result_noone.mp4 -i test_result_light_DSFD.mp4 -i test_result_DSFD.mp4 -filter_complex "[0]drawtext=text='(Original)':borderw=5:bordercolor='WhiteSmoke':fontsize=100:x=w-text_w-10:y=h-text_h-20[v0]; [1]drawtext=text='(Noone video)':borderw=5:bordercolor='WhiteSmoke':fontsize=100:x=10:y=h-text_h-30[v1]; [2]drawtext=text='(lightDSFD)':borderw=5:bordercolor='WhiteSmoke':fontsize=100:x=w-text_w-10:y=20[v2]; [3]drawtext=text='(DSFD)':borderw=5:bordercolor='WhiteSmoke':fontsize=100:x=10:y=20[v3]; [v0][v1][v2][v3]xstack=inputs=4:layout=0_0|w0_0|0_h0|w0_h0[v]" -map "[v]" output_grid.mp4
```

### 1.2.6 Create Images from Video

```python
ffmpeg -i <input> -filter:v fps=1 <output%03d.jpg>
```

### 1.2.7. Create Video from Images

```python
ffmpeg -framerate 30 -pattern_type glob -i '*.png' -c:v libx264 -pix_fmt yuv420p out.mp4
```

### 1.2.8. Download m3u8 playlist video

```python
ffmpeg -protocol_whitelist file,http,https,tcp,tls,crypto -i my_movie.m3u8 -c copy my_movie.mp4
```

### 1.2.9. Using GPU

```python
ffmpeg -hwaccel cuvid -c:v h264_cuvid -i input.MOV -c:v h264_nvenc -b:v 10240k output.mp4
```

- Use `ffmpeg -hwaccels` to check supported hardware


# 2. GIF

## 2.1 Basic

```python
ffmpeg -ss 0 -t 12 -i input.mov -vf "fps=2,scale=2048:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop 0 output.gif
```

ss - 시작 지점

- t 끝나는 지점

fps - 프레임 속도

scale = 가로 해상도:세로 해상도

## 2.2 Multiple Images to gif

```python
ffmpeg -f image2 -i %02d.png -vf scale=900:-1:sws_dither=ed,palettegen palette.png
ffmpeg -f image2 -framerate 2 -i %02d.png video.flv
ffmpeg -i video.flv -i palette.png -filter_complex "fps=2,scale=430:-1:flags=lanczos[x];[x][1:v]paletteuse" out.gif

rm video.flv palette.png
```

## 2.3 Lossy GIF

[https://github.com/kohler/gifsicle](https://github.com/kohler/gifsicle){:target='_blank'}

```bash
brew install gifsicle

gifsicle -O9 --lossy=200 -o output.gif input.gif
# 200 is highest
```
