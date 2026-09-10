# 코퍼스 어휘

## kind (5층)

| kind | 정의 | payload | 정체성 |
|---|---|---|---|
| primitive | 디자인 아이디어 하나짜리 최소 레이어 묶음 | layers(bbox 정규화) + anchor | 3축 + treatments |
| component | 원자의 조합(락업) | layers | 3축 + treatments (+ lineage.primitive_ids) |
| composition | 페이지 공간 배치 문법 | layers(자리 사각형) + `slots[{role,x,y,w,h}]` | structural_pattern(배치 어휘) |
| treatment | 시각 처리 레시피 | 샘플 layers + `labels.spec = {textFxProps?, textProps?, shapeProps?}` | spec |
| style_family | 디자인 언어 | 샘플 layers + `labels.spec = {palette, typography, border, image_treatment, spacing, decoration, geometry}` | spec |

`variant_of` 가 있으면 어떤 kind 든 **변주** — 색·폰트·획·그림자·팔레트·질감만 다른 것. 어휘에 세지 않는다.

## primitive_type (8)

typography · container · decoration · media · layout · information · editorial · surface

## semantic_role (34)

headline · subhead · kicker · eyebrow · caption · body · quote · number · stat · price · rank · step · label · verdict ·
annotation · comparison · checklist · rating · progress · cta · source · footer · page_marker · product_name · benefit ·
ingredient · shade · before_after · definition · takeaway · reaction · photo · ground · divider

## structural_pattern — 원자/조합 (22)

single · stack · row · split · overlap · offset · anchored · floating · framed · inline · radial · diagonal · asymmetric ·
grid · boxed · underlined · rotated · masked · full_bleed · vertical · arched · ticker

## structural_pattern — 배치(composition) (24)

centered_hero · editorial_split · asymmetric_grid · poster_stack · magazine_cover · scrapbook_collage · product_spotlight ·
comparison_split · listicle_stack · quote_focus · full_bleed_caption · floating_annotation · diagonal_flow · dense_info_grid ·
minimal_whitespace · overlap_collage · table_page · step_page · ranking_page · before_after · top_band · bottom_band ·
letterbox · center_card

```
CENTERED HERO            ASYMMETRIC / EDITORIAL SPLIT       FLOATING ANNOTATION
      badge              HEADLINE                            ┌────────────┐
     HEADLINE            HEADLINE      [badge]               │   PHOTO    │ ← label
     HEADLINE                    ┌─────────                   │            │
       photo                     │ PHOTO                      └────────────┘
      caption            caption ┘        ↗ arrow             note ↘   ↙ note
```

## treatments (30)

outline · double_outline · filled · highlight · marker · hard_shadow · soft_shadow · glow · metallic · gradient · handwritten ·
neon · retro · minimal · ghost · halftone · grain · paper · tape · stamp · scribble · italic_mix · two_tone · condensed ·
oversized · tracked_caps · glass · sticker · hairline · inverted

## style_family — 디자인 언어 목표 (14) + 기존 계열

swiss_editorial · y2k_pop · beauty_magazine · ugc_sticker · scrapbook · minimal_luxury · korean_editorial · japanese_magazine ·
infographic · brutalist · soft_cosmetic · pop_commerce · tabloid · tech_editorial
기존: fresh_sticker · campaign_kv · corporate_ink · product_editorial · editorial_compare · keycap_dark · news_brief · anton_guide ·
playfair_guide

style_family 행(kind='style_family')의 spec 예:
```json
{ "palette": { "ink": "#14161A", "ground": "#FBF8F3", "accent": "#E4572E", "accent2": "#1E9E6A" },
  "typography": { "display": "Instrument Serif", "text": "Pretendard", "caps_tracking": 6 },
  "border": { "rule": "hairline 1px ink", "chip": "2.5px ink, radius 40" },
  "image_treatment": "cutout with hard drop shadow",
  "spacing": { "margin": 0.074, "gutter": 0.03 },
  "decoration": ["scribble_circle", "handdrawn_arrow"],
  "geometry": "rotated stickers, no rounded photo corners" }
```

## 목표 개념 (TARGET_CONCEPTS, 129) — 축 3개가 정체성, 이름은 설명

Typography: hero_headline · stacked_headline · split_headline · highlighted_keyword_headline · boxed_keyword_headline ·
oversized_word · underlined_headline · arched_headline · rotated_headline · vertical_headline · offset_headline ·
overlap_headline · number_headline · stat_headline · eyebrow_label · section_kicker · subhead_line · body_caption ·
caption_stack · side_caption · floating_caption · body_paragraph · pull_quote · price_lockup · percentage_stat ·
ranking_number · ghost_number · product_name_lockup · ticker_line

Containers: label_pill · kicker_pill · verdict_chip · benefit_badge · ingredient_chip · shade_chip · ranking_badge ·
rating_lockup · award_stamp · price_tag · chip_row · sticker_tile · speech_bubble · comment_bubble · floating_card ·
ticket_stub · tab_label · definition_box · key_takeaway_box · cta_button · source_tag · band_title

Decoration: underline_stroke · scribble_circle · marker_highlight · handdrawn_arrow · star_cluster · sparkle · tape_strip ·
stamp_mark · corner_bracket · progress_dots · axis_line · hairline_rule · dotted_divider · radial_glow · doodle_cluster

Media: photo_crop · cutout_subject · polaroid_frame · circle_crop · device_frame · split_crop · overlapping_photos ·
photo_strip · photo_grid · scrapbook_photo · cutout_fan · diagonal_photo · fullbleed_photo

Information: ranking_list · pros_cons · score_meter · numbered_steps · step_row · comparison_table · comparison_bar ·
checklist · rating_bar · progress_bar · stat_block · stat_row · do_dont · before_after_label · key_value_rows · timeline ·
annotation_label · shade_swatch_row

Editorial/Social: issue_number · chapter_marker · page_counter · category_kicker · editor_note · source_label · footnote ·
footer_furniture · article_tag · quote_card · testimonial_quote · review_card · reaction_row · callout · verdict_stamp ·
editor_pick · hot_take · table_of_contents · index_list

Surface: scrim · paper_texture · grid_paper · gradient_wash · halftone_field · checker_field · blob_shape · spotlight ·
band_surface · letterbox_bars · diagonal_panel

Layout: stack_lockup · split_lockup · overlap_lockup · anchored_lockup · floating_lockup · diagonal_lockup · grid_lockup
