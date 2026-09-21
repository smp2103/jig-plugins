# 에셋 — run_code 에서 만든다

샌드박스에는 ffmpeg · PIL · cairosvg · rembg 가 있고, 네트워크는 Jig API·Jig 저장소만 열려 있다. 바깥 이미지는 먼저 `import_images_from_urls` 로 들여온 뒤 `jig.download` 한다. 올릴 땐 `jig.upload_image(path, label, project_id=jig.PROJECT_ID)` / `jig.upload_video(path, project_id=jig.PROJECT_ID)`.

## 선 아이콘 (24 그리드, stroke 2, round)

```python
import cairosvg
ICON = {
 'home':   '<path d="M3 11 L12 3 L21 11 V20 a1 1 0 0 1 -1 1 H15 V14 H9 V21 H4 a1 1 0 0 1 -1 -1 Z"/>',
 'folder': '<path d="M3 7 a2 2 0 0 1 2 -2 h4 l2 3 h8 a2 2 0 0 1 2 2 v8 a2 2 0 0 1 -2 2 H5 a2 2 0 0 1 -2 -2 Z"/>',
 'image':  '<rect x="3" y="4" width="18" height="16" rx="2.5"/><circle cx="8.5" cy="9.5" r="1.6"/><path d="M21 15.5 L16 10.5 L6 20"/>',
 'film':   '<rect x="2.5" y="6" width="13" height="12" rx="2.5"/><path d="M15.5 10.5 L21.5 7.5 V16.5 L15.5 13.5 Z"/>',
 'search': '<circle cx="11" cy="11" r="6.5"/><path d="M16 16 L21 21"/>',
 'plus':   '<path d="M12 5 V19 M5 12 H19"/>',
 'mic':    '<rect x="9" y="3" width="6" height="11" rx="3"/><path d="M5.5 11 a6.5 6.5 0 0 0 13 0 M12 17.5 V21"/>',
 'cc':     '<rect x="3" y="5" width="18" height="14" rx="2.5"/><path d="M7 11 H11 M13.5 11 H17 M7 15 H9.5 M12 15 H17"/>',
 'scissors':'<circle cx="6" cy="6.5" r="2.8"/><circle cx="6" cy="17.5" r="2.8"/><path d="M8.2 8.4 L20 19 M8.2 15.6 L20 5"/>',
 'sliders':'<path d="M4 7 H14 M18 7 H20 M4 17 H8 M12 17 H20"/><circle cx="16" cy="7" r="2"/><circle cx="10" cy="17" r="2"/>',
}
def icon_png(name, color, out):
    svg = (f'<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="{color}" '
           f'stroke-width="2" stroke-linecap="round" stroke-linejoin="round">{ICON[name]}</svg>')
    cairosvg.svg2png(bytestring=svg.encode(), write_to=out, output_width=144, output_height=144)
```

회색 `#8a93a3` 이 기본, 선택된 메뉴만 강조색 + 옅은 강조색 배경 판(72×72, `borderRadius 20`).

## 모서리 둥근 썸네일·카드 (이미지 레이어엔 둥글기가 없다)

```python
from PIL import Image, ImageDraw, ImageOps
def rounded(src, dst, size, rad):
    im = ImageOps.fit(Image.open(src).convert('RGB'), size, Image.LANCZOS).convert('RGBA')
    m = Image.new('L', (size[0] * 4, size[1] * 4), 0)
    ImageDraw.Draw(m).rounded_rectangle((0, 0, size[0] * 4 - 1, size[1] * 4 - 1), radius=rad * 4, fill=255)
    im.putalpha(m.resize(size, Image.LANCZOS)); im.save(dst)
```

스크린샷이 있으면 같은 방식으로 영역을 잘라 2배 해상도 그대로 올린다 — 실제 화면이 가장 좋은 밀도다.

## 파형 (나레이션 트랙)

폭 760×56 의 투명 PNG 에 9px 막대를 18px 간격으로, 높이는 `0.35 + 0.65·|sin(i/9)|·(0.6 + 0.4·sin(i/2.3+1))` 에 난수 0.55~1 을 곱해 그린다. 37개마다 3개는 낮게(숨 쉬는 구간).

## 폰이 솟아오르는 베이스 영상

1. 베젤 PNG(둥근 사각 + 먹색 테두리)와 화면 마스크(둥근 사각, **짝수 폭**)를 PIL 로 만든다.
2. 결과물에서 0.95초 컷 4개를 이어 화면 크기로 줄인다.
3. 종이색 캔버스 위에 글로우 PNG → 폰(베젤 + 마스크된 화면)을 같은 y 식으로 overlay:

```
p = clip((t - T0) / 0.6, 0, 1)        # 솟아오름 (ease-out cubic)
q = clip((t - TE) / 0.35, 0, 1)       # 위로 빠짐 (ease-in cubic)
y = 546 + 1500 * pow(1 - p, 3) - 1900 * pow(q, 3)      # overlay 에 eval=frame
```

`enable='between(t, T0, T_end)'`, 무음 오디오 트랙(anullsrc)을 같이 넣는다. 글로우는 강조색 α≈0.47 의 둥근 사각을 GaussianBlur 60 한 PNG.
