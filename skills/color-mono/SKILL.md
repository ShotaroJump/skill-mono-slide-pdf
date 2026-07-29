---
name: color-mono
description: >
  黒白グレーのモノクロームに、差し色をたった1色だけ効かせる配色Skill。
  「モノクロ＋差し色」「グレー背景に白カードを浮かせる配色」「Notion/Linear/Stripeっぽい
  スマートな見た目」「派手な色は1色だけでいい」「情報は密でいいから読みやすく」といった
  見た目の要望が出たら必ずこのSkillを読むこと。非エンジニア向けの解説PDF・ガイド資料・
  ダッシュボード・スライドなど、ブランドカラーの指定が無い成果物で「スマートで大人っぽい」
  「かわいすぎない」配色にしたいときに使う。HTML・reportlab(PDF)・PPTX・docx・SVGなど、
  成果物の形式を問わず参照すること。
---

# Mono Accent Color Skill（モノクロ＋差し色1色）

## 由来

宮城県データ活用ガイドPDF（国交省DPFスキルの解説資料）で確立した配色。ユーザーから
「色味はすごくいい」と高評価だったパターンなので、素の状態から毎回組み立て直さず
このSkillを起点にすること。

技術者向けの詳細仕様書や、ブランドカラーが別途指定されている案件には使わない。
その場合は `color-dads`（デジタル庁DADS準拠）や `color-brand`（K-SOCKETブランドカラー）
Skillを優先する。

## カラー定義

| 役割 | 名前 | HEX | 用途 |
|------|------|-----|------|
| ページ背景 | Mono Canvas | `#F1F0EC` | ページ・キャンバス全体の背景（温かみのある淡いグレー） |
| カード背景 | Mono Card | `#FFFFFF` | カード・パネル・吹き出しの背景 |
| 本文 | Mono Ink | `#17160F` | 見出し・本文（純黒でなく、わずかに温かい黒） |
| 説明文 | Mono Sub | `#66655D` | 二次テキスト・説明文 |
| 補足 | Mono Faint | `#9B9A90` | 出典・キャプション・補足情報 |
| 罫線 | Mono Line | `#E8E7E1` | カード内の区切り線・薄い境界線 |
| 差し色 | Accent（1色のみ） | `#FF3B1F` | バッジ・タグ・見出し番号・強調テキスト・区切りの点線 |
| 差し色（淡） | Accent Tint | `#FFEFEA` | 差し色の極薄トーン（吹き出し・ハイライト背景） |
| 影 | Shadow | `#000000`（不透明度 6%程度） | 白カードをグレー地から浮かせるドロップシャドウ |

## カラー導出ロジック

* Mono Canvas はニュートラルな真グレーではなく、わずかに黄みを足した温かいグレー。冷たい印象を避ける
* Mono Ink も同様に純黒(`#000000`)ではなく、わずかに温かい黒。硬すぎる印象を避ける
* Mono Sub / Mono Faint は Ink を薄めていった2段階の階調。色数を増やす代わりに、この3段階の濃淡
  （Ink→Sub→Faint）で情報の優先度を表現する
* Accent Tint は Accent を白に対して95%程度まで薄めたもの。背景やハイライトボックスに使い、
  Accentそのものは面積の小さい強調要素にしか使わない
* Shadowは黒の6%程度の不透明度。濃い影は安っぽく見えるので、あくまで「気づいたら浮いている」
  程度にとどめる

## 差し色の黄金ルール（最重要）

* **差し色は1資料につき1色だけ**。ジャンルごとに色分けしたくなるが、それをやった瞬間
  「かわいい/子供っぽい/カラフルな資料」になり、スマートさが失われる
* ジャンルや優先度を区別したい場合は、色を増やすのではなく Mono Ink / Mono Sub / Mono Faint の
  文字階調と、太字（Bold）の有無で表現する
* Accentを使ってよい場所：番号バッジ・タグ・見出し番号・強調したい数字やキーワード・区切りの点線
* Accentを使ってはいけない場所：本文の地の色、大面積のベタ塗り背景、複数箇所の同時多用
* 差し色が占める面積は1ページの5〜10%以内が目安（多くて15%）
* 案件のブランドカラーがある場合は Accent / Accent Tint だけを差し替えてよい。それ以外の
  グレー階調はそのまま流用できる

## レイアウトの原則

* **背景はグレー、要素は白**。ページ全体を Mono Canvas で塗り、内容は Mono Card の白いカードとして
  その上に浮かせる。白背景に白カード＋薄い罫線だけだと平坦になりやすいが、グレーの地に白を置くと
  自然に階層と奥行きが出る
* 白カードにはごく薄いドロップシャドウ（Shadow、1mm弱オフセット）を足すとさらに浮いて見える
* 情報量は詰めてよい。スカスカな1メッセージ1ページより、一覧できる密度の方がスマートさに繋がる。
  表形式の一覧（カタログ）は特にこの配色と相性が良い

## 各成果物への適用ルール

### HTML / CSS

```css
:root {
  --mono-canvas: #F1F0EC;
  --mono-card:   #FFFFFF;
  --mono-ink:    #17160F;
  --mono-sub:    #66655D;
  --mono-faint:  #9B9A90;
  --mono-line:   #E8E7E1;
  --mono-accent: #FF3B1F;      /* 1ページにつき1色のみ使用 */
  --mono-accent-tint: #FFEFEA;
}

body { background: var(--mono-canvas); color: var(--mono-ink); }
.card {
  background: var(--mono-card);
  border: 1px solid var(--mono-line);
  border-radius: 8px;
  box-shadow: 0 3px 8px rgba(0,0,0,0.06);
}
.badge { background: var(--mono-accent); color: #fff; }
.tint-box { background: var(--mono-accent-tint); }
```

### React / Tailwind（インライン）

```jsx
const mono = {
  canvas: "#F1F0EC",
  card: "#FFFFFF",
  ink: "#17160F",
  sub: "#66655D",
  faint: "#9B9A90",
  line: "#E8E7E1",
  accent: "#FF3B1F",      // 1ページ1色まで
  accentTint: "#FFEFEA",
};
```

### SVG

```svg
<rect fill="#F1F0EC" />                    <!-- キャンバス -->
<rect fill="#FFFFFF" stroke="#E8E7E1" />   <!-- カード -->
<circle fill="#FF3B1F" />                  <!-- 差し色バッジ -->
```

### PPTX（python-pptx）

```python
from pptx.dml.color import RGBColor

MONO_CANVAS = RGBColor(0xF1, 0xF0, 0xEC)
MONO_CARD   = RGBColor(0xFF, 0xFF, 0xFF)
MONO_INK    = RGBColor(0x17, 0x16, 0x0F)
MONO_SUB    = RGBColor(0x66, 0x65, 0x5D)
MONO_FAINT  = RGBColor(0x9B, 0x9A, 0x90)
MONO_LINE   = RGBColor(0xE8, 0xE7, 0xE1)
MONO_ACCENT = RGBColor(0xFF, 0x3B, 0x1F)  # 1資料1色まで
```

### Word / docx（python-docx）

```python
from docx.shared import RGBColor

MONO_INK    = RGBColor(0x17, 0x16, 0x0F)
MONO_SUB    = RGBColor(0x66, 0x65, 0x5D)
MONO_ACCENT = RGBColor(0xFF, 0x3B, 0x1F)  # 1資料1色まで
```

### reportlab（Python / PDF）

このパレットが生まれたのはreportlabでのPDF生成。フォントはHeiseiKakuGo-W5など
組み込みCIDフォントだと1ウェイトしかなく強弱がつかないので、Yu Gothic（Regular/Bold）
などTTFontを登録して使うとモダンに見える。

```python
import os
from reportlab.lib import colors
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont

FBASE = r'C:\Windows\Fonts'
pdfmetrics.registerFont(TTFont('YUR', os.path.join(FBASE, 'YuGothR.ttc')))
pdfmetrics.registerFont(TTFont('YUB', os.path.join(FBASE, 'YuGothB.ttc')))
# 無ければ BIZ-UDGothicR.ttc / BIZ-UDGothicB.ttc で代替

PAGE_BG = colors.HexColor('#F1F0EC')
CARD    = colors.white
INK     = colors.HexColor('#17160F')
SUB     = colors.HexColor('#66655D')
FAINT   = colors.HexColor('#9B9A90')
LINE    = colors.HexColor('#E8E7E1')
ACCENT  = colors.HexColor('#FF3B1F')
ACCENT_TINT = colors.HexColor('#FFEFEA')

# ページ全体をグレーに塗るのは SimpleDocTemplate の onFirstPage/onLaterPages で行う
# (flowableの描画より先に呼ばれるため、これで全ページグレー地の上にカードが乗る)
def page_background(canvas, doc):
    canvas.saveState()
    canvas.setFillColor(PAGE_BG)
    canvas.rect(0, 0, canvas._pagesize[0], canvas._pagesize[1], fill=1, stroke=0)
    canvas.restoreState()

# 白カード + 薄いドロップシャドウ
def card_bg(c, x, y, w, h, r=4):
    c.saveState()
    c.setFillColor(colors.black)
    c.setFillAlpha(0.06)
    c.roundRect(x + 2.5, y - 2.5, w, h, r, fill=1, stroke=0)
    c.restoreState()
    c.setFillColor(CARD)
    c.roundRect(x, y, w, h, r, fill=1, stroke=0)
```

## 禁則

* 差し色を2色以上、同じページ・同じ資料内で併用しない
* ジャンル別に色分けしない（色の代わりに文字階調・太字で優先度を表現する）
* 濃い影・派手なグラデーションを足さない（このパレットの良さは階調の静けさにある）
* 白背景の上に白カード＋線だけを置いて済ませない（必ずグレーのキャンバスの上に白を浮かせる）

## チェックリスト

- [ ] ページ背景はグレー(`#F1F0EC`)、カード・要素は白になっているか
- [ ] 差し色は1色だけに絞られているか（ジャンル別の色分けをしていないか）
- [ ] 白カードにごく薄いドロップシャドウがついているか
- [ ] 情報の優先度は色数ではなく Ink/Sub/Faint の3階調で表現されているか
- [ ] 差し色の使用面積は1ページの15%以内に収まっているか
