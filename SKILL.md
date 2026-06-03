---
name: slide-visual-reviewer
description: "スライドPDF/PNGのビジュアル品質を検証する。テキスト折り返し・要素の被り・余白の過不足を検出する。marp/pptxスキルのQA工程から呼び出される"
user-invocable: false
allowed-tools: Read, Bash, Glob, Grep
---

# slide-visual-reviewer

スライドのビジュアル品質をPNG画像の視覚検査で検証するサブスキル。
marpスキルやpptxスキルのQA工程から呼び出されるほか、スライド単体の品質改善にも使用する。

## Workflow

### Step 1: PNG生成

対象スライドの種類に応じてPNGを生成する。

**Marpスライド**:

```bash
node scripts/publish/marp-validate.js <slide-dir> --png
```

**PPTX**:

```bash
# LibreOfficeでページ単位PNG変換
soffice --headless --convert-to png --outdir <output-dir> <input.pptx>
```

生成されたPNGファイルのパスを確認し、全スライドを対象とする。

### Step 2: 全スライドPNGの視覚検査

各PNGをRead toolで読み込み、[slide-visual-checklist](references/slide-visual-checklist.md) の観点で検査する。

**検査の姿勢**: 問題がある前提で検査する。「良さそう」ではなく「何が問題か」を探す。

検査プロンプト:

> このスライドPNGを検査する。以下の観点で**問題点を列挙**する:
>
> 1. テキストが不自然に折り返されていないか（CJK禁則含む）
> 2. テキスト同士、テキストと画像の重なりはないか
> 3. 余白の偏り・過不足はないか
> 4. フォントサイズが小さすぎて判読困難でないか
> 5. 画像のアスペクト比が歪んでいないか
> 6. 連続スライド間のレイアウト一貫性は保たれているか
> 7. 背景とテキストのコントラストは十分か（4.5:1 / 大文字3:1・四捨五入しない、図形は1.4.11）
> 8. 色だけで区別していないか（WCAG 1.4.1）
> 9. チャートがミスリードしていないか（基線0・単位・Lie Factor）
> 10. ~3秒のグランステスト（視覚の焦点が1つか）

### Step 3: 問題レポート生成

[review-report-template](assets/review-report-template.md) に従い、スライド番号ごとに問題を整理する。
各問題には [severity-criteria](references/severity-criteria.md) に基づくseverityを付与し、**スライド番号＋要素＋観察状態**を引用する。

**リリースゲート（最悪値）**: error が1つでもあれば **FAIL（発表不可）**、error=0でwarningあり→直してから、infoのみ→PASS。平均的な集計はしない（severityの定義上、error は他スライドで相殺できないため）。

### Step 4: 修正提案

検出した問題に対して [common-visual-fixes](references/common-visual-fixes.md) に基づく修正案を提示する。

**制約緩和の確認**: 修正が現在のレイアウト内で収まらない場合、「フォントサイズの縮小」「要素の再配置」「スライド分割」などの制約緩和をユーザーに提案してから実施する。

### Step 5: 修正後の再検証

修正適用後にStep 1-2を再実行し、問題が解消されたことを確認する。

**無限ループ防止**: 再検証で新たな問題が見つかった場合、即座に修正せず、問題一覧と修正方針をユーザーに提示して承認を得てから対応する。

### Step 6: 指摘の抽象化とチェックリスト更新

ユーザーフィードバックやレビューで検出された問題を、具体的な指摘から汎用的な検査観点に抽象化し、[slide-visual-checklist](references/slide-visual-checklist.md) に追加する。

例:

- 具体: 「3枚目のタイトルが2行に折り返されている」→ 抽象: 「タイトルテキストがスライド幅に収まっているか」
- 具体: 「箇条書きとフッター画像が重なっている」→ 抽象: 「コンテンツ領域とフッター/ヘッダー領域が干渉していないか」

## SVGを含むスライドの検査

スライド内にSVG図解が含まれる場合、SVG単体の品質はsvg-reviewスキルの管轄となる。
本スキルではスライド上でのSVGの表示結果（サイズ、配置、周囲要素とのバランス）を検査する。

## Reference

- [slide-visual-checklist](references/slide-visual-checklist.md) -- 検査観点の詳細チェックリスト
- [common-visual-fixes](references/common-visual-fixes.md) -- 頻出問題の修正パターン集
- [qa-checklist](references/qa-checklist.md) -- 品質チェックリスト
- [severity-criteria](references/severity-criteria.md) -- error/warning/infoの判定基準

## Programmatic Usage

```bash
# 単体実行
node scripts/maintain/skill-runner.js slide-visual-reviewer "content/publish/slides/example/"

# プロンプト確認のみ（dry-run）
node scripts/maintain/skill-runner.js slide-visual-reviewer "content/publish/slides/example/" --dry-run
```

## 自己改善ループ（指摘 → ルール → 横展開 → 再発防止）

指摘をその場の修正で終わらせない:

1. **判断**: 指摘が「一般化できるパターン」か「その場限りの好み」か。**パターンだけルール化**する（肥大化防止）。
2. **ルール更新**: パターンなら [slide-visual-checklist](references/slide-visual-checklist.md) / [common-visual-fixes](references/common-visual-fixes.md) を抽象化して更新（Step 6）。
3. **横展開（全スライド再走査）**: 更新したルールで**全スライドを再検査**し、同種の箇所をすべて直す（silent な見逃し防止）。
4. **再発防止**: 成果物側の問題は通常どおり是正。**レビュアー自身の誤り**（指摘の取り違え・誤判定）は下の「よくある誤読・落とし穴」に記録し次回最初に確認。
- **Trigger**: テキスト折り返し（→/CJK禁則）、要素の被り、余白/下方の張り付き、コントラスト、色のみ依存、チャートの誠実さ。

## よくある誤読・落とし穴（次回まずチェック）

- **「下に行きすぎ」≠「下の余白を埋める」**: 赤文字Punch等が下に離れている指摘は、本文の直下に**上詰め**せよの意味。下側に余白が残るのは問題ない（埋めにいかない）。タイトル位置は固定。
- **隅の画像のキャプション忘れ**: 右上等の小画像には何の画像か分かるキャプションを付ける（大きく見せたいなら半透明 0.2〜0.3 で背景化でも可）。
- **絶対配置の結論が本文に重なる**: Punch の `bottom` を上げすぎると本文に被る。確実に本文直下へ置きたいなら、絶対配置でなくフロー配置にする。

## Cross-Skill Integration

- **← marp**: Marpスライドのビジュアル検証工程を本スキルに委譲
- **← pptx**: PPTX生成後のビジュアルQAに組み込み
- **→ svg-review**: スライド内SVG図解の品質はsvg-reviewに委譲
