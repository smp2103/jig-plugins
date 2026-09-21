# 빌더 — run_code 에 저장해 두고 돌리는 조립 코드

레이어가 100개를 넘으므로 도구를 하나씩 부르지 않는다. 아래 도우미를 run_code 작업 공간에 한 번 저장하고(`session_id` 를 고정), 장면마다 짧은 코드로 불러 쓴다. 좌표는 **px 로 쓰고** 도우미가 0~1 로 바꾼다.

```python
# 저장: open('builder.txt','w').write(SRC)  /  불러오기: exec(open('builder.txt').read())
import jig, json, math
D = '<draft_id>'
W, H = 1080, 1920                      # 16:9 면 1920, 1080
INK, PAPER, ACCENT, GRAY, LINE, SOFT = '#14181f', '#fcfcfd', '#1677ff', '#9aa3af', '#e3e7ee', '#f1f4f8'
SNAP, ZOOM = 'cubic-bezier(0.16,1,0.3,1)', 'cubic-bezier(0.7,0,0.2,1)'
GLOW = {'shadowColor': 'rgba(22,119,255,0.22)', 'shadowBlur': 60, 'shadowOffsetY': 14}
ops, Z = [], [10]

def fs(px):                            # 렌더 px → fontSizeRatio (렌더 px = ratio × H² / 1080)
    return px / (H * H / 1080)

def add(name, typ, x, y, w, h, t0, t1, z=None, **kw):
    Z[0] += 1
    offx = offy = 0
    if x < 0: offx, x = x, 0           # 화면 밖 걸치기 = 상수 키프레임 오프셋
    if y < 0: offy, y = y, 0
    w, h = min(w, W), min(h, H)
    if offx or offy:
        m = dict(kw.get('motion') or {}); k = list(m.get('keyframes') or [])
        if offx: k.append({'timeMs': 0, 'property': 'x', 'value': offx})
        if offy: k.append({'timeMs': 0, 'property': 'y', 'value': offy})
        m['keyframes'] = k; kw['motion'] = m
    op = {'op': 'add_layer', 'format_idx': 0, 'layer_type': typ, 'name': name,
          'x': x / W, 'y': y / H, 'width': w / W, 'height': h / H,
          'zIndex': z if z is not None else Z[0], 'start_ms': int(t0), 'end_ms': int(t1)}
    op.update({k: v for k, v in kw.items() if v is not None})
    ops.append(op); return op

def rect(name, x, y, w, h, t0, t1, fill, r=0, border=None, bw=0, glow=False, opacity=None, fill_opacity=1, **kw):
    sh = {'shapeKind': 'rect', 'backgroundColor': fill, 'backgroundOpacity': fill_opacity,
          'borderRadius': r, 'borderWidth': bw, 'borderColor': border or '#ffffff'}
    if glow: sh['effects'] = GLOW
    return add(name, 'shape', x, y, w, h, t0, t1, shape=sh, opacity=opacity, **kw)

def text(name, x, y, w, h, t0, t1, s, px, color=INK, weight=800, align='center', **kw):
    tp = {'text': s, 'fontFamily': 'Pretendard', 'fontSizeRatio': fs(px), 'fontWeight': weight,
          'fontColor': color, 'textAlign': align, 'verticalAlign': 'middle', 'lineHeight': 1.3}
    tp.update(kw.pop('tp', {}))
    return add(name, 'text', x, y, w, h, t0, t1, text=tp, **kw)

def img(name, x, y, w, h, t0, t1, asset, fit='cover', **kw):
    return add(name, 'image', x, y, w, h, t0, t1, image={'url': asset['url'], 'asset_id': asset['id'], 'objectFit': fit}, **kw)

def en(t, ms, delay=0, easing='easeOut', dist=None):
    d = {'type': t, 'durationMs': ms, 'easing': easing}
    if delay: d['delayMs'] = delay
    if dist is not None: d['distancePx'] = dist
    return d

def kf(t, p, v, e=None):
    d = {'timeMs': int(t), 'property': p, 'value': v}
    if e: d['easing'] = e
    return d

def headline(tag, lines, y, t0, t1, px=96, gap=124, delay=0):
    for i, s in enumerate(lines):      # 회색이 먼저 올라오고 먹색이 차오른다
        dl = delay + i * 110
        text(f'{tag} 헤드 {i+1} 회색', 40, y + i * gap, W - 80, 130, t0, t1, s, px, color=GRAY,
             motion={'enter': en('slideUp', 320, dl, SNAP, 10), 'exit': en('fade', 200)})
        text(f'{tag} 헤드 {i+1} 먹', 40, y + i * gap, W - 80, 130, t0, t1, s, px, color=INK,
             motion={'enter': en('fade', 380, dl + 170), 'exit': en('fade', 200)})

def send(stage):
    global ops
    for i in range(0, len(ops), 20):
        r = jig.api(f'/auto-gen/internal/edit-drafts/{D}/batch', {'ops': ops[i:i + 20]})
        bad = [x for x in (r.get('results') or []) if not x.get('ok')]
        print(stage, i, 'applied', r.get('applied'), 'failed', r.get('failed'), json.dumps(bad, ensure_ascii=False)[:500])
    ops = []

def ids():
    d = jig.api(f'/auto-gen/internal/edit-drafts/{D}', method='GET')['draft']
    return {l['name']: l['id'] for l in d['formats'][0]['layers']}
```

## 조립은 두 번에 나눈다

1. **널·부모 frame 먼저** → `send('parents')` → `I = ids()`.
2. 자식: `attach_to = {'layer_id': I['CAM1'], 'position': True, 'scale': True, 'rotation': True}`, frame 자식은 `parent_id = I['버튼']`.

## 카메라 널 — 버튼으로 줌

```python
BX, BY, BW, BH = 680, 1560, 330, 84                 # 패널 안의 작은 버튼(줌 전 좌표)
bcx, bcy = BX + BW / 2, BY + BH / 2                 # 널은 버튼 중심에
rect('CAM1', bcx - 5, bcy - 5, 10, 10, 1300, 4950, '#ffffff', opacity=0, z=2, motion={'keyframes': [
    kf(0, 'scale', 1), kf(2400, 'scale', 1, 'linear'), kf(2750, 'scale', 2.8, ZOOM), kf(3650, 'scale', 2.86, 'linear'),
    kf(0, 'x', 0), kf(2400, 'x', 0, 'linear'), kf(2750, 'x', 540 - bcx, ZOOM),           # 버튼을 화면 (540, 900) 으로
    kf(0, 'y', 0), kf(2400, 'y', -30, 'linear'), kf(2750, 'y', 900 - bcy, ZOOM)]})       # 그 전엔 느린 드리프트
```

키프레임 시각은 **레이어 시작 기준**이다. 줌과 동시에 나머지 조각은 `exit fade 250` 으로 끝나고, 버튼만 다음 구간까지 남는다.

## 커서 — 곡선으로 들어와 누르고 나간다

```python
CUR = "M2 2 L2 80 L22 61 L37 97 L52 90 L37 55 L66 55 Z"
add('커서', 'shape', 700, 960, 62, 88, 4100, 4950, z=99,
    shape={'shapeKind': 'svg', 'svgPathData': CUR, 'svgViewBox': '0 0 70 100', 'backgroundColor': INK,
           'backgroundOpacity': 1, 'borderColor': '#ffffff', 'borderWidth': 5},
    motion={'enter': en('fade', 120), 'exit': en('fade', 150), 'keyframes': [
        kf(0, 'x', 420), kf(450, 'x', 0, SNAP), kf(680, 'x', 0), kf(850, 'x', 480, 'easeIn'),          # x 급감속
        kf(0, 'y', 620), kf(450, 'y', 0, 'easeInOut'), kf(680, 'y', 0), kf(850, 'y', 420, 'easeIn'),   # y 는 다른 이징 → 곡선
        kf(480, 'scale', 1), kf(540, 'scale', 0.84, 'easeOut'), kf(700, 'scale', 1, 'easeOutBack')]})  # 클릭 딥
```

눌리는 버튼에도 같은 시각에 `scale 1 → 0.93 → 1(easeOutBack)` 키프레임을 준다.

## 입력창 · 타이핑

```python
rect('입력창', 76, 880, 929, 300, a, b, '#ffffff', r=46, border=LINE, bw=3, glow=True, z=30,
     motion={'enter': en('slideUp', 480, 180, SNAP, 70), 'exit': en('fade', 220)})
for i, p in enumerate(thumbs):                                   # 첨부 썸네일 — 앞 장면의 재료가 이어진다
    img(f'첨부 {i}', 116 + i * 104, 916, 88, 88, a, b, p, z=31, motion={'enter': en('pop', 360, 420 + i * 90, 'spring')})
text('타이핑', 116, 1034, 740, 104, a, b, '이 사진들로 릴스 만들어줘', 48, weight=600, align='left', z=31,
     tp={'charAnimation': 'animator', 'animator': {'unit': 'char', 'order': 'forward', 'easing': 'linear',
         'delayMs': 950, 'opacity': 0, 'staggerMs': 75, 'durationMs': 40}})
```

## 자라는 결과 패널 (트랙)

트랙마다 아이콘 이미지 `pop` → 라벨 `slideUp 14px` → 내용 `wipeRight 240~900` → 체크 path `draw 300`. 재생 헤드는 폭 5px rect 에 `x 0 → 690 linear` 2.2초. 전부 같은 널에 붙여 널이 `y 30 → −40` 으로 천천히 민다.

## 폰 주변 기능 칩

frame(흰색·테두리·글로우, `borderRadius` 는 높이의 1/3 이하로 — 캡슐 금지) + 투명 spacer 자식 + text 자식. 아이콘 이미지는 **같은 카메라 널에** 절대 좌표로 붙이고 칩과 같은 `loop float` 을 준다. 화면 밖에 걸치는 칩은 x 를 음수로 주면 도우미가 오프셋 키프레임으로 바꾼다.
