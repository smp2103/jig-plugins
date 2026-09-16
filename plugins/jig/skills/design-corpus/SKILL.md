---
name: design-corpus
description: 디자인 코퍼스(/corpus, design_assets) 헌법 + 운용법 — 코퍼스를 "같은 부품의 스킨 변주"가 아니라 디자인 어휘·구조적 다양성 저장소로 키운다. 원자(primitive)/조합(component)/배치 문법(composition)/처리 레시피(treatment)/디자인 언어(style_family) 5층, 정체성 3축(primitive_type·semantic_role·structural_pattern)+treatments, 변주 규칙(variant_of), 신규성 게이트, 갭 분석→생성 루프, 코퍼스에서 새 디자인을 합성하는 절차. "코퍼스 늘려줘", "새 디자인 뽑아줘", "디자인 다양하게", "재료 저장해", "빈칸 채워", "이 페이지 원자로 쪼개" 요청 시, 그리고 save_design_asset / search_design_assets / get_corpus_coverage 를 부르기 전에 로드한다.
---

# 디자인 코퍼스 — 헌법과 운용

코퍼스는 **템플릿 창고가 아니라 디자인 어휘(vocabulary)** 다. 목표는 개수가 아니라
**구조적으로 서로 다른 디자인 아이디어의 수**, 그리고 그 아이디어들을 조립해 **아무도 본 적 없는 페이지**를
만드는 것이다. 2026-09-06 실측: 265개 중 215개가 같은 부품의 색·폰트·이펙트 변주였고 구조적 어휘는 50개였다.
그 뒤로는 서버가 색만 다른 "새 재료"를 거부한다.

## 0. 절대 규칙 — 텍스트가 도형 위/안에 놓이면 frame (2026-09-06 사용자 재지적)

칩·캡슐(pill)·라벨·배지·CTA·번호 원·TIP 박스·카드 헤더처럼 **텍스트 뒤에 도형이 있는 락업은 shape + text 를 absolute 로 겹치지 않는다.**
`add_frame_layer`(배경/라운드/테두리/padding/layout 은 프레임이 가짐, 칩은 `auto_width:true`) → 응답엔 id 가 없으니 draft 를 덤프해 프레임 id 를 읽고 → `add_text_layer(parent_id)` 로 글자만 넣는다.
이유: 에디터와 렌더의 글자 폭이 달라 absolute 겹침은 잘리거나 두 줄로 접히는데, 얼핏 보면 멀쩡해 보여서 깨진 채 나간다. 배치 응답의 `warnings[kind=shape_text_overlap]` 는 경고가 아니라 **실패**다 — 프레임으로 다시 짠다.
예외는 사진/판 위에 자유 배치되는 헤드라인·본문뿐(그건 배경 도형이 "락업" 이 아니라 바탕이다). 코퍼스 원자도 chip/pill/badge/cta 계열은 프레임 구조로 저장한다.

## 1. 헌법 (서버 게이트가 강제한다)

1. **원자(primitive) = 디자인 아이디어 하나.** 헤드라인 3줄이 각각 다른 처리(이중 아웃라인 / 단일 획 / 흰 후광)면
   그건 **원자 3개 + 그 배열(component) 1개**다. 하나의 덩어리로 저장하지 않는다.
2. **색·폰트·획 두께·그림자·팔레트·질감만 다르면 새 재료가 아니다.** `variant_of=<기준 id>` 로 변주 저장.
   변주는 어휘에 기여하지 않고, 목록에서 기준 재료 아래로 접힌다.
3. **새 재료의 정체성 = primitive_type / semantic_role / structural_pattern + treatments 집합.**
   같은 정체성이 이미 있으면 서버가 400 을 돌려준다 — `novelty{semantic,structural,geometry,composition,visual}`
   가중합 ≥ 0.4 를 넘기고 description 에 무엇이 구조적으로 새로운지 써야 통과한다.
4. **새 재료는 다음 중 하나가 의미 있게 달라야 한다**: semantic role · 정보 위계 · 기하 · 레이아웃 동작 · 배치 ·
   요소 간 관계(글자↔사진, 글자↔도형).
5. **만들기 전에 커버리지를 본다.** `get_corpus_coverage` → 비어 있는 개념(gaps)·비어 있는 배치 패턴·비어 있는
   처리를 먼저 채운다. 있는 걸 색만 바꿔 넣는 건 금지.
6. **composition 은 페이지 복사본이 아니라 공간 배치 문법(슬롯 스켈레톤)이다.** 스타일은 원자에서 오고 배치는
   스켈레톤에서 온다. 그래서 같은 원자 5개로도 배치를 바꾸면 전혀 다른 디자인이 나온다.
7. **마감 게이트(2026-09-06 "AI slop" 반려 후 신설).** 단색 채움 + 단색 글자로만 된 원자는 저장이 거부된다. 원자/조합은 깊이·질감 장치
   **2개 이상**(surface 는 1개): textFx 획 / 이중 획 / 그림자 스택 / 그라데이션 채움 / roughen, 텍스트 outline / textShadows / fontGradient /
   edgeRoughen / run 크기 대비, 도형 선형·라디얼 그라데이션 / 그림자 / 글래스 / **svg 패스**, blend mode, perspective, mask, image.
   헤어라인·스크림처럼 평면이 정답인 것만 `labels.flat_ok="<사유>"` 로 통과. 저장 응답 `labels.craft_devices` 로 무엇이 인정됐는지 본다.
   **프로가 쓰는 스택**: 바탕 → 글로우/스크림 → 판(그림자·글래스) → 글자(획 2겹 + 그림자 2겹, 필요하면 메탈 그라데) → 손그림 장식(svg 패스). 5~6겹이 기본이다.
   레시피 정본: variety-caption-style(골드/실버 스톱값, 이중 테두리 비율, 블러글로우 발광판), ad-creative-core references/craft.md, 여기 references/svg-paths.md.

## 2. 5층과 축 (전체 어휘 = references/vocabulary.md)

| kind | 뜻 | 저장 단위 | 필수 |
|---|---|---|---|
| `primitive` | 원자. 레이어 1~N개짜리 디자인 아이디어 하나 | layer_names (bbox 정규화) | 3축 + treatments |
| `component` | 원자의 조합(락업). 헤드라인 스택, 칩 row, 푸터 가구 | layer_names | 3축 + treatments, lineage.primitive_ids |
| `composition` | 페이지 공간 배치 문법. `payload.slots[{role,x,y,w,h}]` | 페이지 전체 | structural_pattern ∈ COMPOSITION_PATTERNS |
| `treatment` | 시각 처리 레시피 — 어느 원자에나 씌우는 `{textFxProps|textProps|shapeProps}` 패치 | 샘플 레이어 + spec | spec |
| `style_family` | 디자인 언어 — `{palette, typography, border, image_treatment, spacing, decoration, geometry}` | 샘플 + spec | spec |

축 값(요약 — 전체는 references/vocabulary.md):
- `primitive_type`: typography · container · decoration · media · layout · information · editorial · surface
- `semantic_role`: headline · subhead · kicker · eyebrow · caption · body · quote · number · stat · price · rank · step · label ·
  verdict · annotation · comparison · checklist · rating · progress · cta · source · footer · page_marker · product_name ·
  benefit · ingredient · shade · before_after · definition · takeaway · reaction · photo · ground · divider
- `structural_pattern`(원자/조합): single · stack · row · split · overlap · offset · anchored · floating · framed · inline ·
  radial · diagonal · asymmetric · grid · boxed · underlined · rotated · masked · full_bleed · vertical · arched · ticker
- `structural_pattern`(배치): centered_hero · editorial_split · asymmetric_grid · poster_stack · magazine_cover ·
  scrapbook_collage · product_spotlight · comparison_split · listicle_stack · quote_focus · full_bleed_caption ·
  floating_annotation · diagonal_flow · dense_info_grid · minimal_whitespace · overlap_collage · table_page · step_page ·
  ranking_page · before_after · top_band · bottom_band · letterbox · center_card
- `treatments[]`: outline · double_outline · filled · highlight · marker · hard_shadow · soft_shadow · glow · metallic · gradient ·
  handwritten · neon · retro · minimal · ghost · halftone · grain · paper · tape · stamp · scribble · italic_mix · two_tone ·
  condensed · oversized · tracked_caps · glass · sticker · hairline · inverted
- `style_family` 는 이펙트 이름이 아니라 **디자인 언어**: swiss_editorial · y2k_pop · beauty_magazine · ugc_sticker · scrapbook ·
  minimal_luxury · korean_editorial · japanese_magazine · infographic · brutalist · soft_cosmetic · pop_commerce · tabloid ·
  tech_editorial (+ 기존 fresh_sticker · corporate_ink · product_editorial · editorial_compare · keycap_dark · news_brief …)

### 신규성 점수 기준 (novelty, 서버 가중합 = semantic .3 + structural .3 + geometry .2 + composition .1 + visual .1)

| 무엇이 다른가 | 대략 |
|---|---|
| 색만 | 0.05 → 변주 |
| 폰트 + 색 | 0.10 → 변주 |
| pill → 사각형(기하만) | 0.20 → 변주 또는 treatments 차이로 처리 |
| 헤드라인 구조 자체(stack→split, 글자↔사진 관계) | 0.60 |
| 새로운 이미지/텍스트 관계 | 0.80 |
| 코퍼스에 없는 정보 구조(pros/cons, timeline, 점수 미터…) | 0.90 |

## 3. 도구

| 도구 | 언제 |
|---|---|
| `get_corpus_coverage` | 코퍼스를 키우거나 "새 디자인" 요청을 받으면 **첫 호출**. summary·matrix·gaps·unlabeled·vocabulary 를 준다 |
| `search_design_assets` | kind·3축·treatment·style_family 로 검색. 기본은 독립 재료만(variant_count 표시). `variants='only'`/`variant_of=` 로 변주 |
| `save_design_asset` | kind + name + layer_names + 3축 + treatments (+ novelty / variant_of / spec_json). 응답의 `near_duplicates` 를 읽는다 |
| `apply_design_asset` | 원자/조합은 x,y 에 놓기, composition 은 페이지 교체(스켈레톤이면 자리 사각형이 깔린다), treatment 는 `layer_ids` 에 패치 |

이름 규약: snake_case **개념명**(`double_outline_pop_line`, `speech_bubble`, `comparison_split`). 스타일명(`gold_`, `neon_`)을
이름 앞에 붙이지 않는다 — 그건 treatments 가 말한다. 레이어 이름도 저장 전에 의미 있게 붙인다(`hl1`, `kicker_pill`, `cap1`).

## 4. 워크플로

### A. 페이지를 저장할 때 — 분해(atomize)
1. 페이지의 레이어를 역할별로 본다. **시각 처리가 다른 줄/장치마다** `save_design_asset(kind='primitive')` — 3축 + treatments.
2. 그 줄들의 배열(스택·row·split)을 `kind='component'` 로 저장 — `lineage` 에 원자 id 를 적는다(설명에 열거해도 됨).
3. 페이지 전체가 새로운 공간 배치라면 `kind='composition'` + 배치 패턴. 배치가 이미 있는 패턴이면 저장하지 않는다.
4. 서버가 near-duplicate 400 을 주면: 색·폰트만 다르면 `variant_of`, 아니면 novelty + description 을 채워 재시도.

### B. 코퍼스를 키울 때 — 갭 분석 → 생성 (절대 "예쁘게 10개 더" 로 시작하지 않는다)
1. `get_corpus_coverage`. `gaps`(0개인 목표 개념)·비어 있는 composition 패턴·비어 있는 treatments 를 본다.
2. **생성 전에 제안서를 쓴다** (사용자에게 보여준다): 채울 gap 10~20개를 고르고 각각
   `semantic purpose · structural novelty · use cases · compatible content · 조합 가능한 기존 원자 · 왜 기존과 중복이 아닌가` 한 줄씩.
   구조적 신규성 높은 순으로 정렬한다.
3. 스크래치 피드 draft(`create_feed_draft`, 4:5, page_count = 만들 개수/2 정도)에 **한 페이지에 원자 2~3개**를 크게 그린다.
   textFx 는 `fx` 필드, 도형은 `backgroundOpacity 1` 명시(ad-creative-core 함정 그대로). `preview_edit_frame` 으로 확인.
   **글자는 textFx 로 그린다**(획·이중 획·그림자 복사·그라데이션·roughen 을 한 레이어에서). 손그림 장치(화살표·스크리블·테이프·찢김·버스트·도장)는
   `shapeKind:'svg' + svgPathData`(references/svg-paths.md) — 사각형·원·삼각형으로 흉내내지 않는다. 사진 자리는 회색 판이 아니라 실제 image 레이어
   (`list_layer_images` 의 제품 컷아웃)나 `generate_image` 결과를 쓴다.
4. 각 원자를 `save_design_asset(kind='primitive', layer_names, 3축, treatments, style_family, tags, description)`.
   같은 정체성이 있다고 거부되면 그 개념은 이미 있는 것 — 다음 gap 으로.
5. 배치 gap(magazine_cover, quote_focus, floating_annotation, diagonal_flow, table_page, step_page, ranking_page, before_after…)은
   슬롯이 보이는 페이지를 실제로 조립해서 `kind='composition'` 으로 저장한다(스켈레톤은 서버 스크립트가 아니라 실제 페이지에서도 뽑힌다).
6. 끝나면 `get_corpus_coverage` 를 다시 불러 concepts_covered 가 늘었는지 숫자로 보고한다.

### C. 코퍼스로 새 디자인을 만들 때 — 합성(compose), 복사가 아니다
1. 브리프에서 **배치 문법**을 먼저 고른다: `search_design_assets(kind='composition', structural_pattern=…)`.
   시리즈가 이미 쓰던 배치는 피한다(`get_recent_layouts` 의 avoid 와 같은 논리).
2. 슬롯 역할마다 원자를 고르되 **서로 다른 style_family 에서 섞는다** — 예: `comparison_split` 배치 + `product_cutout`(media) +
   `ranking_badge`(container) + `handdrawn_arrow`(decoration) + 디자인 언어 `japanese_magazine`.
3. 처리(treatment)를 한 겹 얹는다. 색·폰트는 style_family spec 에서 가져온다.
4. 조립 결과가 코퍼스에 없는 배열이면 그 페이지를 다시 A 로 저장한다 — 이게 "쌓일수록 좋아지는" 경로다.
5. `apply_design_asset` 으로 놓고 텍스트·사진만 바꾸는 건 **시리즈 다음 화**(feed-carousel `duplicate_draft` 규칙)에서만.
   "새 디자인" 요청에는 쓰지 않는다.

## 5. 함정 (실측)

- `variant_of` 를 주면 3축은 기준에서 물려받고 `treatments` 만 이 변주의 것으로 저장된다 — 3축을 다시 줄 필요 없다.
- `composition` 은 near-duplicate 게이트가 없다(패턴이 같아도 슬롯 배치가 다르면 다른 문법). 대신 `structural_pattern` 이 COMPOSITION_PATTERNS 에 없으면 400.
- `treatment` / `style_family` 는 `spec_json` 이 정체성이다. 없으면 400. treatment 는 apply 때 `layer_ids` 필수.
- 스켈레톤 composition 을 apply 하면 페이지가 자리 사각형(`slot:<role>`)으로 바뀐다 — 슬롯마다 원자를 놓고 자리 사각형은 `delete_layer`.
- 자동 추정 라벨(devices/typography/palette/photo_dependency/density)은 정체성이 아니다. 검색 필터로만 쓴다.
- 새 계정의 코퍼스는 비어 있을 수 있다. 그때는 B(갭 분석→생성)로 시작한다.

## 6. 생성 실측 함정 (1)

- **`apply_edit_batch` 의 `add_layer` 는 z_index 를 무시한다** — 텍스트는 30부터 오름차순, 도형은 전부 z 0(배열 순서 = 나중이 위). 도형끼리 겹치면
  **아래 것을 먼저** 넣거나 `update_shape_layer` patch `{zIndex}` 로 바로잡는다. `add_text_layer` 단독 호출은 z_index 를 지킨다(도형 z 0 위에 두려면 ≥1).
- batch 의 텍스트는 `text_props` 부분집합만 받는다(text/fontFamily/fontSizeRatio/fontColor/fontWeight/textAlign/verticalAlign). letterSpacing·lineHeight·runs·
  writingDirection·textDecoration 이 필요하면 `add_text_layer` 를 따로 부른다. 4:5 에서 fontSizeRatio = 목표px / 1688, runs 의 fontSize 는 편집px(렌더 = ×1.25).
- `shape.borderRadius` 상한 500(pill 은 200 이면 충분). `shapeKind: circle` 은 **박스 폭 기준 원**을 그리고 높이로 잘라 버린다 — 타원이 아니라 원이 필요하면 박스를 정사각(w×1080 = h×1350)으로.
- 프리뷰는 `(format_idx, at_ms)` 로 캐시된다 — 같은 페이지를 다시 보려면 at_ms 를 10, 20… 으로 바꾼다.
- 글리프 tofu 실측: Bebas Neue 에 ✦ 없음(• 는 있음), Instrument Serif Italic 에 IPA(ˈ ʌ ɪ ŋ) 없음. Pretendard 는 ★ → ✓. Caveat Bold 는 라틴만.
- 자리 라벨은 `_lbl_*` 이름으로 두면 인제스트가 무시한다. 원자 레이어는 `<개념명>__<부위>` 로 이름 짓고, 조합은 prefix 여러 개로 잡는다.

## 7. 생성 실측 함정 (2)

- **프리뷰 캐시는 프레임 단위**(30fps): at_ms 50/60/70 은 전부 프레임 2 → 같은 그림. 재확인은 at_ms 를 **34ms 이상** 올린다(0 → 40 → 100 → 130 …).
- svg 에는 `svgPreserveAspectRatio:'none'` 이 필요하다(비율 유지 vs 늘림의 문제). 점선 리더처럼 **점 모양을 지켜야 하면 viewBox 를 박스 비율로**(`0 0 100 12`) 만들고 meet 로 둔다.
- batch `add_layer` 의 `text_props` 에는 그림자·획이 없다 → 평문이 마감 게이트 장치 0 이 된다. 본문 평문에도 `update_text_layer` patch `{shadowColor, shadowBlur, shadowOffsetY}` 로 그림자 1개를 준다.
  (batch `update_text_layer` patch 는 shadow*·outline*·runs·fontGradient 를 받는다. `textShadows` 배열은 `add_text_layer`/`update_text_layer` 단독에서만.)
- 손그림 링은 굵기 5 안팎 + 두 번째 획 3 안팎(어긋나게). 9 이상이면 마커가 아니라 도넛으로 보인다.

## 8. 생성 실측 함정 (3)

- batch `add_layer` 의 **image 도 텍스트처럼 z 30+** 를 받는다(도형만 z 0). 사진 위에 얹을 스크림·글자·점·마스크는 `zIndex 60` 으로 올려라(update_shape_layer / update_text_layer patch, textFx 는 update_text_fx_layer z_index).
- image 의 `mask` 는 **리빌 애니메이션**이지 정적 클립이 아니다. 원형 크롭은 흰 바탕 한정으로 svg `circle_mask_frame`(사각 − 원, evenodd) 을 이미지 위(z 6)에 덮는다. 사각 크롭은 image `objectFit: cover` 로 충분.
- 회전 락업의 글자는 **textFx** 로 — text 레이어 rotation 은 도형과 회전 중심이 어긋나 pill 밖으로 튄다(diagonal_flow CTA 실측).
- 크기 감: text `fontSizeRatio × 1688 = 렌더 px`(4:5). 464px 셀에서 .02 → 28자, .017 → 34자. 종서는 405px 높이에 ~16자, 넘치면 두 번째 줄로 접힌다.
- 아치 텍스트는 글자별 textFx: 중심 (cx, cy) 반지름 r, 각 a 에 대해 x = cx + r·sin a − w/2, y = cy − r·cos a − h/2, rotation = a. 7자면 −54°…+54° 18° 간격.
- 할프톤은 svg 원 60개(viewBox 120×80, 반지름 4.6→0.7)를 스크립트로 생성(references/svg-paths.md `halftone_60`). path 는 3.2KB, 상한 20KB.
- **레퍼런스 재현 실측(2026-09-06, 종이 질감 포스터)**: ① 종이/벽 질감은 `generate_image(provider:"openai")` 로 구김 종이 텍스처를 뽑아 full-bleed image + 같은 이미지를 **multiply 0.4** 로 한 겹 더(주름 대비). Vertex(imagen-4) 는 404 라 openai 로. ② image 레이어 **z 음수는 페이지 배경 아래로 사라진다** — 바탕 이미지는 z 0, 나머지 도형을 1+ 로. ③ 이모지는 렌더러에 컬러 이모지 폰트가 없어 tofu → 3D 아이콘을 흰 배경으로 생성하고 **흰 라운드 타일** 위에 얹어 배경을 숨긴다. ④ 찢은 종이 띠는 난수 지그재그 svg(위·아래 가장자리, 좌우는 직선) + 그라데이션 + 그림자, 뒤에 회색 스크랩 조각(z 1) 한두 장. ⑤ `export_feed_pages` 는 `f{idx}-0ms` 경로로 캐시돼 수정 후 재수출해도 같은 파일이 온다 — 검증은 `preview_edit_frame` at_ms 를 바꿔서, 최종은 UI 에서 내보내기.

## 9. AI 레퍼런스 → 재구성 워크플로 (2026-09-06 실측, 사용자 제안)

"창조" 가 평균값으로 회귀하는 문제의 우회로: **gpt-image-2 로 완성된 피드 한 장을 먼저 뽑고(취향·치수·물성이 한 점으로 고정됨), 그걸 레이어로 옮긴다.** 옮긴 페이지는 텍스트·사진만 바꿔 같은 유형을 계속 찍을 수 있고, 배치 문법으로 코퍼스에 저장할 수 있다.

1. `generate_image(provider:"openai", aspect_ratio:"4:5", no_text_suffix:false)` 에 **주제(한국어 문구 포함) + 디자인 언어 + 구성요소 목록**을 적는다. 한글 조판까지 대체로 맞게 나온다. **`no_text_suffix:true` 는 오히려 "텍스트 금지" 문구를 붙인다(플래그 의미 반대)** — 레퍼런스용은 false.
2. 생성은 비동기다(30~90초). 완료 후 결과 이미지를 열어 직접 본다.
3. 분해: 바탕(격자/질감) · 헤드라인 폰트/크기 · 액센트 1색 · 카드/행 구조 · 아이콘 · 손글씨 · 사진. 없는 재료는 만든다 — 아이콘은 evenodd svg(링+점, 방패, 느낌표, 막대그래프, 전구), 격자는 얇은 사각형 path, 손글씨는 `EF_제주돌담체(OTF)`, 사진 질감(세럼 얼룩·종이)은 흰 배경으로 생성해 흰 카드 위에 얹는다.
4. 조립 후 `apply_edit_batch` 응답의 `warnings[kind=text_overflow]` 가 실측 폭(needs Npx > box Mpx)을 준다 — 이걸로 폰트/박스를 조정한다(줄바꿈 있는 텍스트는 한 줄로 재서 과대평가함, 프리뷰로 최종 판단).
5. 실측: 나이아신아마이드 5% vs 10% 비교 페이지를 이 방식으로 재구성했을 때 조판 재현도가 높았다. 헤드라인 폭·카드 내부 여백은 레퍼런스가 더 촘촘했다.
- **Paperlogy-9Black 은 textFx 에서 렌더 워커가 간헐적으로 기본 폰트로 떨어뜨린다**(원인 미확인). 굵은 한글 헤드라인은 `Freesentation-8ExtraBold` 가 안정적 — 이걸 기본으로.

## 10. 코퍼스 루프 — 쌓일수록 좋아지는 절반

코퍼스 → 생성(apply)만 있으면 `used_count` 는 "꺼내 쓴 횟수" 지 "잘 된 횟수" 가 아니다. 이제 생성 → 코퍼스로 돌아오는 신호가 있다:

```
코퍼스 ──apply_design_asset──▶ draft   (어느 재료가 어디에 놓였는지 + 그때 스냅샷)
   ▲                            ├─ 사람이 에디터에서 저장 → 무엇을 얼마나 고쳤나
   │                            ├─ 렌더 성공 / 공유 링크 / 게시
   │  sweep_corpus_outcomes — 스냅샷 vs 현재: kept / edited / removed + rendered / shared / published
   │  quality.score ← search_design_assets 기본 정렬(sort='score')
   └──save_design_asset(source='internal_harvest')── list_harvest_candidates
```

### 10.1 순위 읽는 법

- `score` = 승인 +3 · 무수정 렌더(kept) 가장 큼 · 렌더 · 공유/게시 · 신규성 − 고쳐짐(edited) − 지워짐(removed). 카운트는 전부 log 라 한두 개가 독점 못 한다.
- 검색 시점에만 **탐색 보너스**: 14일 안 된 미사용 재료 +0.6, 날짜·id 지터 0~0.3(동률 회전). 같은 질의도 날마다 조금 다른 순서 — 무작위가 아니라 제약 안의 회전이다.
- `outcomes {kept, edited, removed}` 와 `edited_props`(사람이 이 재료를 놓은 뒤 가장 많이 고친 속성: `x`, `textProps.fontFamily`, `motion.enter`…)를 읽는다. **edited_props 는 교정 데이터다** — 같은 실수를 반복하지 않는다.
- 글자·사진 교체는 edited 로 안 센다. 위치·크기·타이밍·폰트·색·모션 같은 디자인 판단만 센다.

### 10.2 합성할 때 (§4 C 에 얹는 규칙)

1. 상위 3개만 쓰지 않는다. 슬롯마다 **검증된 것 1 + 미검증 1** 을 후보로 두고 브리프에 맞는 쪽을 고른다. 미검증이 렌더까지 가면 그게 다음 표본이 된다.
2. `outcomes.removed` 가 kept 보다 큰 재료는 브리프가 정확히 그걸 원할 때만.
3. 구조는 표본에서, 표면은 이번 브리프에서. 저장 게이트의 신규성 어휘를 생성 시점에도 스스로 적용한다 — 표본 근처지만 같지 않은 지점에 선다.
4. 자유도를 올린 만큼 판정을 세게: `preview_edit_frame`(피드) / `lint_motion` + `preview_motion_strip`(영상) 을 통과시킨 뒤 끝낸다.

### 10.3 수확 — 우수사례를 코퍼스로

1. `list_harvest_candidates` — 렌더·공유·게시된 draft 중 아직 수확 안 된 것. `verdict`: `kept`(Claude 가 만들었고 사람이 거의 안 고침 → 그대로 표본) · `corrected`(사람이 25% 이상 고침 → **사람이 고친 최종본이 표본**, `edits.top_changed_props` 가 틀린 지점) · `human_made`(사람이 만듦 → 취향 표본).
2. `get_edit_draft` 로 열고 §4 A(분해) 그대로: 줄/장치마다 primitive, 락업은 component, 페이지 문법은 composition, **움직이는 락업은 kind='motion'**.
3. `save_design_asset(..., source='internal_harvest')`. 신규성 게이트는 그대로 — 색만 다른 건 `variant_of`.
4. 수확한 draft 는 후보에서 빠진다. 다시 렌더/공유되면 다시 후보.

### 10.4 모션 재료 (kind='motion')

- 영상 draft 의 mograph 레이어 / `motion.enter|exit|loop` 가 있는 레이어 묶음. 저장 시 가장 이른 startMs 가 0 이 된다. `motion_devices`·`duration_ms` 가 붙는다.
- 축: primitive_type 은 자동 `motion`. semantic_role 은 기존 어휘, structural_pattern ∈ `sequence | stagger | reveal | loop | kinetic | count | transition`.
- 움직이는 레이어가 1개도 없으면 저장 거부(정지면 원자/조합으로). 마감(craft) 게이트는 모션엔 안 건다.
- 적용: `apply_design_asset(asset_id, draft_id, format_idx, at_ms)`. 피드 draft 에는 못 놓는다.
- 템플릿 자체는 부품이고, 코퍼스에 쌓이는 건 **어떤 상황에 어떤 템플릿을 어떤 파라미터·순서·스태거로 얹었나** 하는 연출 판단이다.

### 10.5 수렴 방지

자기 산출물만 표본 삼으면 한 룩으로 좁아진다. `get_corpus_coverage.summary` 의 `source_internal_harvest` 대 `source_external_reference` 비율을 보고, 외부 레퍼런스를 저장할 땐 `source='external_reference'`. 루프가 도는지의 지표는 하나 — `uses_kept` 가 `uses_edited + uses_removed` 보다 커지는가.
