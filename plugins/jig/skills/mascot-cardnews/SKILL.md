---
name: mascot-cardnews
description: "한국식 카드뉴스(마스코트 캐릭터 + 구겨진 종이 배경 + 통통한 아웃라인 제목 + 제품 카드 + 하단 브랜드 밴드)를 Jig 피드 draft 로 만든다. \"카드뉴스 만들어줘\", \"마스코트 넣어서\", \"귀여운 캐릭터 스타일로\", \"정보성 카드 6장\" 요청 시 사용. 마스코트는 이미지 생성으로 히어로 1장을 확정한 뒤 레퍼런스 체이닝으로 포즈를 변주한다."
---

# 마스코트 카드뉴스

## 1. 무엇을 만드는가

- 결과물: 1:1 1080×1080 카드뉴스 6장 — 커버 / 픽 ×3 / 하우투 / 저장.
- 어울리는 내용: 제품 3개 추천 + 사용법 팁, 정보성 요약.
- 에디토리얼 캐러셀(`feed-carousel`)과는 다른 장르다. 박스·카드·라벨 필이 이 장르의 문법이라 써도 된다.

## 2. 재료 만들기 (전부 Jig 도구)

1. **마스코트 히어로** — `generate_image`(aspect_ratio `1:1`, count 1).
   프롬프트 뼈대: `A cute flat cartoon mascot girl for a K-beauty brand, holding a sheet mask, oversized pastel-pink hoodie, towel headband, thick bold black outlines, limited palette of pink/black/white only, simple vector sticker illustration style, full body, centered, plain pure white background, no text`.
   **팔레트를 3색으로 못박고 "plain pure white background, no text" 를 붙인다.** 흰 배경이 깨끗해야 흰 카드·흰 밴드 위에 얹었을 때 경계가 안 보인다.
2. **포즈 변주** — 같은 도구에 `reference_image_urls: [히어로 이미지 URL]` 을 넣는다.
   프롬프트: `The same cartoon mascot girl as the reference image, identical character design, same palette, now <포즈>, full body, centered, plain pure white background, no text`.
   환호·손가락으로 가리키기·앉기 3포즈에서 캐릭터가 유지된다. **레퍼런스 없이 뽑으면 매번 다른 캐릭터가 나온다.**
3. **배경 정리** — 가장 안정적인 방법은 흰 배경 컷을 **흰 카드·흰 밴드·밝은 종이 위에** 얹는 것이다.
   투명 배경이 꼭 필요하면 `edit_image` 로 배경 제거를 지시해 보되, 결과가 투명 PNG 가 아닐 수 있으니 `preview_edit_frame` 으로 확인한다.
4. **종이 질감** — `generate_image` "Crumpled white paper texture, top-down flat scan, subtle soft creases, seamless, no objects, no text" → 풀블리드 이미지(objectFit cover, opacity 0.85).
5. **도들(반짝이·별·풀)** — 이미지 생성은 이런 작은 장식에서 자주 실패한다. `design-corpus` 스킬 `references/svg-paths.md` 의 `sparkle_4` · `star_hand` 를 `add_shape_layer`(shapeKind `svg`)로 그린다.
6. **제품 사진** — 사용자가 준 사진이나 사용 권리가 확인된 컷만 쓴다. 할인·증정 배지가 박힌 컷은 즉시 낡는 정보라 쓰지 않는다.

## 3. 디자인 규율

- 팔레트: paper_bg `#F4F2EE` · accent `#F7B8C4` · ink `#1A1616` · white · gray `#6B6664`. 마스코트 색과 accent 를 맞춘다.
- 서체: 제목 **Titan One**(textFx) · 본문 Poppins Bold · 설명 Inter Medium/Regular. 한국어판은 Jalnan2 / Cafe24Ssurround-v2.0 / ONE Mobile POP / Pretendard.
- **아웃라인 제목 = textFx** (`add_text_fx_layer`):
  `{fill: accent, strokeColor: ink, strokeWidth: 7~8, stroke2Color: white, stroke2Width: 14~16, shadows: [{color: ink, x: 0, y: 8, blur: 0}]}`.
  둘째 줄은 fill white + stroke ink(stroke2 없음). 소제목은 fill ink + stroke white 6(섀도 없음).
- **라벨 필(PICK 1 / 01) = 프레임 + 자식 텍스트.** `add_frame_layer`(layout row · justify center · align center · padding 10) → `add_text_layer(parent_id=프레임)`. 도형과 텍스트를 따로 겹쳐 놓지 않는다 — 에디터와 렌더의 글자 폭이 달라 잘린다.
- 제품 카드: 흰 rect radius 22 + ink border 3 + 하드 섀도 `effects: {shadowColor: "rgba(26,22,22,0.18)", shadowBlur: 0, shadowOffsetX: 5, shadowOffsetY: 6}`. 안에 필 번호 · 이름(Poppins 24) · 구성(Inter 19) · 제품 컷.
- 하단 곡선 밴드: 큰 radius rect(x 는 0 이상 — 음수 좌표는 거부된다). 브랜드 밴드: accent 띠 + Titan One 34 자간 2.
- 마스코트 배치: 커버 중앙 하단, 픽 페이지 우측(카드와 겹치지 않게), 저장 페이지 중앙.

## 4. 콘텐츠 규율

- 페이지 = 커버(제목 2줄 + 서브) → 픽 N(라벨 필 + 결론어 큰 제목 + 상황 소제목 + 카드 1 + 설명 1문장) → 하우투(규칙 3개 카드) → 저장(SAVE THIS + 질문).
- **가격은 넣지 않는다**(변동). 구성(10 sheets a box)으로 대체한다.
- 효능 수치는 `*` + 각주 `*brand claim, not our test` 로 브랜드 주장임을 밝힌다.

## 5. 조립 절차

1. `create_feed_draft`(ratio `1:1`, page_count 6).
2. **커버(0페이지)**: 종이 질감 `add_image_layer`(z 0) → 곡선 밴드 `add_shape_layer` → 제목 textFx 2줄 → 서브 → 마스코트 `add_image_layer` → 브랜드 밴드.
3. **픽 템플릿(1페이지)** 1장을 완성한다: 라벨 필(프레임) → 큰 제목 → 소제목 → 제품 카드 → 설명 → 마스코트.
4. `add_feed_page(copy_from_idx=1)` 로 2장 더 복제하고 **내용만** 바꾼다 — 텍스트는 `update_text_layer` / `update_text_fx_layer`, 사진은 `delete_layer` 후 `add_image_layer`.
5. 하우투(4페이지) → 저장(5페이지).
6. `update_draft_meta` 로 제목·캡션을 저장한다.
7. QA(아래) 통과 후 보고. **`export_feed_pages` 는 사용자가 내보내라고 했을 때만.**

## 6. QA 게이트

- [ ] 6페이지 전부 `preview_edit_frame`(at_ms 를 바꿔 캐시를 피한다) — 제목 2줄이 서로·서브와 겹치지 않는가
- [ ] textFx 제목이 박스 폭 안에 있는가 — 12자 이상이면 fontSize 를 줄인다
- [ ] 라벨 필 텍스트가 필 안 가운데인가(프레임 자식인가)
- [ ] 마스코트가 카드·설명을 가리지 않는가, 흰 배경 경계가 보이지 않는가
- [ ] `*brand claim` 각주와 브랜드 밴드 문구가 들어갔는가

## 7. 실측 함정

- `add_frame_layer` 응답에 layer_id 가 안 실려 올 수 있다. 그때는 `get_edit_draft` 로 프레임 id 를 읽어 `parent_id` 로 쓴다.
- 레이어 x/y 는 [0,1] — 캔버스 밖으로 빼려고 음수를 주면 거부된다. 큰 radius rect 를 안쪽에서 쓴다.
- 캐릭터 일관성은 `reference_image_urls` 를 넣은 생성에서만 나온다.
- "sparkle", "grass tufts" 같은 장식 프롬프트는 자주 실패한다 — svg 패스로 그린다.
- Titan One 12자 제목은 fontSize 118 이 960px 박스에 딱 맞는다(SHEET MASK / STARTER PACK). 더 길면 줄인다.
- 종이 질감 opacity 0.85 면 텍스트 대비가 유지된다. 1.0 이면 대지색이 사라져 페이지 톤을 조절할 수 없다.
- 마스코트는 계정당 한 번 포즈 6~8장짜리 시트를 만들어 두면, 이후 화는 새로 생성할 필요가 없다.
