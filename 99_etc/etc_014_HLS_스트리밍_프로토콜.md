# 🚀 HLS 스트리밍 프로토콜

## 🎥 스트리밍 프로토콜 이해

현대 웹/모바일에서 실시간 영상, VOD, 라이브 방송을 송출할 때 가장 널리 쓰는 두 가지 프로토콜:

### 🔵 HLS vs DASH — 두 개의 대표 스트리밍 표준

| 프로토콜                      | 만든 곳    | 비디오 컨테이너     | 플레이리스트 형식 | 코덱 지원                 | 주요 특징                         |
| ----------------------------- | ---------- | ------------------- | ----------------- | ------------------------- | --------------------------------- |
| **HLS (HTTP Live Streaming)** | Apple      | MPEG-TS (.ts), fMP4 | .m3u8             | H.264 / AAC / HEVC        | 안정적, 브라우저 호환성 최고      |
| **MPEG-DASH**                 | MPEG (OPF) | fMP4                | .mpd              | H.264 / H.265 / VP9 / AV1 | 표준 기반, 유연함, 고급 코덱 호환 |

둘 다 동작 방식은 거의 동일하며,

- “긴 영상 파일을 작은 조각(segment)으로 나눠서 HTTP로 전달”하는 방식이다.

## 🔥 1. 스트리밍이란?

일반적인 다운로드와 다르게 전체 파일을 다운받지 않고, 조각난 파일(segments)을 연속적으로 받아
재생하는 기술.

특징:

- 재생과 다운로드를 동시에 수행
- 네트워크 품질에 따라 화질 자동 조절(Adaptive Bitrate Streaming)
- 라이브와 VOD 모두 가능
- HTTP 기반 → CDN 친화적

## 🟦 2. HLS(HTTP Live Streaming) 구조

HLS는 Apple이 만든 기술이며 가장 널리 쓰인다
— iOS, Safari 기본 지원 + 모든 브라우저에서 hls.js로 재생 가능.

```shell
playlist.m3u8
├── segment_000001.ts
├── segment_000002.ts
└── segment_000003.ts
```

핵심 구성 요소

| 요소                         | 설명                                     |
| ---------------------------- | ---------------------------------------- |
| **Playlist (.m3u8)**         | 재생할 segment 목록을 저장한 텍스트 파일 |
| **Segment (.ts, 혹은 .m4s)** | 실제 비디오 조각                         |
| **Master Playlist**          | 서로 다른 화질의 m3u8을 묶어둔 playlist  |

## 🟧 3. MPEG-DASH 구조

DASH는 국제 표준(ISO/IEC 23009-1)로 정의된 스트리밍 방식.

```shell
manifest.mpd
├── segment_1.m4s
├── segment_2.m4s
└── segment_3.m4s
```

- Special — MPD(Manifest)
- DASH에서 .mpd는 JSON+XML 같은 구조로, 매우 상세한 타이밍/코덱 정보 제공.

## 🔥 4. HLS vs DASH — 차이점 정리

✔ 동작 원리는 거의 동일

둘 다:

- segment 파일
- playlist(manifest)
- adaptive bitrate
