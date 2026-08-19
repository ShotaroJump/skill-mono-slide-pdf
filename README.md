# mono-slide-pdf skill

Claude Code用のSkillです。Googleドキュメント/Word/テキストなど文章主体の資料を、
モノクロ＋差し色1色のスライド風PDF（横長16:9／縦長9:16）に変換します。

## セットアップ

このリポジトリを clone し、`skills/` 配下のフォルダをそれぞれ
Claude Code の Skills ディレクトリ（`~/.claude/skills/`）にコピーしてください。

```bash
git clone https://github.com/ShotaroJump/skill-mono-slide-pdf.git
cp -r skill-mono-slide-pdf/skills/mono-slide-pdf ~/.claude/skills/
cp -r skill-mono-slide-pdf/skills/color-mono ~/.claude/skills/
```

- `mono-slide-pdf`: スライド風PDF変換の一連のフロー・テンプレート・ノウハウ
- `color-mono`: `mono-slide-pdf` が参照する配色定義（依存Skill。両方コピーしてください）

配置後、Claude Code のセッションを開き直すと Skill 一覧に表示されます。

## 前提環境

- Google Chrome（ヘッドレスモードでPDF化するために使用。wkhtmltopdf・weasyprint等は
  日本語フォント・CSS Gridとの相性が悪いため使わない）
- Python + [`pypdfium2`](https://pypi.org/project/pypdfium2/)（生成したPDFをページごとに
  画像化して目視確認するために使用）

```bash
pip install pypdfium2
```

## 起動条件

「スライドっぽいPDFにして」「16:9のPDFで」「資料をスライドPDF化して」などと
Claude Codeに伝えると自動的にこのSkillが起動します。詳細は
[`skills/mono-slide-pdf/SKILL.md`](skills/mono-slide-pdf/SKILL.md) を参照してください。

## 構成

```
skills/mono-slide-pdf/SKILL.md                        Skill本体（起動条件・作業フロー・レイアウト原則）
skills/mono-slide-pdf/template-landscape.html          横長16:9テンプレート（画面投影用）
skills/mono-slide-pdf/template-portrait.html            縦長9:16テンプレート（スライド分割型）
skills/mono-slide-pdf/template-portrait-doc.html        縦長9:16テンプレート（ドキュメント流し込み型）
skills/mono-slide-pdf/references/縦長レイアウトパターン.md  縦長3パターンの選び方・実装詳細
skills/mono-slide-pdf/references/アイコン素材調達.md        絵文字を使わないアイコン・イラスト調達方法
skills/color-mono/SKILL.md                              依存Skill（モノクロ＋差し色の配色定義）
```
