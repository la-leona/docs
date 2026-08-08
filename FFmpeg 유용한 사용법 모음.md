# FFmpeg 유용한 사용법 모음

FFmpeg은 강력한 멀티미디어 처리 도구이지만, 명령어가 복잡할 수 있습니다. 실무에서 자주 사용되는 유용한 사용법들을 정리했습니다.

## 1. 기본 비디오 변환

### 비디오 형식 변환
```bash
ffmpeg -i input.avi output.mp4
```

### 비디오 품질 설정 (CRF 값: 0-51, 낮을수록 높은 품질)
```bash
ffmpeg -i input.mp4 -crf 23 output.mp4
```

### 비디오 해상도 변경
```bash
ffmpeg -i input.mp4 -vf scale=1280:720 output.mp4
```

### 비디오 프레임 레이트 변경
```bash
ffmpeg -i input.mp4 -r 30 output.mp4
```

## 2. 오디오 처리

### 비디오에서 오디오만 추출
```bash
ffmpeg -i input.mp4 -q:a 0 -map a output.mp3
```

### 오디오 비트레이트 설정
```bash
ffmpeg -i input.mp3 -b:a 192k output.mp3
```

### 오디오 샘플링 레이트 변경
```bash
ffmpeg -i input.mp3 -ar 44100 output.mp3
```

### 비디오에 오디오 추가
```bash
ffmpeg -i video.mp4 -i audio.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 output.mp4
```

### 오디오 음량 조정 (1.0 = 원본, 2.0 = 2배)
```bash
ffmpeg -i input.mp3 -af "volume=1.5" output.mp3
```

## 3. 비디오 편집

### 비디오 자르기 (10초부터 30초까지)
```bash
ffmpeg -i input.mp4 -ss 10 -to 30 -c copy output.mp4
```

### 비디오 회전 (시계방향 90도)
```bash
ffmpeg -i input.mp4 -vf "transpose=1" output.mp4
```

### 비디오 속도 변경 (2배 빠르게)
```bash
ffmpeg -i input.mp4 -filter:v "setpts=0.5*PTS" output.mp4
```

### 비디오 속도 변경 (0.5배 느리게)
```bash
ffmpeg -i input.mp4 -filter:v "setpts=2.0*PTS" output.mp4
```

### 비디오 반전 (좌우 반전)
```bash
ffmpeg -i input.mp4 -vf hflip output.mp4
```

### 비디오 상하 반전
```bash
ffmpeg -i input.mp4 -vf vflip output.mp4
```

## 4. 이미지 처리

### 비디오에서 프레임 추출 (1초마다)
```bash
ffmpeg -i input.mp4 -vf fps=1 frame_%03d.png
```

### 특정 시간의 프레임 추출 (10초 지점)
```bash
ffmpeg -i input.mp4 -ss 10 -vframes 1 thumbnail.png
```

### 이미지 시퀀스로 비디오 생성 (1초마다 1프레임)
```bash
ffmpeg -framerate 1 -i frame_%03d.png output.mp4
```

### 이미지 크기 변경
```bash
ffmpeg -i input.jpg -vf scale=1920:1080 output.jpg
```

## 5. 자막 처리

### 자막 파일 추출
```bash
ffmpeg -i input.mkv -map 0:s:0 subtitle.srt
```

### 자막 파일 삽입 (하드서브: 영구적으로 적용)
```bash
ffmpeg -i input.mp4 -vf "subtitles=subtitle.srt" output.mp4
```

### 자막 파일 포함 (소프트서브: 선택 가능)
```bash
ffmpeg -i input.mp4 -i subtitle.srt -c:s mov_text output.mp4
```

## 6. 배치 처리 및 자동화

### 폴더의 모든 AVI 파일을 MP4로 변환 (Windows)
```bash
for %i in (*.avi) do ffmpeg -i "%i" "%~ni.mp4"
```

### 폴더의 모든 AVI 파일을 MP4로 변환 (Linux/Mac)
```bash
for file in *.avi; do ffmpeg -i "$file" "${file%.avi}.mp4"; done
```

### 진행 상황 표시하며 변환
```bash
ffmpeg -i input.mp4 -progress pipe:1 output.mp4
```

## 7. 고급 기능

### 워터마크 추가
```bash
ffmpeg -i input.mp4 -i watermark.png -filter_complex "overlay=10:10" output.mp4
```

### 여러 비디오 연결
```bash
ffmpeg -f concat -safe 0 -i filelist.txt -c copy output.mp4
```
(filelist.txt 내용: `file 'video1.mp4'` 및 `file 'video2.mp4'` 각각 한 줄씩)

### 비디오 스케일 및 패딩 추가
```bash
ffmpeg -i input.mp4 -vf "scale=1280:720:force_original_aspect_ratio=decrease,pad=1280:720:(ow-iw)/2:(oh-ih)/2" output.mp4
```

### 비디오 인코딩 속도 조정 (preset: ultrafast, superfast, veryfast, faster, fast, medium, slow, slower, veryslow)
```bash
ffmpeg -i input.mp4 -preset fast -crf 23 output.mp4
```

### 비디오 정보 확인 (코덱, 해상도, 프레임 레이트 등)
```bash
ffmpeg -i input.mp4
```

## 8. 실무 팁

### 1. 빠른 변환이 필요할 때
```bash
ffmpeg -i input.mp4 -c:v libx264 -preset ultrafast -crf 28 output.mp4
```

### 2. 고품질 변환이 필요할 때
```bash
ffmpeg -i input.mp4 -c:v libx264 -preset slow -crf 18 output.mp4
```

### 3. 원본 파일 손상 없이 변환
```bash
ffmpeg -i input.mp4 -c copy output.mp4
```
(코덱 재인코딩 없이 스트림만 복사 - 매우 빠름)

### 4. 여러 옵션 조합 (고품질 MP4 생성)
```bash
ffmpeg -i input.avi -c:v libx264 -crf 23 -preset medium -c:a aac -b:a 192k output.mp4
```

### 5. 배치 처리 시 오류 무시하고 계속 진행
```bash
ffmpeg -i input.mp4 output.mp4 -y
```
(-y 옵션: 기존 파일 덮어쓰기)

### 6. 조용하게 처리 (로그 최소화)
```bash
ffmpeg -i input.mp4 output.mp4 -loglevel quiet
```

## 9. 자주 사용되는 옵션

| 옵션 | 설명 | 예시 |
| :--- | :--- | :--- |
| `-i` | 입력 파일 | `-i input.mp4` |
| `-o` | 출력 파일 | `-o output.mp4` |
| `-c:v` | 비디오 코덱 | `-c:v libx264` |
| `-c:a` | 오디오 코덱 | `-c:a aac` |
| `-crf` | 품질 (0-51) | `-crf 23` |
| `-b:v` | 비디오 비트레이트 | `-b:v 5000k` |
| `-b:a` | 오디오 비트레이트 | `-b:a 192k` |
| `-r` | 프레임 레이트 | `-r 30` |
| `-s` | 해상도 | `-s 1280x720` |
| `-ss` | 시작 시간 | `-ss 10` |
| `-t` | 지속 시간 | `-t 30` |
| `-vf` | 비디오 필터 | `-vf scale=1280:720` |
| `-af` | 오디오 필터 | `-af volume=1.5` |
| `-preset` | 인코딩 속도 | `-preset fast` |
| `-y` | 기존 파일 덮어쓰기 | `-y` |

## 10. 결론

FFmpeg은 명령어가 많고 복잡해 보이지만, 자주 사용하는 패턴을 익히면 매우 강력한 도구입니다. 위의 예시들을 참고하여 자신의 필요에 맞게 조합하여 사용하면, 대부분의 멀티미디어 처리 작업을 효율적으로 수행할 수 있습니다.
