# yt-dlp로 YouTube 비디오 다운로드하기 (MP4, 720p)

YouTube에서 MP4 형식, 720p 해상도로 `C:/tmp` 폴더에 다운로드받는 yt-dlp 명령어를 소개합니다.

## 기본 명령어

```bash
yt-dlp -f "bestvideo[height<=720]+bestaudio/best" -o "C:/tmp/%(title)s.%(ext)s" "https://www.youtube.com/watch?v=VIDEO_ID"
```

## 명령어 설명

각 옵션의 의미는 다음과 같습니다:

*   **`-f "bestvideo[height<=720]+bestaudio/best"`**: 형식 선택 옵션입니다. 720p 이하의 최고 품질 비디오와 최고 품질 오디오를 선택하여 병합합니다. 만약 720p가 없으면 더 낮은 해상도를 선택하고, 오디오가 별도 스트림으로 없으면 비디오에 포함된 오디오를 사용합니다.
*   **`-o "C:/tmp/%(title)s.%(ext)s"`**: 출력 경로 및 파일명 템플릿입니다. 다운로드된 파일이 `C:/tmp` 폴더에 저장되며, 파일명은 비디오의 제목으로 자동 설정됩니다.
*   **`"https://www.youtube.com/watch?v=VIDEO_ID"`**: 다운로드할 YouTube 비디오의 URL입니다. `VIDEO_ID` 부분을 실제 비디오 ID로 바꿔야 합니다.

## 사용 예시

실제 YouTube 비디오를 다운로드하려면:

```bash
yt-dlp -f "bestvideo[height<=720]+bestaudio/best" -o "C:/tmp/%(title)s.%(ext)s" "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
```

## 추가 옵션

### MP4 형식 강제

항상 MP4로 저장하려면 다음 명령어를 사용하세요:

```bash
yt-dlp -f "bestvideo[height<=720][ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]" -o "C:/tmp/%(title)s.%(ext)s" "https://www.youtube.com/watch?v=VIDEO_ID"
```

### 병합 후 원본 파일 삭제

FFmpeg이 설치되어 있다면, 비디오와 오디오를 자동으로 병합합니다. 병합 후 원본 파일을 삭제하려면:

```bash
yt-dlp -f "bestvideo[height<=720]+bestaudio/best" --merge-output-format mp4 -o "C:/tmp/%(title)s.%(ext)s" "https://www.youtube.com/watch?v=VIDEO_ID"
```

### 진행 상황 표시

다운로드 진행 상황을 더 자세히 보려면:

```bash
yt-dlp -f "bestvideo[height<=720]+bestaudio/best" -o "C:/tmp/%(title)s.%(ext)s" --progress "https://www.youtube.com/watch?v=VIDEO_ID"
```

## 주의사항

*   **FFmpeg 필수**: 비디오와 오디오를 병합하려면 FFmpeg이 설치되어 있어야 합니다. FFmpeg이 없으면 별도의 비디오 및 오디오 파일이 생성될 수 있습니다.
*   **폴더 존재 확인**: `C:/tmp` 폴더가 존재하는지 확인하세요. 폴더가 없으면 명령어 실행 전에 생성해야 합니다.
*   **비디오 ID 확인**: YouTube URL에서 `v=` 뒤의 문자열이 비디오 ID입니다. 예를 들어, `https://www.youtube.com/watch?v=dQw4w9WgXcQ`에서 비디오 ID는 `dQw4w9WgXcQ`입니다.

## 자주 사용되는 옵션

| 옵션 | 설명 |
| :--- | :--- |
| `-f FORMAT` | 다운로드할 비디오 형식 지정 |
| `-o OUTPUT` | 출력 파일명 및 경로 지정 |
| `-x` | 오디오만 추출 |
| `--write-subs` | 자막 다운로드 |
| `--sub-lang LANG` | 자막 언어 지정 (예: ko, en) |
| `-U` | yt-dlp 업데이트 |
| `--list-formats` | 사용 가능한 모든 형식 표시 |

## 형식 선택 예시

### 최고 품질 다운로드 (모든 해상도)

```bash
yt-dlp -f "bestvideo+bestaudio/best" -o "C:/tmp/%(title)s.%(ext)s" "https://www.youtube.com/watch?v=VIDEO_ID"
```

### 1080p 다운로드

```bash
yt-dlp -f "bestvideo[height<=1080]+bestaudio/best" -o "C:/tmp/%(title)s.%(ext)s" "https://www.youtube.com/watch?v=VIDEO_ID"
```

### 480p 다운로드

```bash
yt-dlp -f "bestvideo[height<=480]+bestaudio/best" -o "C:/tmp/%(title)s.%(ext)s" "https://www.youtube.com/watch?v=VIDEO_ID"
```

### 오디오만 MP3로 추출

```bash
yt-dlp -x --audio-format mp3 -o "C:/tmp/%(title)s.%(ext)s" "https://www.youtube.com/watch?v=VIDEO_ID"
```
