# Mono Slide PDF（モノクロ＋差し色 スライド風PDF化）

## 由来

Googleチャット・カレンダー社内活用ガイド（260729_第1回会議資料）を、画面投影用の
横長16:9スライド風PDFに変換したセッションで確立したデザイン・実装パターン。
ユーザーから「めちゃくちゃ良かった」と高評価だったため、毎回ゼロから組み立て直さず
このSkillを起点にすること。

配色は [`color-mono`](../color-mono/SKILL.md) をそのまま使う。color-monoは配色ルールのみを
定義するSkillで、こちらはその配色を使って「元ドキュメントをスライド風PDFに変換する」
一連の作業フロー（構成・図解化・実装・PDF化・検証）を定義する。

## 起動条件

- 「スライドっぽいPDFにして」「16:9のPDFで」「スライド形式のPDFにして」
  「資料をスライドPDF化して」「/color-mono でPDFにして」
- Googleドキュメント/Word/テキストなど文章主体のドキュメントを、画面投影・配布・
  一枚読み物用のPDFに変換したいとき
- 縦長（配布・印刷・スマホ閲覧向け）を明示されたときも同じ配色・フォントで対応する

技術者向けの詳細仕様書や、DADS/ブランドカラー指定がある案件には使わない
（その場合は `color-dads` / `color-brand` を使う `dads-html` 等に譲る）。

## 全体フロー

1. **元ドキュメントを全文取得する**（Google Docs MCP の `read_file_content` 等）。
   要約・圧縮はしない。ユーザーが「かなり練って作った」ドキュメントであることが多く、
   再要約は望まれない。誤字脱字だけ適宜修正してよい。
2. **向きを決める**。ユーザー指定があればそれに従う。無ければ用途から判断する：
   - 画面投影・会議で映す → **横長 landscape**（`template-landscape.html`, 1280×720 / 16:9）。
     「スライド」らしく、1トピック1ページ・余白にもゆとりを持たせる。
   - 配布資料・印刷・スマホでの一枚読み物 → **縦長 portrait**（720×1280 / 9:16）。
     縦長にはさらに2つのパターンがあるので、資料の性格で選ぶ（詳細は下記
     「4. 縦長(portrait)の2パターン」）：
     - **スライド分割型**（`template-portrait.html`）：見出し単位で1ページ1トピックに
       区切る。ページ数が資料の構成そのものを表す資料（章立てを明示したい提案書等）向け。
     - **ドキュメント流し込み型**（`template-portrait-doc.html`）：docx/Wordのように
       本文を上から連続して流し込み、ページはみ出した分だけ自然に変わる。「ページで
       区切らないでほしい」「下の余白が気持ち悪い」と言われたらこちら。カタログ・
       レポート・ガイドなど文書然とした資料のデフォルトはこちらを優先してよい。
       さらに、**1〜2ページ程度で収まりそうな資料なら、複数ページに分けず
       コンテンツ高さぴったりの単一ページにする**（詳細は下記「4. パターンB'」）。
       最終ページの余白が気になる／気にされそうな場合はこちらを優先する。
3. **該当するテンプレート（`template-landscape.html` / `template-portrait.html` /
   `template-portrait-doc.html`）を作業用にコピー**して、スクラッチパッドで編集する
   （元テンプレートは直接壊さない）。
4. **見出し単位で1スライド1トピックに分割**する。テンプレート内のコンポーネント
   （下記「コンポーネント一覧」）をコピペしながら、元ドキュメントの構造に当てはめる。
   - 箇条書きが並ぶだけの内容 → `card` グリッド（`grid2`/`grid3`/`grid4`）
   - 時系列の経緯 → `timeline`（横長）/ `timeline-v`（縦長）
   - 現状→対応の対比 → `ba-wrap`（横長）/ `ba-stack`（縦長）
   - プロセス・フロー → `flow`（横長）/ `flow-v`（縦長）
   - 「これだけは伝えたい」という核心メッセージ → `callout-strong`
   - Before/Afterのマッピング（A→Bの変換規則など） → `mapping-row`
   - 補足・宿題・参考資料など優先度の低いページは**デッキの一番最後**に回す
5. **文字は大きめ・余白は自然に埋める**（詳細は下記「レイアウトの原則」）。
6. **PDF化する前に、ブラウザ上のJSではみ出しを自動チェック**する（詳細は下記
   「2. はみ出しは目視の前にJSで自動検知する」）。HTMLを直接ブラウザプレビューで開き、
   `javascript_tool` で全 `.slide` の overflow を一括判定 → 引っかかった要素だけ
   HTMLを直して再チェック、を **overflow ゼロになるまで**回す。この段階では
   PDF化・画像化はまだ行わない（トークン消費の大きい工程を後段に温存する）。
7. **overflowゼロを確認できた時点で、一度ユーザーに見せて承認を取る。** ここでPDF化まで
   自動で進めない。完成したHTMLファイルをそのまま渡す（`SendUserFile`）か、ブラウザ
   プレビューで見せて、内容・構成にOKが出てから次に進む。PDF化（Chromeヘッドレス実行・
   `pypdfium2`での画像化）は後段の重い処理で、承認前にそこまで進めると内容修正が
   入った際の手戻りが大きいため。「ユーザーの承認が無い限りはHTMLで止めてほしい」と
   明示されたことがある。
8. 承認が得られたら、**Chromeヘッドレスで PDF 化**し、`pypdfium2` で
   ページを画像化して**最終確認として1回だけ目視**する（レイアウト崩れはJSで
   拾えるが、文字の視覚的な詰まり具合や配色バランスはJSでは判断できないため）。
   ここで問題が見つかった場合のみ、6に戻って再度JSチェック→PDF化を繰り返す。
9. 完成したPDFを `SendUserFile` でユーザーに渡す。

## レイアウトの原則（このSkill最大の学び）

各項目は「こうする」という肯定形で書いてある。応用が効くよう、まずこの形のまま
適用し、個別ケースでのズレは項目末尾の理由を手がかりに微調整する。

### 1. 文字は大きめに、余白は自然に埋める

- 本文は16〜18px以上、見出しは19〜24px以上を使う（最低ラインは14px。ラベルタグ
  のみ例外で下記参照）。
- `.slide`はflex-column、ヘッダー（eyebrow/title/subtitle/rule）は固定サイズにし、
  `.content`を`flex:1`にして残り高さを埋めさせる。
- 内容が薄いスライドは`.content.center`でブロックごと縦中央に寄せ、上下に均等な
  余白を作る。
- 内容が濃いスライド（4象限カード等）は`grid3`/`grid4`を`flex:1`の`.content`に
  そのまま入れて高さいっぱいに伸ばし、カード内は上詰め（`justify-content:flex-start`）
  にする。
- 3ブロック以上を1枚に収める場合は`.content.stack`（`space-evenly`）を使い、
  ブロック間の余白を均等配分する。
- 短い一言だけのボックス（「入力→処理→出力」の入力・出力側等）は、親コンテナに
  `justify-content:center`を使い、ボックス自体は`flex:1`を付けず内容量なりの
  高さで中央に浮かせる。箱の大きさは中の文字量に合わせる。
- ラベルタグ（`.eyebrow`・`.footer`等）だけ14px未満を許容し、`--mono-sub`/
  `--mono-faint`で薄い色にして「本文ではなく補足情報」だと示す。

### 2. はみ出しはPDF化の前にJSで検知する

- はみ出しが出たら、padding（`22px 28px`目安）→line-height（1.55〜1.65目安）
  →ヘッダー余白（`.rule`のmargin-bottom等）の順に詰めて対処する。
- HTMLをブラウザプレビューで開き、`javascript_tool`で以下を実行して
  `.card`/`.content`/`.callout-strong`/`.flow-box`のoverflowを一括検知する。
  空配列になるまでHTML修正→再実行を繰り返す（画像を作らないので何周でも軽い）。

  ```javascript
  Array.from(document.querySelectorAll('.slide')).map((slide, i) => {
    const overflowing = [];
    slide.querySelectorAll('.card, .content, .callout-strong, .flow-box').forEach(el => {
      if (el.scrollHeight > el.clientHeight + 1) overflowing.push(el.className);
    });
    return { page: i + 1, overflowing };
  }).filter(r => r.overflowing.length > 0);
  ```

- overflowゼロを確認できたら、PDF化＋`pypdfium2`画像化で最終目視を1回行う
  （文字の詰まり具合・余白バランス・配色の見え方はJSでは判定できないため）。

### 3. 差し色は1資料1色、強調は太めの2pxボーダーで統一する

- 差し色（`--mono-accent`）は、「ここだけ強調したい」ボックス
  （`.card-highlight`/`.flow-box.flow-highlight`/`.callout-strong`）と、
  eyebrow・タグなど資料全体で共通の小さな装飾要素にだけ使う。
- カタログ・一覧の連番（`.cat-num`）や手順バッジ（`.badge-num`）は既定で
  `--mono-ink`（黒系）にし、`.card-highlight`の中にあるときだけ赤くする
  （`.card-highlight .badge-num { background: var(--mono-accent); }`）。
  `.cat-num`は`--mono-faint`（薄いグレー）にする。項目が多い一覧を全部赤に
  すると本当に注意すべき箇所が埋もれる、という実フィードバックに基づく。
- `.card-highlight`は「見逃すと読み手が損する/誤解するリスク・警告・結論」に
  使う。迷ったら「これが無いと読み手が損をする/誤解する情報か？」で判定する。
- 強調ボックスは全て「background: accent-tint + border: 2px solid accent」で
  統一する。
- 例外：GASの自動色分け例のように実際のプロダクト機能の色（Googleカレンダーの
  ラベル色等）を説明する図解は、事実を示す色としてそのまま使ってよい
  （`color-dot`参照）。

### 4. 縦長(portrait)は資料の性格でパターンを選ぶ

縦長にはA：スライド分割型（`template-portrait.html`）／B：ドキュメント流し込み型
（`template-portrait-doc.html`）／B'：1〜2ページで収まる場合の単一ページ化、の
3パターンがある。選び方は「全体フロー」の2番を使う。実装の詳細（CSS・改ページ
制御・単一ページ化の実測手順）は `references/縦長レイアウトパターン.md` にある。
**縦長を作る前に必ずこのファイルを読む**（特にB'は目視だけでは再現できない）。

### 5. ページは優先度順に並べる

- 宿題・補足資料・参考リンクなど本編ではない情報は、デッキの一番最後に置く。
- 「①②③…」と番号立てできる内容は、その番号を`badge-num`として視覚化すると
  探しやすくなる。
- ある概念に「もっと詳しく知りたくなる」補足が必要なときは、独立した1ページを
  本編の直後に挿入し、本編側のカードから`→ 次ページで詳しく解説`のようにリンク
  させ、デッキ全体の流れを保つ。

### 6. 表紙はページ番号にカウントしない

- `.footer`の連番は、表紙を除いた本編の1枚目を「1」として数える。表紙の
  `.footer`は`<span>発行元</span><span></span>`のように右側を空にする
  （`template-landscape.html`/`template-portrait.html`の1枚目を参照）。
- 対象はスライド分割型のみ。ドキュメント流し込み型（`template-portrait-doc.html`）
  はページごとの`.footer`を持たないため対象外。
- 区切りページ（`.divider-wrap`）は本編として通常どおり連番に含める。除外するのは
  表紙1枚だけ。

## コンポーネント一覧

`template-landscape.html` / `template-portrait.html` に、以下すべての実装例が
入っている。新しいスライドを作るときは、該当ブロックをコピーして中身を差し替える。
`template-portrait-doc.html`（ドキュメント流し込み型）はこれらのコンポーネントの
うち `.card` / `.callout-strong` / `.cat-row` 等の**カード系だけ**を流用し、
`.slide` / `.content` / `grid2〜4` のようなページ単位のレイアウト要素は使わない
（本文がそのまま連続して流れるため）。

| 用途 | クラス（横長） | クラス（縦長） |
|---|---|---|
| ページ全体 | `.slide` | `.slide` |
| 見出し帯 | `.eyebrow` + `.title` + `.subtitle` + `.rule` | 同じ |
| 残り高さを埋める領域 | `.content` (+ `.center` / `.stack`) | 同じ |
| カード | `.card`（+ `.card-center` / `.card-highlight`） | 同じ |
| カードグリッド | `.grid2` / `.grid3` / `.grid4` | `.grid1`（縦積み）/ `.grid2` |
| 番号バッジ | `.badge-num` | 同じ |
| 箇条書き | `ul.plain`（`li.sub` で二階層目） | 同じ |
| セクション区切りページ | `.divider-wrap` + `.divider-num` | 同じ（数字は小さめ） |
| 時系列 | `.timeline` / `.tl-item` / `.tl-dot` | `.timeline-v` / `.tlv-item` / `.tlv-dot`（縦並び） |
| 現状→対応の対比 | `.ba-wrap` / `.ba-box` / `.ba-arrow`（→） | `.ba-stack` / `.ba-box` / `.ba-arrow-v`（↓） |
| プロセスフロー | `.flow` / `.flow-box` / `.flow-arrow`（→） | `.flow-v` / `.flow-box` / `.flow-arrow-v`（↓） |
| 核心メッセージの強調ボックス | `.callout-strong` | 同じ |
| A→Bのマッピング図 | `.mapping-row` / `.bracket-tag` / `.map-arrow` / `.color-badge` | 同じ |
| Key-Valueの説明行 | `.kv-row` / `.kv-label` / `.kv-val` | 同じ |
| フッター（ページ番号） | `.footer` | 同じ |

縦長で `grid3`/`grid4` に相当する内容（3〜4カードの横並び）が出てきた場合は、
無理に詰め込まず `.grid1`（1カラム縦積み）か `.grid2`（2カラム）で構成し直す。
横長のまま流用すると文字が窮屈になりやすい。

## アイコン・イラスト素材の調達（絵文字を使わない）

`case-icon` やバッジ、tint-boxの先頭記号などに絵文字（📄✅💡など）をそのまま
使うと、デフォルトの色（黄色い月、緑のチェック等）がmono配色の「差し色1色」
ルールと衝突し、資料全体が安っぽく見える。**絵文字は使わず、必ず外部アイコン
素材に置き換える。** ユーザーから「絵文字はダサさが出る」と明示的な指摘を受けて
確立した方針。

調達方法（ブランドロゴ：LobeHub/svgl.app、汎用アイコン：Iconify、場面イラスト：
Loose Drawing）と使い方の原則は `references/アイコン素材調達.md` を参照する
（アイコン・イラストを使うスライドを作るときだけ読めばよい）。タイトル・区切り
ページ・まとめページなど文字量が少なく余白が余るスライドでは、積極的にLoose
Drawingのイラストを検討する。

## カラー定義（color-monoを継承）

```css
:root {
  --mono-canvas: #F1F0EC;
  --mono-card:   #FFFFFF;
  --mono-ink:    #17160F;
  --mono-sub:    #66655D;
  --mono-faint:  #9B9A90;
  --mono-line:   #E8E7E1;
  --mono-accent: #FF3B1F;      /* 1資料1色。バッジ・強調枠・矢印のみ */
  --mono-accent-tint: #FFEFEA;
}
```

案件のブランドカラーが別途指定された場合は `--mono-accent` / `--mono-accent-tint`
だけを差し替える（`color-mono` 参照）。グレー階調はそのまま流用してよい。

## PDF化コマンド（Windows / Chrome）

HTMLファイルをスライド風に組んだら、Chromeヘッドレスで直接PDF化する。
wkhtmltopdf・weasyprint等は使わない（日本語フォント・CSS Grid対応が弱いため）。

```bash
"/c/Program Files/Google/Chrome/Application/chrome.exe" --headless --disable-gpu \
  --no-pdf-header-footer --print-to-pdf="出力先.pdf" --print-to-pdf-no-header \
  --run-all-compositor-stages-before-draw --virtual-time-budget=10000 \
  "file:///絶対パス/スライド.html"
```

- `@page { size: 1280px 720px; margin: 0; }`（横長）/ `size: 720px 1280px;`（縦長）を
  HTML側のCSSに必ず入れておくこと。これがそのままPDFのページサイズになる。
- 出力先パスに日本語ユーザー名等が含まれると `ERROR ... アクセスが拒否されました`
  で失敗することがある。その場合は `C:\temp\` のような単純なパスに一旦出力し、
  最後に目的のファイル名へコピーする。

## PDF検証コマンド（必須）

生成したPDFは必ず画像化して目視確認する。文字のはみ出し・余白の偏りはテキスト
情報だけでは分からない。

```bash
C:/Python314/python.exe -c "
import pypdfium2 as pdfium
pdf = pdfium.PdfDocument('出力先.pdf')
print('pages:', len(pdf))
for i in range(len(pdf)):
    page = pdf[i]
    bitmap = page.render(scale=1.3)
    bitmap.to_pil().save(f'check_p{i+1:02d}.png')
"
```

保存した `check_pXX.png` を `Read` ツールで開いて確認する。特に：
- カード内の文章がカードの下端・スライド下端をはみ出していないか
- 情報量が薄いページで不自然に上寄せ/中央寄せになっていないか
- ヘッダーとフッターが重なっていないか（`.footer` は `position:absolute` なので
  コンテンツが伸びすぎると重なる）

## 完成後の受け渡し

- ファイル名は `<ドキュメント名>.pdf` など内容が分かる名前にする。
- `SendUserFile` で `status: "normal"` として渡す。何を強調したか・どう構成したかを
  一言で添える（詳しい変更ログは不要、要点だけ）。
