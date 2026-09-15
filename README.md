# NDS Tracklist Parser

복사한 음원 트랙리스트를 브라우저에서 바로 파싱하여, **Stereo ISRC**와 **Dolby ISRC**가 분리된 표 형태의 데이터로 정리하는 로컬 전용 HTML 도구입니다.

음원 유통 플랫폼, 카탈로그 관리 도구, 트랙리스트 화면 등에서 `# / Title / Artist / ISRC` 형태로 복사한 텍스트를 붙여넣으면 트랙 번호, 릴리즈명, 재생시간, 트랙명, 아티스트, Stereo ISRC, Dolby ISRC를 자동으로 추출합니다. 결과는 화면에서 수정할 수 있으며, Google Sheets·Excel 등에 바로 붙여넣기 좋은 TSV 또는 CSV 파일로 내보낼 수 있습니다.

## Description

**NDS Tracklist Parser** is a privacy-first, single-file HTML utility for parsing copied music tracklists into structured metadata tables. It automatically identifies track numbers, release titles, durations, track titles, artists, Stereo ISRCs, and Dolby Atmos ISRCs. All processing happens locally in the browser, with no login, server, API, or external data transmission required.

The parser applies a practical delivery rule for ISRC mapping:

- One detected ISRC is always assigned as the **Stereo ISRC**.
- Two detected ISRCs are assigned in order: first as the **Stereo ISRC**, second as the **Dolby ISRC**.
- A Dolby ISRC is never created on its own without a Stereo ISRC.
- Three or more detected ISRCs in one track block are flagged for review.

## 주요 기능

- 단일 HTML 파일로 실행되는 로컬 브라우저 도구
- 서버·로그인·외부 API 없이 동작
- 입력 데이터의 외부 업로드 또는 전송 없음
- 트랙리스트 복사 텍스트 붙여넣기 및 자동 파싱
- `#`, `Title`, `Artist`, `ISRC` 등 헤더가 섞인 입력 처리
- 진행률 텍스트(`0%`, `32%`, `99%`, `100%` 등) 자동 제외
- 트랙 번호, 릴리즈명, Duration, Track Title, Artist 추출
- 단일 ISRC를 Stereo ISRC로 자동 배정
- 2개의 ISRC를 Stereo ISRC / Dolby ISRC로 순서대로 분리
- 결과 테이블에서 직접 편집 및 즉시 재검증
- ISRC 형식, 누락, 중복, Stereo/Dolby 동일값, ISRC 3개 이상 탐지
- 재생시간 `MM:SS` ↔ `HH:MM:SS` 양방향 변환
- TSV 클립보드 복사
- UTF-8 BOM 포함 CSV 다운로드
- TSV 파일 다운로드
- 행 추가, 행 삭제, 트랙 번호 재정렬
- 라이트/다크 모드
- 마지막 입력과 편집 결과를 브라우저 `localStorage`에 저장

## 사용 방법

1. `nds-tracklist-parser.html` 파일을 다운로드합니다.
2. 파일을 더블클릭하거나 Chrome, Edge, Safari, Firefox에서 엽니다.
3. 원본 시스템에서 `#`부터 마지막 ISRC까지 복사합니다.
4. 좌측 입력창에 원문을 붙여넣습니다.
5. 자동 파싱이 켜져 있으면 잠시 후 결과가 생성됩니다. 자동 파싱이 꺼져 있다면 `파싱하기`를 누릅니다.
6. 우측 결과 표에서 자동 추출된 내용을 확인하고 필요한 셀을 수정합니다.
7. 필요하면 `시간을 HH:MM:SS로 변환` 또는 `시간을 MM:SS로 되돌리기`를 눌러 Duration 형식을 변경합니다.
8. `TSV 복사`를 눌러 Google Sheets 또는 Excel에 붙여넣거나, CSV/TSV 파일을 다운로드합니다.

## 지원 입력 예시

### Stereo ISRC만 있는 트랙

```text
#
Title
Artist
ISRC
1
Vol. 1
02:32
Just Let It Go
IAN KIM
USB8U2603047
```

결과:

| Track No. | Release Title | Duration | Track Title | Artist | Stereo ISRC | Dolby ISRC |
|---:|---|---|---|---|---|---|
| 1 | Vol. 1 | 02:32 | Just Let It Go | IAN KIM | USB8U2603047 | |

### Stereo + Dolby ISRC가 있는 트랙

```text
#
Title
Artist
ISRC
1
Vol. 1
03:58
Histoire du Tango: I. Bordel 1900
Tommaso Benciolini and Lorenzo Bernardi
USB8U2603014
USB8U2603030
```

결과:

| Track No. | Release Title | Duration | Track Title | Artist | Stereo ISRC | Dolby ISRC |
|---:|---|---|---|---|---|---|
| 1 | Vol. 1 | 03:58 | Histoire du Tango: I. Bordel 1900 | Tommaso Benciolini and Lorenzo Bernardi | USB8U2603014 | USB8U2603030 |

## ISRC 매핑 규칙

이 도구는 복사된 데이터에서 ISRC의 의미를 개별 코드 자체로 판단하지 않고, **한 트랙 블록 안에서 탐지된 ISRC의 개수와 순서**를 기준으로 구분합니다.

| 트랙 블록 내 ISRC 수 | Stereo ISRC | Dolby ISRC | 처리 결과 |
|---:|---|---|---|
| 0개 | 빈 값 | 빈 값 | 오류 |
| 1개 | 첫 번째 ISRC | 빈 값 | Stereo 트랙으로 처리 |
| 2개 | 첫 번째 ISRC | 두 번째 ISRC | Stereo + Dolby 트랙으로 처리 |
| 3개 이상 | 첫 번째 ISRC | 두 번째 ISRC | 오류 표시 및 추가 코드 검토 필요 |

### 핵심 업무 규칙

- ISRC가 하나만 있으면 무조건 **Stereo ISRC**입니다.
- Dolby ISRC는 Stereo ISRC가 있을 때에만 존재할 수 있습니다.
- ISRC가 두 개이면 첫 번째는 **Stereo ISRC**, 두 번째는 **Dolby ISRC**입니다.
- 단일 ISRC를 시스템이 자동으로 Dolby ISRC로 판단하지 않습니다.
- Stereo ISRC와 Dolby ISRC가 동일하면 오류로 표시합니다.

## 출력 컬럼

기본 TSV/CSV 출력은 아래 순서로 구성됩니다.

| 순서 | 컬럼명 | 설명 |
|---:|---|---|
| 1 | Track No. | 릴리즈 내 트랙 번호 |
| 2 | Release Title | 앨범 또는 릴리즈명 |
| 3 | Duration | 재생시간 (`MM:SS` 또는 `HH:MM:SS`) |
| 4 | Track Title | 트랙명 또는 작품명·악장명 |
| 5 | Artist | 표시 아티스트 |
| 6 | Stereo ISRC | 스테레오 마스터 ISRC |
| 7 | Dolby ISRC | Dolby Atmos/Immersive Audio 마스터 ISRC |

`Status`와 `Note`는 화면에서 검증을 확인하기 위한 컬럼이며, 기본 TSV/CSV 내보내기에는 포함하지 않습니다.

## 시간 변환

파싱 직후 Duration은 기본적으로 원문에서 인식한 `MM:SS` 형식으로 표시됩니다.

| 변환 전 | HH:MM:SS 변환 후 | 되돌리기 후 |
|---|---|---|
| `03:58` | `00:03:58` | `03:58` |
| `07:19` | `00:07:19` | `07:19` |
| `01:02:03` | `01:02:03` | `62:03` |

시간 변환 버튼은 표에 표시되는 값 자체를 변경합니다. 따라서 변환 후 TSV 복사, CSV 다운로드, TSV 다운로드를 하면 선택한 시간 형식이 그대로 반영됩니다.

## 검증 항목

도구는 파싱 및 편집 후 다음 항목을 검사합니다.

| 검증 항목 | 결과 |
|---|---|
| 트랙 번호 없음 | Error |
| 트랙 번호 중복 | Error |
| 트랙 번호가 1부터 연속되지 않음 | Warning |
| Track Title 없음 | Error |
| Stereo ISRC 없음 | Error |
| Stereo ISRC 형식 오류 | Error |
| Dolby ISRC 형식 오류 | Error |
| Dolby ISRC는 있으나 Stereo ISRC 없음 | Error |
| Stereo ISRC와 Dolby ISRC가 동일 | Error |
| 다른 행에 같은 ISRC가 중복 사용됨 | Error |
| 한 트랙에 ISRC가 3개 이상 존재 | Error |
| Duration이 없거나 형식이 맞지 않음 | Warning |
| Artist가 비어 있음 | Warning |

ISRC 값은 입력 중 하이픈, 공백, 소문자가 섞여도 정규화합니다.

```text
US-B8U-26-03014
us b8u 26 03014
usb8u2603014
```

모두 아래 값으로 정규화됩니다.

```text
USB8U2603014
```

## TSV 및 CSV 내보내기

### TSV 복사

`TSV 복사`는 탭으로 구분된 데이터를 클립보드에 복사합니다. Google Sheets, Microsoft Excel, Apple Numbers, Airtable 등의 첫 셀에 붙여넣으면 각 값이 열로 분리됩니다.

기본적으로 헤더가 포함되며, 상단의 `헤더 포함` 옵션을 해제하면 데이터 행만 복사·다운로드할 수 있습니다.

### CSV 다운로드

`CSV 다운로드`는 UTF-8 BOM이 포함된 `.csv` 파일을 생성합니다. 한글, 프랑스어 악센트 문자, 따옴표, 쉼표가 포함된 트랙명도 Excel에서 최대한 안정적으로 열 수 있도록 처리합니다.

### TSV 다운로드

`TSV 다운로드`는 탭 구분 텍스트 파일을 생성합니다. Google Sheets나 스프레드시트 기반의 데이터 정리 업무에 적합합니다.

## 개인정보 및 로컬 저장

NDS Tracklist Parser는 외부 서버를 사용하지 않는 단일 파일 도구입니다.

- 붙여넣은 트랙리스트와 ISRC는 외부로 전송되지 않습니다.
- 로그인, 계정 생성, API 키, 네트워크 연결이 필요하지 않습니다.
- 파싱, 검증, 시간 변환, TSV 복사, CSV/TSV 생성은 모두 브라우저 안에서 처리됩니다.
- 마지막 입력 텍스트, 파싱 결과, 테마, 설정은 편의를 위해 브라우저의 `localStorage`에 저장될 수 있습니다.
- 같은 브라우저에서 해당 HTML 파일의 저장 정보를 삭제하거나 브라우저 사이트 데이터를 제거하면 로컬 저장 데이터도 삭제됩니다.

## 알려진 한계

이 도구는 원본 페이지의 HTML 구조가 아닌, 사용자가 복사한 **비정형 텍스트**를 분석합니다. 따라서 아래 상황에서는 Track Title과 Artist의 경계가 추정값일 수 있습니다.

- 제목과 아티스트가 모두 여러 줄로 분리된 경우
- 반복되는 아티스트명 또는 릴리즈명 패턴이 없는 경우
- 원문이 트랙별로 일정한 필드 순서를 유지하지 않는 경우
- 작품명, 부제, 연주자, 크레딧, 카탈로그 정보가 한 블록에 혼재한 경우

이 경우에도 ISRC의 개수와 순서에 따른 Stereo/Dolby 분리 규칙은 유지되며, 결과 테이블에서 직접 수정한 뒤 TSV/CSV로 내보낼 수 있습니다.

## 기술 구성

- HTML5
- CSS3
- Vanilla JavaScript
- Clipboard API
- Blob API
- localStorage

외부 프레임워크, CDN, 백엔드, 데이터베이스를 사용하지 않습니다. 따라서 파일 하나만 배포하면 사용할 수 있고, 네트워크가 차단된 환경에서도 핵심 기능을 이용할 수 있습니다.

## 파일 구조

배포 파일은 하나입니다.

```text
nds-tracklist-parser.html
```

개발 또는 수정 시에도 HTML 내부의 CSS와 JavaScript만 편집하면 됩니다.

## 향후 개선 아이디어

- 실제 업무 시스템별 복사 포맷 프리셋
- 복수 릴리즈를 한 번에 구분·파싱하는 배치 모드
- UPC/EAN, Catalog No., Label, P-line, C-line, Composer, Publisher 등 메타데이터 열 확장
- Stereo, Dolby Atmos, Hi-Res, Instrumental, Clean, Edit 등 오디오 버전별 데이터 모델 확장
- 원문 줄과 결과 행의 양방향 하이라이트
- 드래그 앤 드롭 순서 변경 및 다중 행 편집
- JSON 저장·불러오기
- Excel `.xlsx` 내보내기
- 기존 카탈로그 데이터와 ISRC 중복 대조
- 다국어 UI 지원

## License

Internal utility / private use.

프로젝트의 배포·공개·상업 이용 조건은 운영 주체의 정책에 따라 별도로 정의합니다.
