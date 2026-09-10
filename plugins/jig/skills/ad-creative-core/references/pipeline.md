# 엔드투엔드 앞단 — 기획→생성→조립

"소재를 처음부터 만들어줘" 요청이면 이 문서를 **디자인 워크플로(SKILL.md) 앞에** 붙인다.
이미 클립이 있는 소재에 레이어만 얹는 작업이면 이 문서는 건너뛴다.

## 1. 기획

- 15초 = 4씬(4+4+3+4초), 8초 숏츠 = 2씬(4+4초)이 검증된 골격.
- **씬별 카피는 기획 단계에서 확정**한다 — 생성 후에 카피를 끼워 맞추면 존이 안 맞는다.
- 검증된 셀프-리빌 구조: 페이크 광고 N씬 + 리빌 1씬("X didn't make this ad. An AI agent did.").

## 2. 이미지 생성 (`generate_image`, 9:16, count 1)

- **S1 히어로를 먼저** 뽑고 눈QA → 이후 씬은 `reference_image_urls: [S1_url]` 로 **체이닝**
  (제품/피사체 일관성 유지).
- 실브랜드 로고 방지: 대상을 무마킹으로 묘사 ("unmarked stainless steel tumbler").
- **제품 라벨 fine-print 는 반드시 환각된다** (실측: PDRN→"PORN-based").
  해결 = 라벨 허용 텍스트를 exact 나열 + "absolutely NO fine-print paragraph" 지시 →
  클린 라벨 성공본을 후속 씬 레퍼런스로 재체이닝.
- **라벨 오염 2차 경로**: ① 과거 자산 재사용 시 클린 처리 이전 세대가 섞여 있을 수 있다 —
  재사용 전 전체 크롭 검사. ② **i2v 가 라벨을 재렌더하며 훼손**한다 — 라벨 클로즈업은
  nearly-static 팩샷 푸시인으로만, 인물 발화 씬에선 라벨 노출을 피한다.
- 추상/미니멀 프롬프트는 브랜드 목업 환각이 잦다 — "abstract minimal" 대신 구체적 사진
  장르 문법으로 ("photorealistic astrophotography: dense star field...").
- **타깃 시장 인물은 명시한다** — 영미권 소재면 "young Caucasian woman" 처럼 쓰지 않으면
  아시안으로 나온다. 매 이미지 눈QA, 실패 시 재생성.

## 3. 영상 생성 (`generate_video`)

- i2v = `reference_image_url` + `source_image_id`. provider `kling` 이 기본 우회처.
- **모션 프롬프트는 절제가 고급**: "nearly static", "extremely slow push-in", "subtle".
  과한 모션 지시는 왜곡을 부른다. 움직임은 이후 켄번스(`set_clip_framing` end_scale)로 얹는다.
- 대사 없으면 `duration_seconds` 지정(생략 시 kling 5초 고정), 대사 있으면 서버가
  대사 길이로 자동 산정한다(한국어 4.0자/초, 영어는 1/3 가중 — 서버가 언어를 감지한다).
- 완료 폴링 `check_video_status` (분 단위 — 백그라운드 대기).
- **merge 전 클립 눈QA**: `preview_video_frame`(at_ms=클립 중간) — 얼굴 왜곡·라벨 깨짐·아티팩트.

## 4. 조립

- 클립이 모이면 `create_edit_draft_from_clips`(재생 순서대로 video id 목록)로 편집 draft 를 만든다.
- `set_clip_trim` 으로 씬 길이 확정 후 **`compact_clip_timeline` 필수 1회** — 트림은 뒤
  클립을 안 당겨서 start_at 갭이 남고, 레이어/자막/오디오 절대시각은 컴팩트된 타임라인
  기준이어야 한다. 응답의 timeline 이 씬 경계표다.

## 5. 나레이션/오디오

- **`list_tts_voices` 로 보이스를 먼저 고른다** (기본 = 즐겨찾기 큐레이션. typecast 한국어
  보이스 id 는 `tc_...` 불투명이라 이 도구 없이는 못 쓴다. **나레이션 언어와 보이스
  로케일을 맞출 것** — vertex id 에 로케일이 박혀 있다: en-US-…/ko-KR-…).
- `generate_tts`(text, provider, voice_id, instructions=톤 지시, speed) → 응답의
  {id, public_url, duration_ms} 를 `add_audio_layer`(start_ms=씬 시작+α, volume, fades)로 배치.
- **씬 트림 길이는 나레이션 duration_ms + 400ms 이상** — 아니면 발화가 잘린다.
- BGM 은 사람이 에디터에서 얹거나 무음 배포(피드 자동재생은 무음 기본).

## 6. 멀티포맷 (9:16 마스터 원칙)

- **9:16 마스터 + 1:1 은 커버 크롭, 16:9 는 네이티브 재생성** (기존 씬 이미지를 레퍼런스
  체이닝하면 인물 일관성 유지). 9:16→16:9 크롭은 3.3배 업스케일이라 실용 한계 밖.
- 커버 크롭은 `set_clip_framing`: 9:16→1:1 scale 1.85, 16:9→1:1 1.8.
  `offset_y = (0.5 − 원하는 소스 중심 f) × contain높이` — 씬별 얼굴 잘림 방지가 핵심.
  offset 을 쓰려면 그만큼 scale 여유 필수(딱 맞는 scale 에서 밀면 반대편 검은 띠).
- `add_edit_format` 은 스켈레톤만 만든다 — **자막·오디오·레이어를 포맷별로 전부 재구축**
  (안 하면 새 포맷 렌더가 무음·무자막).
- **16:9 는 9:16 축소판이 아니다**: 대형 타이포(헤드라인 80~110px@h1080), 좌/우 존 분리
  (피사체 반대편이 텍스트존), 밀도 요소 5+/씬. 폰트가 작으면 반려된다.
  포맷별 ratio 재계산은 measurements.md §4.

## 7. 자막 함정 (사람 에디터와의 동거)

- **사람이 에디터로 만든 draft 는 자막 전 라인에 per-entry style 오버라이드가 박혀 있는
  경우가 많다** — 이때 `set_subtitle_style`(포맷 스타일)은 렌더에 안 먹는다. 개편 전
  `get_edit_draft(summary=false)` 로 entry.style 유무 확인, 필요하면
  `update_subtitle_entry(clear_style=true)` 로 전 라인 청산부터.
- 자막이 좌/상단으로 밀려 보이면 `reset_subtitle_frames` — 에디터가 심은 오염된 editor
  프레임을 스타일 기본 배치로 복귀시킨다. 렌더 직전 1회가 안전.
- `update_subtitle_entry` 타이밍 변경은 재정렬을 유발한다 — **응답 entry_idx 로 새 인덱스 확인**.
- runs(키워드 강조)와 charAnimation 은 상호배타. 텍스트 교체는 기존 runs 를 제거한다.

## 8. 프리뷰 운영 팁

- `preview_edit_frame` 은 `at_ms_list` 일괄이 빠르지만 **한 번에 2~4장으로 끊는다**
  (payload 한도로 이미지가 빠질 수 있고, 빠지면 응답이 알려준다). 크게 봐야 하면
  그 시점 하나만 단독 렌더(자동 최대 크기).
- 샘플링은 **씬경계+200ms(스태거 완료 직후) + 씬중앙** 두 세트.
- 프리뷰를 사람에게 보여줄 땐 URL 을 마크다운 링크로도 정리 — 도구 카드 속 이미지는 작다.
