# mono-slide-pdf skill

Claude Code用のSkillです。Googleドキュメント/Word/テキストなど文章主体の資料を、
モノクロ＋差し色1色のスライド風PDF（横長16:9／縦長9:16）に変換します。

## 使い方

このリポジトリを clone し、`skills/` 配下のフォルダをそれぞれ
Claude Code の Skills ディレクトリ（`~/.claude/skills/`）にコピーしてください。

```bash
git clone <このリポジトリのURL>
cp -r mono-slide-pdf-skill/skills/mono-slide-pdf ~/.claude/skills/
cp -r mono-slide-pdf-skill/skills/color-mono ~/.claude/skills/
```

- `mono-slide-pdf`: スライド風PDF変換の一連のフロー・テンプレート・ノウハウ
- `color-mono`: `mono-slide-pdf` が参照する配色定義（依存Skill。両方コピーしてください）

## 起動条件

「スライドっぽいPDFにして」「16:9のPDFで」「資料をスライドPDF化して」などと
Claude Codeに伝えると自動的にこのSkillが起動します。詳細は
[`skills/mono-slide-pdf/SKILL.md`](skills/mono-slide-pdf/SKILL.md) を参照してください。
