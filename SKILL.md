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
> 1. テキストが不自然に折り返されていないか
> 2. テキスト同士、テキストと画像の重なりはないか
> 3. 余白の偏り・過不足はないか
> 4. フォントサイズが小さすぎて判読困難でないか
> 5. 画像のアスペクト比が歪んでいないか
> 6. 連続スライド間のレイアウト一貫性は保たれているか
> 7. 背景とテキストのコントラストは十分か

### Step 3: 問題レポート生成

[review-report-template](assets/review-report-template.md) に従い、スライド番号ごとに問題を整理する。
各問題には [severity-criteria](references/severity-criteria.md) に基づくseverityを付与する。

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

## Self-Improvement

- **Feedback Capture**: ユーザーFBやレビューで検出された問題を汎用的な検査観点に抽象化する（Step 6で実装済み）
- **Checklist Evolution**: 抽出した観点を [slide-visual-checklist](references/slide-visual-checklist.md) に追記する
- **Trigger**: テキスト折り返し問題、要素の被り、余白バランス崩れ、コントラスト不足

## Cross-Skill Integration

- **← marp**: Marpスライドのビジュアル検証工程を本スキルに委譲
- **← pptx**: PPTX生成後のビジュアルQAに組み込み
- **→ svg-review**: スライド内SVG図解の品質はsvg-reviewに委譲
