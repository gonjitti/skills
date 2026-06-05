# HTML レポート形式

アーキテクチャレビューは、OSの一時ディレクトリ内に、単一の自己完結型HTMLファイルとしてレンダリングされます。Tailwind CSSとMermaidはどちらもCDNから読み込みます。Mermaidはグラフ形状の図（依存関係、コールフローなど）を確実に処理するのに適しており、手作りのdivやインラインSVGは、より編集性の高い視覚表現（質量図、断面図など）に適しています。これら2つを組み合わせ、すべての図画をMermaidだけに頼らないようにしてください（単調な見栄えになるのを防ぐためです）。

## 基本構造（骨組み）

```html
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <title>アーキテクチャレビュー — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* Tailwindで綺麗にカバーできない小さなカスタムスタイルレイヤー:
         シームを表す破線、手描き風の矢印の頭など */
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## ヘッダー（Header）

リポジトリ名、日付、および簡潔な凡例を表示します（実線のボックス ＝ モジュール、破線 ＝ シーム、赤の矢印 ＝ リーク/漏洩、太い暗色のボックス ＝ 深いモジュール）。前置きの段落は省き、すぐに提案候補の表示に入ります。

## 候補カード（Candidate card）

図が主役です。説明文は簡潔かつ平易にし、用語集の言葉（[LANGUAGE.md](LANGUAGE.md)）をごく自然に使用します。

各候補は1つの `<article>` タグとして構成します：

- **タイトル** — 短く、モジュールを深くする内容を名付けます（例: "Collapse the Order intake pipeline" ＝ 「Order受付パイプラインの統合」）。
- **バッジ行** — 推奨度（`Strong` ＝ エメラルド、`Worth exploring` ＝ アンバー、`Speculative` ＝ スレート）、および依存関係カテゴリタグ（`in-process`、`local-substitutable`、`ports & adapters`、`mock`）。
- **ファイル** — 等幅フォントのリスト、`font-mono text-sm`。
- **Before / After 図** — 最大の見せ場です。2つのカラムに左右並列で配置します。パターンは以下を参照。
- **問題点** — 1文で記述。何が摩擦を引き起こしているか。
- **解決策** — 1文で記述。何が変更されるか。
- **メリット（Wins）** — 箇条書きで、各項目は6語/10文字程度。例: 「テストを1つのインターフェースに集約」、「価格設定ロジックのリーク停止」、「4つの浅いラッパーの削除」。
- **ADRへの言及**（該当する場合） — アンバー色のボックスに1行で記載。

長々とした説明段落は不要です。図を理解するために段落での説明が必要な場合は、図を描き直してください。

## 図のパターン

候補に合わせて適切なパターンを選択し、組み合わせてください。すべての図が同じ見栄えにならないように、多様性を持たせることが重要です。

### Mermaid グラフ（依存関係やコールフローの定番）

「XがYを呼び出し、YがZを呼び出しており、全体が複雑に絡み合っている」ことを示すには、Mermaidの `flowchart` または `graph` を使用します。全体が浮いて見えないように、Tailwindで装飾したカードで囲みます。`classDef` を使用して、リークしている矢印を赤色に、深いモジュールを暗色にスタイル指定します。シーケンス図は、「Before: 6回の往復、After: 1回の往復」を表現するのに適しています。

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.leak.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```

### 手作りのボックスと矢印（Mermaidの自動レイアウトが崩れる場合）

モジュールを `<div>` タグ（枠線とラベル付き）で表現します。矢印は、相対配置（`relative`）されたコンテナの上に、絶対配置（`absolute`）されたインラインSVGの `<line>` または `<path>` 要素として重ねます。Mermaidでは表現しきれない「枠線の太い1つの深いモジュールの内部に、グレーアウトされた内包物が存在するAfterの状態」を描く場合などに使用します。

### 断面図 (Cross-section)（レイヤー化された浅さを表現）

横方向のバンド（`h-12 border-l-4`）をスタックして、呼び出しが通過するレイヤーを表現します。Before: 何もしていない6つの薄いレイヤー。After: 統合された責任のラベルが付いた1つの厚いバンド。

### 質量図 (Mass diagram)（インターフェースが実装と同じ広さであることを表現）

モジュールごとに2つの長方形を描きます（1つはインターフェースの表面積用、もう1つは実装用）。Before: インターフェースの長方形が実装の長方形とほぼ同じ高さ（浅い）。After: インターフェースの長方形が短く、実装の長方形が縦に長い（深い）。

### コールグラフの折りたたみ (Call-graph collapse)

Before: ネストされたボックスとしてレンダリングされた関数呼び出しのツリー。After: 同じツリーが1つのボックスに折りたたまれ、内部の呼び出しが薄く表示されている状態。

## スタイリング指針

- コーポレートダッシュボード風ではなく、雑誌の編集ページのような雰囲気を目指します。余白（ホワイトスペース）を十分に取ります。見出しにはセリフ体（`font-serif`）を stone/slate 色と組み合わせると効果的です。
- 配色は最小限にします：アクセントカラー1色（エメラルドまたはインディゴ）、リーク表現用の赤、警告用のアンバー。
- スクロールせずにBefore/Afterが快適に横並びで見られるよう、図の高さは320px程度に抑えます。
- 図の中のモジュールラベルには `text-xs uppercase tracking-wider` を使用し、デザイン的（図面的）に読めるようにします。
- スクリプトは Tailwind CDN と Mermaid ESM インポートのみを使用します。レポート自体は静的であり、アプリ側のコードや、Mermaid自体が持つ描画処理以外のインタラクティブな動きは含めません。

## 最優先の推奨事項セクション

少し大きめのカードを1つ配置します。候補名、推奨理由を1文、そのカードへのアンカーリンクを記載します。これだけです。

## トーン

簡潔で平易な日本語としますが、アーキテクチャの主語や述語は [LANGUAGE.md](LANGUAGE.md) の語彙を正確に使用します。簡潔に書くことを、曖昧な用語に流れる言い訳にしないでください。

**必ず以下の用語を使用してください：**
モジュール（module）、インターフェース（interface）、実装（implementation）、深さ/深い（depth/deep）、浅い（shallow）、シーム（seam）、アダプター（adapter）、レバレッジ（leverage）、ローカリティ（locality）。

**以下の言葉で代用しないでください：**
コンポーネント（component）、サービス（service）、ユニット（unit）（これらは「モジュール」の代用不可） · API、シグネチャ（signature）（これらは「インターフェース」の代用不可） · 境界（boundary）（「シーム」の代用不可） · レイヤー（layer）、ラッパー（wrapper）（モジュールを意味する場合の代用不可）。

**表現の例：**

- 「Order受付モジュールは浅い — インターフェースが実装とほぼ一致している。」
- 「価格設定（Pricing）がシームを越えてリークしている。」
- 「深くする：インターフェースを1つにし、テストの場所を1つにする。」
- 「2つのアダプターがシームを正当化する：本番用のHTTP、テスト用のインメモリ。」

**メリット（Wins）の箇条書き** では、用語集の言葉を使って効果を表現します：
*「ローカリティ: バグが1つのモジュールに集中する」*、*「レバレッジ: 1つのインターフェース、N個 of 呼び出し箇所」*、*「インターフェースの縮小：実装がラッパー群を吸収する」*。
「メンテナンスしやすくなる」「より綺麗なコード」といった、用語集に含まれていない抽象的な表現は使用しないでください。

前置き（「〜に注意することが重要です」など）は一切省きます。箇条書きにできる文章はすべて箇条書きにし、不要な箇条書きは削除します。新しい言葉を造る前に、[LANGUAGE.md](LANGUAGE.md) に定義された語彙で表現できないか検討してください。
