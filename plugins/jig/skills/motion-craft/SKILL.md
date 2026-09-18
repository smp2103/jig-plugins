---
name: motion-craft
description: 모션그래픽 제작 문법 — 디자이너가 After Effects 에서 하는 일을 Jig 레이어로 하는 법. "모션 넣어줘", "애니메이션 고급스럽게", "AE 느낌으로", "로어서드", "콜아웃", "차트 애니메이션", "키네틱 타이포", "글자가 한 글자씩", "정지 이미지 같아", "움직임이 촌스러워", "리듬이 안 맞아" 요청과 설명형·광고·피드 릴스의 모든 모션 결정에 쓴다. 템플릿 75종 두 계열(list_mograph_templates)·트랜스폼 부모(set_layer_parent)·모션 경로·애니메이션 매트·텍스트 애니메이터·cubic-bezier 이징·스트로크 트림·루프·키프레임의 **언제 무엇을 얼마나** 를 정하고, 조립 뒤에는 lint_motion 으로 점검하고 preview_motion_strip 으로 리듬을 본 뒤에만 렌더한다. 수치는 실측(2026-09-14 원포핸드 벤치마크 v2/v3) 기준.
---

# 모션 크래프트 — AE 에서 하는 일을 Jig 로

> 모션이 "텍스트 애니메이션 수준"에 머무는 이유는 도구가 아니라 **문법**이 없어서다.
> 이 문서는 (1) 무엇을 움직일지 고르는 법 (2) 얼마나·언제 움직일지의 숫자 (3) 조립 순서 (4) 점검 루프를 고정한다.
> 그래픽은 항상 레이어다(생성 이미지는 배경 플레이트만). 생성 비용 0, 틀리면 값만 고친다.

## 0. 세 가지 원칙

1. **모든 게 계속 숨쉰다.** 등장한 뒤 완전히 멈추면 정지 이미지에 영상을 덧댄 것처럼 보인다. 배경은 `drift {end_scale 1.04}`, 패널·배지는 `loop breathe 0.03`, 숫자는 `loop pulse 0.06`. lint 의 `static-layer` 가 이걸 잡는다.
2. **읽는 순서대로 조립된다.** 한 장면의 요소는 한꺼번에 튀지 않는다 — 앵커 → 선 → 라벨 → 수치, 80~120ms 스태거. lint 의 `simultaneous-enter`.
3. **착지에만 스프링.** 오버슈트는 "도착"에 한 번. 이동 중엔 easeOut/easeInOut, 무거운 것은 `easeOutBack`, 기계는 `cubic-bezier(0.2,0.8,0.2,1)`. linear 는 등속 이동이 필요한 티커·회전에만.

## 1. 타이밍 바이블 (ms)

| 무엇 | 값 | 비고 |
|---|---|---|
| 등장 (pop/slideUp/blurIn) | 300~500, spring 380~450 | 900 넘으면 늘어짐 (`enter-too-slow`) |
| 퇴장 | 220~350 | 등장의 반대 방향. 없으면 컷 (`exit-missing`) |
| 형제 스태거 | 80~120 | 글자 30~60 · 단어 80~140 · 줄 160~260 |
| 선 그리기(draw) | 600~1600 | 앵커 점은 250~350 먼저 |
| 카운터 | 1000~1600 | 한 장면에 한 숫자 |
| loop pulse / breathe / float | 900~1200 / 2400~3200 / 2000~3000 | 진폭 0.06~0.1 / 0.03~0.05 / 8~14px |
| 모션 블러 | 4~8px | 450ms 이하의 빠른 슬라이드에만 |
| 장면 사이 숨 | 200 | 배경은 겹침 크로스페이드 250 |
| 마지막 프레임 | 정지 300 | 끝까지 움직이면 컷이 급해 보인다 |

**한 화면 밀도**: 0.5초 안에 5개 이상 등장하면 나뉘어야 할 장면이다(`too-many-enters`). 1.5초 넘게 아무 변화가 없으면 죽은 구간(`dead-air`).

## 2. 무엇으로 만들까 — 선택 표

먼저 `list_mograph_templates` 를 본다. **템플릿이 맞으면 손으로 조립하지 않는다** — 디자이너가 등장·퇴장·타이밍을 이미 넣어 두었다.

| 필요한 것 | 도구 | 핵심 파라미터 |
|---|---|---|
| 이름·직함 | `add_mograph_layer lowerThird` | text, subtext, style bar/line/box, accent |
| 사진 위 한 부분 가리키기 | `callout` | anchorX/Y(박스 안 0~1), direction, text |
| 핵심 숫자 하나 | `counterCard` | from, to, suffix, text(라벨) |
| 훅 문장 | `kineticTitle` | text(\n 줄바꿈), highlight(강조 단어), style pop/blur |
| 비교·추이·비율 | `barChart` / `lineChart` / `pieChart` / `progressRing` / `comparison` | items[{label,value,color}], max 0=자동 |
| 구간 강조 | `bracket` / `highlightSweep` | style horizontal/vertical · marker/underline/box |
| 판정 도장 | `badgeStamp` | text, tilt −12~12, style rect/round |
| A→B 관계 | `arrowConnector` | x1/y1/x2/y2, style curve/elbow, text/subtext |
| 섹션 타이틀 | `revealText` / `stepNumber` / `glitchTitle` | direction · value(번호) · seed |
| 절차·순서 | `listReveal` / `timeline` | items, style dot/number/check · value(강조 단계) |
| 인용 | `quoteCard` | text, subtext(출처) |
| 배경 결 | `dotGrid` (opacity 0.4, blend screen) | style ripple/twinkle, gap |
| 가격·할인 | `priceTag` | value(원가)→to(할인가), % 배지 자동 |
| 대화 재현 | `chatBubble` ×N (start_ms 늘려 쌓기) | direction left/right, subtext 이름 |
| 알림 연출 | `phoneNotification` | highlight 앱명, imageUrl 아이콘 |
| 영상 끝 CTA | `subscribeCta` / `swipeHint` | 마지막 3초 · 하단 중앙 |
| 전후 비교 | `beforeAfter` | imageUrl/imageUrl2 같은 구도 |
| 장소 | `mapPin` | anchorX/Y 를 지도·사진 위 지점에 |
| 평점·수치 요약 | `ratingStars` / `kpiRow` | value/max · items 2~4 |
| 뉴스·긴급 | `breakingNews` | 하단 1/3, 헤드라인 한 줄 |
| 시작 신호 | `countdown` | value×periodMs+800 만큼 길이 |
| 오프닝·엔딩 | `cinematicTitle` / `logoReveal` | 어두운 플레이트 위, imageUrl 로고 |
| 감탄사·스티커 | `list_lottie_assets` 기본 세트: confettiBurst·checkSuccess·heartPop·loadingDots·swipeUp·bellRing·sparkle·soundWave·labelBadge(문구 치환) | 박스 0.15~0.3, loop 는 로딩·웨이브·스와이프만 |
| 운동사슬·힘의 경로 | `add_path_layer` + draw + dots + pulse | points 0~1, taper grow, arrowEnd |
| 선을 따라 달리는 조각 | `path.keyframes trimStart/trimEnd` | 0→0.8 / 0.2→1 동시 |
| 여러 요소가 같이 흔들림·회전 | `set_layer_parent` (null = 보이지 않는 도형) | 부모에 keyframes/loop/path |
| 곡선 이동(공·화살표·카메라) | `set_layer_motion motion.path` | points, smooth, orient |
| 와이프·아이리스·도형 리빌 | `update_layer_box mask.anim` | rect w 0→100 / circle size 0→80 / path scale |
| 단어·글자별 등장/퇴장 | `text.charAnimation:'animator'` + `animator` | unit word, y 40, blur 10, order center, exit |
| 디자이너 AE 조각 그대로 | `list_lottie_assets` → `add_lottie_layer` | textReplacements, colorReplacements |

### 2-b. 시네마·쿠튀르 계열 (진지한 광고)

영화 예고편·화장품·럭셔리 패션·프리미엄 제품·다큐에서 실제로 쓰는 조각들. 위 표(예능 계열)와 **섞지 않는다**.

| 필요한 것 | 도구 | 핵심 파라미터 |
|---|---|---|
| 예고편 제목 | `filmTitleCard` | style track/split/settle, prefix(제작사), subtext(태그라인), letterSpacing 0.18 |
| 예고편 크레딧 덩어리 | `billingBlock` | text(\n), value(가로 압축 0.74) — 읽히라고 넣는 게 아니다 |
| 개봉·출시일 | `releaseDate` | style rules/stack, text(날짜), subtext |
| 시네마스코프 바 | `letterbox` | **gap 0.08~0.14** (세로 영상에서 value 비율 계산은 화면 3/4를 먹는다) |
| 평론·리뷰 인용 | `reviewQuote` | value(별 개수), text(한 줄), subtext(매체) |
| 관람등급 고지 | `certRating` | text(등급), subtext(사유 \n), accent(밴드색) |
| 장/챕터 구분 | `chapterSlate` | prefix, value(번호), text(제목), align left |
| 역할·이름 크레딧 | `creditRoll` | items[{label:이름, sub:역할}], style list/roll |
| 아카이브·비하인드 톤 | `timecodeBurn` | value(시작 초), periodMs(깜빡임) |
| 필름 카운트다운 | `academyLeader` | value(3), periodMs(1000) |
| 제품명 블록 | `productName` | prefix(브랜드), text(제품), subtext(용량), align |
| 성분 지시선 | `ingredientPin` | anchorX/Y(박스 안 0~1), direction, text/subtext |
| 임상 수치 | `clinicalStat` | from→to, suffix %, text(주장), subtext(각주 n=) — 각주를 빼면 지어낸 숫자로 보인다 |
| 호수·컬러 | `shadeSwatch` / `colorWay` | items[{label,color}], value(선택 번째) |
| 감촉·제형 한 줄 | `textureSweep` | text 한 줄만 |
| 전/후 | `splitCompare` | imageUrl(전)/imageUrl2(후), from→to(슬라이더) |
| 수상·선정 | `awardSeal` | prefix, text(수상명), subtext(연도) |
| 사용 순서 | `usageStep` | prefix STEP, value, text |
| 메종 마감 | `monogramReveal` / `foilWordmark` / `endSlate` | imageUrl(로고), letterSpacing, color2(금속 하이라이트) |
| 매거진 2단 | `editorialSplit` | text(\n 제목), subtext(\n 본문), value(분할비) |
| 룩 번호 | `lookIndex` | value/max, text |
| 흐르는 띠 | `marqueeTicker` | periodMs 8000~12000, direction |
| 절제된 가격 | `priceLuxe` | from→to, suffix, subtext(노트) — 배지·취소선 없음 |
| 도시·매장 | `boutiqueList` | items[{label:도시, sub:주소}] |
| 제품 컷 프레이밍 | `hairlineFrame` | gap(여백 0.03~0.06), text(아래 라벨), bg(라벨 뒤 바탕) |
| 핵심 스펙 3개 | `specGrid` | items[{label,value,suffix}] — 열마다 단위가 다르면 item.suffix |
| 기능 목록 | `featureTick` | items[{label}] — 체크 아이콘 대신 대시 |
| 부품 지시 | `techCallout` | anchorX/Y, direction up/down, 모노 라벨 |
| 치수 | `measureLine` | value + suffix mm, direction right/up |
| 기계식 숫자 | `rollingNumber` | to, prefix/suffix, text(라벨) |
| 로고+태그라인 | `logoLockup` | imageUrl, subtext, style row/column |
| 다큐 이름표 | `docLowerThird` | text/subtext, bg(스크림) |
| 장소·좌표 | `locationCard` | text(지명), subtext(좌표) |
| 선언문 | `statementText` | text(\n), style serif/cond/sans, highlight |
| 인터뷰 인용 | `interviewQuote` | text(\n 2~3줄), subtext(화자) |
| 컷 때리기 | `flashCut` | 레이어 200~400ms, 연속 사용 금지 |
| 필름 질감 | `filmGrain` | opacity 0.10~0.20, **어두운 화면이면 style screen** (overlay 는 검정을 보존해 안 보인다) |
| 막 전환 | `barsReveal` | value(바 5~9), staggerMs 60~90 |
| 유리·금속 광택 | `lightSweep` | 한 번만, thickness 12~22, opacity ≤ 0.5, delayMs 로 착지에 맞춤 |

### 계속 움직이는 것들 (v4, 2026-09)
위 75종은 등장 진행도 하나로만 굴러가 **착지하면 멈춘다**. 아래 7종은 경로·매트·지속 루프·글자
애니메이터·키프레임을 직접 쓰므로, 레이어가 살아 있는 동안 계속 움직인다. 손으로 path 레이어를
조립하던 자리를 대신한다.

| 필요한 것 | 도구 | 핵심 파라미터 |
|---|---|---|
| 동선·궤적·"여기서 저기로" | `pathTravel` | style arc/s/swoop/zigzag/loop, durationMs(주행), text(끝 라벨), seed |
| 계속 도는 인장·보증 배지 | `orbitSeal` | text(4자 이내), subtext, periodMs 4000~7000, max(톱니 수) |
| 한 문장 강조·장면 갈이 | `maskReveal` | style iris/wipe/box, iris 는 anchorX/Y 가 열리는 중심, gap(페더) |
| 움직이는 피사체를 오래 가리키기 | `pingBeacon` | anchorX/Y, radius(링), periodMs 1200~1800 — 박스를 넉넉히 |
| 순서가 읽히는 훅 문장 | `charCascade` | style(순서) forward/backward/center/edges/random, direction(단위) left=글자·down=단어·up=줄, suffix(등장) drop/rise/scatter/flip/focus/swipe |
| 곡선을 지정한 카운터 | `odometer` | from→to, style glide/snap/crawl/overshoot, prefix/suffix, durationMs < 레이어 길이 |
| 매달린 가격표·꼬리표 | `swingTag` | 화면 **위쪽 가장자리에 걸치게**, anchorX(못), tilt 3~6도 |

- 이 7종은 **레이어 길이 = 연출 길이**다. `durationMs`(주행·카운트)를 레이어보다 짧게 두어야
  착지한 뒤 쉬는 구간이 생긴다. 꽉 채우면 끝나자마자 잘린다.
- `pingBeacon`·`orbitSeal`·`swingTag` 는 등장 후에도 계속 도므로 `dead-air` 를 메우는 데 쓴다.
  단, 한 화면에 둘 이상 돌리지 않는다 — 시선이 갈린다.
- `callout` 과 `pingBeacon` 은 용도가 다르다: 한 번 가리키고 마는 건 `callout`, 오래 따라붙는 건 `pingBeacon`.

## 3. 조립 문법 — 장면 하나의 시간표

```
0        배경 플레이트 fade 250 (겹침 크로스페이드, 밑은 항상 검정 blank-base)
0~250    앵커 점(dots) / null 오브젝트 시작
250~1000 선 draw (taper grow) → 화살촉 자동
1100     라벨 pop 360 (spring) / 콜아웃 템플릿이면 이 순서가 내장
1400     핵심 수치 counterCard (1400 동안 올라감)
계속     pulse 가 선을 따라 돎, 배지 breathe
끝-300   퇴장: mask exitTo / animator exit / 템플릿 exitMs — 그리고 정지 300
```

여러 장면은 `stepNumber` 의 박스 위치를 고정해 번호가 같은 자리에 떨어지게 한다.
색은 한 영상에 한 체계, 그리고 **원색 하나**만: 기본 노랑 `#ffe500`, 경고·틀린 길 빨강 `#ff4438`, 정보·흐름 하늘 `#4da3ff`, 그 밖에 분홍 `#ff6fae`·라임 `#c6f23a`·주황 `#ff8a00` 중 하나. 면은 먹 `#161616` 아니면 종이 `#ffffff`. lint 는 세지 않지만 눈은 센다.

### 비주얼 문법 — 예능 자막·컷아웃 (v2, 2026-09)
- 템플릿은 **먹/종이/원색 + 두꺼운 테두리 + 오프셋 솔리드 그림자(hard shadow)** 로 그려진다. 반투명 카드·글로우·얇은 구분선·원형 체크·그라데이션은 없다. 대시보드처럼 보이면 잘못 조립한 것.
- 글자는 채움 + 먹 스트로크 + 하드 섀도(예능 자막). 강조어는 원색 채움에 살짝 기울어진다. 라벨은 찢어진 테이프·스티커·도장, 선은 손으로 그은 듯 흔들린다.
- 패널이 있는 템플릿(comparison·kpiRow·quoteCard·priceTag·mapPin·phoneNotification·callout)은 `bg` 에 종이색을 주고 글자색은 자동(밝은 면=먹, 어두운 면=종이). `bgOpacity` 는 1(불투명) 이 기본 — 0.7 같은 반투명을 주지 않는다.
- `counterCard` 는 기본 패널 없음(`bgOpacity 0`) — 큰 스트로크 숫자 + 테이프 라벨. 패널이 필요하면 `bg:'#ffffff', bgOpacity:1`.
- 사진 위 리스트·제목은 `listReveal`/`kineticTitle` 의 스트로크 글자면 어떤 배경에서도 읽힌다. 별도 스크림이 필요 없다.

### 비주얼 문법 — 시네마·쿠튀르 (v3, 2026-09)
영화 예고편·화장품·럭셔리·프리미엄 제품·다큐의 문법. 위 예능 계열과 **한 소재 안에서 섞지 않는다** — 섞이면 둘 다 싸구려가 된다.
- 면이 아니라 **헤어라인(1.3~2px)** 으로 구조를 잡는다. 두꺼운 테두리·하드 섀도·반투명 카드 전부 없다.
- 글자는 **대문자 + 넓은 자간(0.16~0.42em)** 이거나 **디스플레이 세리프**. 스트로크·다중그림자 없음.
- 색은 먹빛 검정 `#0b0b0c` · 본화이트 `#f2efe9` + 금 `#c8a15a`(또는 브랜드색) **하나**. 채도 높은 원색 금지.
- 모션은 느리고 길다(700~1500ms). 등장은 마스크 리빌 · 자간 수축 · 블러 해제 셋 중 하나. **팝·바운스·스프링 금지**(광고는 안 튄다).
- 한국어 세리프는 명조를 직접 지정한다(`fontFamily`). 이름에 '클래식'이 들어가도 고딕인 폰트가 있다 — 실제로 한 프레임 찍어 확인할 것.
- 어두운 플레이트가 기본이다. 빛은 도형 라디얼 그라데이션(빛 뭉치)과 기울인 선형 그라데이션(빛기둥, `blend_mode screen`)으로 만든다.
  **그라데이션 stop 의 `offset` 은 0~1 이 아니라 퍼센트(0~100)** — 0.3 을 주면 0.3% 로 나가 점 하나가 된다(2026-09-15 실측).

## 4. 고급 — AE 에서 당연한 것들

- **부모(null)**: 보이지 않는 도형(opacity 0)에 `keyframes rotation 0→8→0` 또는 `loop sway` 를 주고 라벨·선·아이콘을 `set_layer_parent` 로 붙인다. 부모 중심을 축으로 함께 움직인다(enter/exit·drift 는 상속 안 함).
- **모션 경로**: `points` 는 캔버스 0~1 의 **레이어 중심** 위치. `relative:true` 면 첫 점이 현재 자리. `orient` 는 화살표·공에만.
- **애니메이션 매트**: 배경 사진 위에 새 사진을 `mask rect {x:0,w:0} anim.to {w:100}` 로 밀어 넣으면 와이프 컷. `exitTo` 로 닫으면 장면 전환이 된다.
- **텍스트 애니메이터**: 훅 문장은 `unit:'word', y:40, blur:10, opacity:0, easing:'easeOutBack', staggerMs:110, exit:true`. 글자 단위는 짧은 단어(≤8자)에만.
- **이징 문자열**: AE 그래프 에디터 곡선은 `cubic-bezier(x1,y1,x2,y2)` 로 그대로. 자주 쓰는 것: 스냅 `0.2,0.8,0.2,1` · 묵직한 착지 `0.34,1.56,0.64,1` · 급감속 `0.16,1,0.3,1`.
- **스트로크 키프레임**: 임팩트 순간 `strokeWidth 6→16` + `glowBlur 0→14`, 흐름은 `dashOffset` 행진.
- **이펙트 스택**(2026-09-16): `update_layer_box.effects[]` — blur / glow(radius·color·intensity 1~3) / dropShadow(x·y·radius·color) / colorCorrect(brightness·contrast·saturate·hueRotate·grayscale·sepia·invert). 순서대로 filter 체인, 각각 `anim {to, durationMs, delayMs, easing, exitTo, exitMs}` — 글로우 0→24 점등, 블러 12→0 초점, 채도 0→1 흑백에서 색이 번짐. 글로우·그림자는 박스 밖으로 번진다(클리핑 없음). `clear_effects:true` 로 비운다. 예능 컷아웃엔 dropShadow `y 8, radius 0`(하드 섀도) 하나면 충분, 글로우 3겹은 여전히 금지.
- **손그림 선**(2026-09-16): path 에 `wobble 3~6`(점 사이를 잘게 나눠 흔든 손맛, `wobbleSeed` 결정적) + `outlineColor:'#161616', outlineWidth:4`(먹 이중선). 손밑줄 = 글자 폭 박스·높이 30·`points [{0,0.4},{1,0.65}]`·노랑 굵기 12·`wobble 5`·단어가 다 뜬 뒤 `draw 420`. 집중선 = 짧은 path 9~14개를 18~30ms 스태거로 `draw 180`, 수명 0.5초. `clear_outline:true` 로 이중선 제거.
- **애니메이터 jitter**: `animator.jitter 1.5~3` = 단위마다 다른 정지 기울기(예능 단어 팝의 삐뚤빼뚤, 등장 후에도 남는다). 예능 단어 팝 `{unit:'word', staggerMs:90, durationMs:420, scale:0.2, opacity:0, easing:'spring', jitter:2.5}`.
- **카운터 함정**: `textProps.counter` 가 켜진 레이어는 문구가 안 보인다 — 에디터에서 사람이 문구를 "198,000원" 으로 고치면 끝값·단위가 따라가게 해 뒀고, MCP 에선 `counter.to` 를 직접 고친다.
- **에디터와 같은 자료구조**: 위 전부가 에디터의 키프레임 섹션(⏱ 스톱워치·∿ 값 그래프·타임라인 ◆ 레인)·이펙트 스택 패널에 그대로 보인다. 사람이 그래프에서 핸들을 끌면 `cubic-bezier(...)` 로 굳는다 — 내가 넣은 값을 디자이너가 이어서 고치는 구조라, 정성껏 넣을수록 되돌아오는 정답이 는다.

### 4-b. 그래픽 템플릿 — 편집 가능한 레이어 묶음 (AE 프리컴프, 2026-09-16)

| | 파라미터 템플릿(§2, MOGRT식) | **그래픽 템플릿(레이어 묶음)** |
|---|---|---|
| 도구 | `list_mograph_templates` → `add_mograph_layer` | `list_layout_templates`(kind='graphic') → `apply_layout_template(mode:'insert', at_ms)` → `inserted_layer_ids` |
| 정체 | 코드 조각 82종. 한 레이어, 파라미터만 바뀜 | **실제 텍스트·도형·경로 레이어 + 키프레임·이펙트·마스크·애니메이터가 풀려 들어옴** → 아무 레이어나 `update_text_layer`/`update_layer_box`/`update_path_layer` 로 고침 |
| 언제 | 빠르게 정확한 조각(차트·링·타임라인·글리치) | 사람이 이어서 손볼 소재, 브랜드에 맞춰 바꿀 소재 |
| 저장 | 불가 | `save_graphic_template(draft_id, name, layer_ids)` — 잘 만든 묶음을 재사용 템플릿으로. 에디터 ⚡ 첫 탭에 뜬다 |

기본 제공 그래픽 템플릿 47종 — `list_layout_templates` 의 name 으로 찾는다.
- 예능 컷아웃(먹/종이/노랑·테이프·슬램): 로어서드 · 이름/직함 / 카운터 카드 · 핵심 수치 / 키네틱 타이틀 · 훅 라인 / 배지 스탬프 / 콜아웃 · 부위 지목 / 가격표 · 정가/할인가 / 인용문 · 후기/한마디 / 진행 바 · 퍼센트 / 주의 문구 · 경고 띠 / 말풍선 · 리액션 / 체크리스트 · 3항목 / 스텝 넘버 · 01 / 비포 / 애프터 라벨 / 할인 스타버스트 · 50% / CTA 버튼 · 지금 예약 / 별점 · 4.9 리뷰 / SNS 아이디 · @핸들 / 위치 핀 · 주소 / 카운트다운 · 10초 / 랭킹 · 1위 / 질문 카드 · Q. / 예능 자막 · 2줄 키워드 / 뉴스 띠 · 속보 / 리액션 · ㅋㅋㅋㅋ / 진행 단계 · 3단계 / D-day · D-3 / 비교 · VS 두 패널 / 화살표 지목 · 여기! / 손 동그라미 · 강조 / 형광펜 강조 · 키워드 / 해시태그 · 3개 / 채팅 대화 · 2줄 / 박스 자막 · 먹 박스 / REC 타임코드 · 촬영 중 / 폰 알림 · 카드 / TOP 3 · 순위 리스트 / 전화 안내 · 예약 문의 / 더보기 ↓ · 스크롤 유도 / 페이지 · 1/5 / 구독 · 좋아요 버튼
- 시네마·쿠튀르(먹빛/본화이트/금 헤어라인, 느린 등장): 챕터 슬레이트 / 로케이션 자막 · 시네마 / 시네마 인용 · 한 문장 / 헤어라인 콜아웃 · 쿠튀르 / 에디션 넘버 · No. 042 / 엔딩 크레딧 · 브랜드 / 인터뷰 자막 · 시네마
- 쓰는 법: `apply_layout_template(mode:'insert', at_ms:대사 시각)` → `inserted_layer_ids` 의 텍스트를 `update_text_layer` 로 바꾼다. 카운터(카운터 카드·카운트다운·REC·진행 바)는 `counter.to` 를, 밝은 컷 위 시네마 조각은 함께 들어온 '어두운 플레이트' 불투명도를 고친다. 널 `⊕ … 축` 에 슬램이 있다.

## 5. 점검 루프 — 예쁜 것과 맞는 것은 다르다

1. `lint_motion` → `error`·`warn` 을 전부 고친다(각 항목에 fix 가 적혀 있다). 점수 85 이상.
2. `preview_motion_strip from_ms/to_ms` 로 장면 하나(2~5초)를 8칸으로 본다 — 읽는 순서대로 조립되는가, 멈춘 칸이 있는가, 퇴장이 있는가.
3. 의심되는 칸은 `preview_edit_frame at_ms` 한 장으로 크게.
4. 렌더는 **사용자가 지시할 때만.** `create_share_link` 로 전달.

## 6. 하지 말 것

- 생성 모델에 글자·선·화살표를 그리게 하지 않는다(깨지고, 틀리면 재생성뿐).
- 도형+텍스트 2장으로 pill 을 만들지 않는다 — frame(autoWidth)+text 또는 템플릿.
- 글로우 3겹, 진폭 큰 루프, 900ms 넘는 등장, linear 슬라이드, 한 화면 5개 동시 등장.
- Tailwind 팔레트(`#22d3ee` 시안·`#0b1220` 슬레이트·`#facc15`)와 반투명 둥근 카드 — shadcn 대시보드처럼 보이는 즉시 반려된다.
- 템플릿 길이를 minDurationMs 보다 짧게 두지 않는다(등장이 잘린다).
