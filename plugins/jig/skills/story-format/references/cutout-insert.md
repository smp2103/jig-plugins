# 누끼 인서트 — 생성 + 크로마키 절차 (run_code)

SKILL.md "누끼 인서트" 절의 2번 경로. Jig MCP 도구만으로 끝난다: `generate_image` → `run_code`(키잉·업로드) → `add_image_layer` → `set_layer_motion`.

## 1. 생성

항목 하나에 이미지 하나. 1:1, 단색 초록 배경.

```
generate_image(project_id, aspect_ratio='1:1', no_text_suffix=true,
  prompt="A single <item>, studio product photograph, centered, isolated on a flat solid bright green chroma-key background (#00FF00) that fills the entire frame, no shadow on the background, no reflections of green on the object, no text, no other objects")
```

완료 확인은 `run_code` 안에서 `jig.api('/image-gen/{id}/status', method='GET')` 의 `status == 'completed'`. 보통 30~60초.

## 2. 키잉 + 업로드 (run_code)

```python
import jig, os, numpy as np
from PIL import Image, ImageFilter
PID = '<project_id>'
ITEMS = {'coffee': '<image id>', 'soda': '<image id>', 'beer': '<image id>'}
os.makedirs('out', exist_ok=True)

def key_green(path):
    im = Image.open(path).convert('RGB')
    a = np.asarray(im).astype(np.float32)
    r, g, b = a[..., 0], a[..., 1], a[..., 2]
    gd = g - np.maximum(r, b)                # 초록 우세도. 배경 ≈ 240, 물체 ≈ 0 이하
    near = gd > 60
    H, W = near.shape
    bg = np.zeros_like(near)                 # 테두리에서 이어진 초록만 배경으로(플러드필)
    bg[0, :] = near[0, :]; bg[-1, :] = near[-1, :]; bg[:, 0] = near[:, 0]; bg[:, -1] = near[:, -1]
    for _ in range(4000):
        grown = bg.copy()
        grown[1:, :] |= bg[:-1, :]; grown[:-1, :] |= bg[1:, :]
        grown[:, 1:] |= bg[:, :-1]; grown[:, :-1] |= bg[:, 1:]
        grown &= near
        if (grown == bg).all(): break
        bg = grown
    bg = bg | (gd > 140)                     # 손잡이 구멍처럼 고립된 진초록도 배경
    ring = (np.asarray(Image.fromarray((bg * 255).astype(np.uint8)).filter(ImageFilter.MaxFilter(7))) > 0) & (~bg)
    alpha = np.ones_like(gd); alpha[bg] = 0
    alpha[ring] = (1.0 - np.clip((gd - 10.0) / 60.0, 0, 1))[ring]   # 가장자리 7px 소프트
    rgb = a.copy(); spill = ring & (gd > 0)
    rgb[..., 1][spill] = np.maximum(r, b)[spill]                      # 디스필: 가장자리 초록 빼기
    am = Image.fromarray((alpha * 255).astype(np.uint8)).filter(ImageFilter.GaussianBlur(0.6))
    rgba = Image.merge('RGBA', (*Image.fromarray(rgb.clip(0, 255).astype(np.uint8)).split(), am))
    bb = am.point(lambda v: 255 if v > 8 else 0).getbbox(); p = 10
    return rgba.crop((max(0, bb[0] - p), max(0, bb[1] - p), min(W, bb[2] + p), min(H, bb[3] + p)))

out = {}
for name, iid in ITEMS.items():
    rec = jig.api(f'/image-gen/{iid}/status', method='GET')
    jig.download(rec['public_url'], f'{name}.png')      # public_url 은 서명 URL 이어야 한다
    cut = key_green(f'{name}.png')
    cut.save(f'out/{name}_cut.png')
    up = jig.upload_image(f'out/{name}_cut.png', f'{name} 누끼', project_id=PID)
    out[name] = {'w': cut.size[0], 'h': cut.size[1], 'asset_id': up['id'], 'url': up['url'].split('?')[0]}
print(out)
```

`out/*.png` 는 응답에 첨부되므로 체커보드 없이도 구멍·잔여 초록을 눈으로 확인한다.

## 3. 얹기

```
add_image_layer(draft_id, format_idx, url=<out.url>, asset_id=<out.asset_id>, object_fit='contain',
  x=0.06, y=0.345, width=0.30, height=0.30 × 1080/1920 × h/w, z_index=24,
  start_ms=<단어 시작>, end_ms=<씬 끝>)
set_layer_motion(draft_id, format_idx, layer_id, motion={enter:{type:'pop', easing:'spring', durationMs:420}})
update_layer_box(draft_id, format_idx, layer_id, patch={effects:[{type:'dropShadow', params:{radius:18, x:0, y:14, color:'rgba(0,0,0,0.55)'}}]})
```

## 실측(2026-09-19, 장염 릴스 5번 씬)

| 항목 | 단어 시작 | 레이어 | 결과 |
|---|---|---|---|
| 커피 | 씬+4.19s | x 0.06 / w 0.30 | 흰 배경 생성본은 잔 윗면이 잘려 폐기 → 초록 배경 재생성으로 해결 |
| 탄산 | 씬+4.49s | x 0.39 / w 0.22 | 은색 캔, 초록키 깨끗 |
| 술 | 씬+4.79s | x 0.66 / w 0.26 | 홀드 0.5초라 씬 꼬리 +0.5초 연장(뒤 씬 전부 +500ms) |

무료 검색 결과(같은 검색에서 만화 캔·벡터 컵·사진 맥주가 섞여 나옴)는 한 화면에 못 놓아 버렸다.
