# DESIGN-BOOTSTRAP.md — 初回実装ガイド(移行完了後に削除)

`DESIGN.md`(クロマティック・ニューモーフィズム)を現行コードベースに初めて適用するための情報。
**恒久ルールは一切ここに書かない。** ここにあるのは「現状とゴールの差分」と「一度だけ必要な作業」のみ。
全フェーズ完了後、このファイルは削除してよい。

> **2026-07-07 追記:** Phase 0〜3(色トークン移行)に続き、初回実装が既存レイアウトへの装飾追加に留まり
> 承認済みモック(案A)と乖離しているという指摘を受け、テンプレートHTML構造を含めて**全面改修**した
> (ヘッダー再構築・ヒーロー画像廃止・記事カード独立化・ウィジェット再構築・Bulma詳細度/tocbot初期化順序/
> サムネイル404の修正・muted色のAA調整 等)。DESIGN.md はこの実装に合わせて §2.2/§2.4/§3.1/§3.3/§4 を更新済み。
> ただし下記 §5.2(シェアボタン)・§5.3(手動テーマトグル)は今回のスコープ外のままで、
> このファイルは**まだ削除しない**(§7 の完了条件を参照)。

---

## 1. 作業環境の注意(このリポジトリ固有)

- このテーマは親リポジトリ azw-blog の **git submodule**。コミットはテーマ側で行い、親リポジトリで submodule 参照を更新する(2段階コミット)。
- Hugo はビルドを**親リポジトリから**実行する。`hugo` は PATH にない — 親リポジトリ直下の未追跡バイナリを使う(macOS では `./hugo`、Windows では `./hugo.exe`。extended **0.147.5**)。
  ```bash
  cd ../..          # azw-blog ルートへ
  ./hugo server     # ローカル確認(macOS)
  ```
- Bulma 0.9.0 が CDN で読み込まれており、詳細度の戦いが起きやすい(タグ `.tag` などは Bulma のクラス)。既存コードが `// override bulma` とコメントしている箇所が参考になる。
- Prism.js の CSS は `prism--` プレフィックス付き(親リポジトリ側 `static/css/`)。本文コードブロックの見た目には触れない(DESIGN.md §4「本文」)。

## 2. 現状インベントリ(2026-07-06 時点)

> **注意(2026-07-07):** 以下の行番号・セレクタ名は改修前のスナップショットであり、全面改修後の現状とは一致しない
> (例: `.list-tags .tag` → `.pill` に置換済み)。現在の構成は CLAUDE.md の Template Hierarchy / Styling 節を参照。
> この節は「移行時に何がどこにあったか」の記録としてのみ残す。

### 2.1 SCSS 構成(すべて `assets/scss/`)

| ファイル | 役割 | 今回の関与 |
|---|---|---|
| `_variables.scss` | 全カラートークン定義(SCSS変数 → CSS変数の2段構え) | **主戦場**(Phase 0) |
| `_mixins.scss` | ブレークポイント+ `neu-*` シャドウ mixin | 新 mixin 追加(Phase 0) |
| `_layout.scss` | ヘッダーナビ(L2-42)、`.site-title`(L48)、フッター(L70) | Phase 1, 2 |
| `_components.scss` | `.article-continue`(L47)、ページネーション(L72)、シェアボタン(L128) | Phase 2, 3 |
| `_article.scss` | 記事カード・本文まわり(neu-raised 多用) | Phase 2(タイトルホバーのみ) |
| `_widgets.scss` | サイドバー `aside`(L56)、`.widget-title`(L70)、タグ `.list-tags .tag`(L100 付近) | Phase 2 |
| `_base.scss` | リンク色(L15-22) | Phase 3 |

### 2.2 ダークモードの現状

`@media (prefers-color-scheme: dark)` のみで、`data-theme` 属性による手動トグルは**未実装**。
ブランドトークン定義に `[data-theme]` セレクタが含まれているのは将来のトグル導入を見据えたもの。
初回移行は media query のみで進めてよいが、トークンを書く際は後から `:root[data-theme="dark"]` ブロックを追加しやすい構造(トークン一括再定義)にしておくこと。

## 3. 現状 → ゴールのトークンマッピング

### 3.1 ⚠️ 最重要: セカンダリカラーの色相移行(83 → 148)

現行の `$light-second-color: hsl(83, 68%, 62%)` は**黄緑**であり、ブランドの `#1AEA7C` = **hsl(148, 83%, 51%)(エメラルド)とは別の色**。今回の移行で最も影響範囲が広い変更。

| 現行変数 | 現行値 | 新値(基準) |
|---|---|---|
| `$light-second-color` | `hsl(83, 68%, 62%)` | `#16D977`(= accent-secondary-ui light) |
| `$light-second-color2` | `hsl(83, 55%, 45%)` | `#007C4B`(= accent-secondary-text light)基準で再導出 |
| `$light-second-color3` | `hsl(83, 35%, 62%)` | hue 148 で彩度・明度を踏襲して再導出 |
| dark 系(`$dark-second-*`) | `adjust-color()` で自動導出 | dark の基準は `#1AEA7C`。自動導出結果を目視確認し、ずれる場合は直接指定に切替 |

`--strong-text-color: var(--second-color2)` として本文の強調色にも使われているため、**全記事の `<strong>` の色が変わる**。移行後に記事ページで目視確認すること。

### 3.2 プライマリ(ピンク)は現状維持

`$light-first-color: hsl(337, 68%, 62%)` ≈ `#DC4F88`。ブランドと実質一致しているので変更不要。

### 3.3 追加するトークン(DESIGN.md §2.3)

`_variables.scss` の `:root` / dark ブロックに追加:

```scss
// Light
--glow-pink:   rgba(220, 79, 136, 0.22);
--glow-green:  rgba(22, 217, 119, 0.22);
--split-pink:  rgba(220, 79, 136, 0.55);
--split-green: rgba(13, 176, 103, 0.50);

// Dark
--glow-pink:   rgba(234, 92, 153, 0.40);
--glow-green:  rgba(26, 234, 124, 0.35);
--split-pink:  rgba(234, 92, 153, 0.75);
--split-green: rgba(26, 234, 124, 0.65);
```

### 3.4 追加する mixin(`_mixins.scss`)

既存の `neu-*` は**変更しない**(まず追加、置換は各フェーズで段階的に):

```scss
// 色付きグロー: 既存の neu シャドウの「上に重ねる」用
@mixin neu-glow($glow-color, $blur: 12px) {
    box-shadow: $neu-radius $neu-radius $neu-blur $shadow-color,
                (-$neu-radius) (-$neu-radius) $neu-blur $shadow-highlight-color,
                0 0 $blur $glow-color;
}

// 色収差シグネチャ(DESIGN.md §3.2 の適用制限を厳守)
@mixin chromatic-split($offset: 2.5px) {
    text-shadow: (-$offset) 0 0 var(--split-pink),
                 $offset 0 0 var(--split-green);
}
```

## 4. フェーズ計画

各フェーズ完了ごとに light/dark 両方を目視確認してからコミットする。

- **Phase 0 — トークン基盤**: §3.1 の色相移行 + §3.3 のトークン追加 + §3.4 の mixin 追加。
  この時点でサイト上の緑がすべて変わる。強調テキスト・カテゴリ表示を重点確認。
- **Phase 1 — シグネチャ**: `.site-title` に `chromatic-split(2.5px)` 常時適用(`_layout.scss` L48)。
  記事タイトル(`_article.scss` の `article.content h1` / カードタイトル)ホバーに `chromatic-split(1.5px)`。
- **Phase 2 — アクセントUI**: タグ(`_widgets.scss` `.list-tags .tag`)をピンク枠+グロー化。
  ヘッダーナビホバー(`_layout.scss` L27-33 の手書き box-shadow)にピンクグロー追加。
  `.widget-title` をグリーン化。セクション区切りをグラデーションライン化。
- **Phase 3 — 収束**: リンク hover のシアン `hsl(194, 86%, 51%)`(`_base.scss`)をピンク系
  `--accent-primary-text` へ移行(DESIGN.md §5-3)。`$link-text-color`(`.article-continue` が使用)も同時に。
  ページネーション active のダークグロー。背景明度の最終調整(下記 §5)。

## 5. 初回に判断が必要な残論点

1. **ライト背景の明度**: 現行 `hsl(337, 20%, 98%)` はかなり白く、ハイライト影(白)との差が小さい。
   モックでは `95%` 程度に落として立体感を強調した。Phase 3 で 96〜95% への変更を試し、目視で決める。
   → **決定: 95%**(2026-07-06、98/96/95% の3案スクリーンショット比較で決定。DESIGN.md §2.4 に反映済み)。
2. **シェアボタン**(`_components.scss` L128-): X/Facebook 等の他社ブランド色がベタ塗りで並び、
   2色相ルール(DESIGN.md §5-3)の例外になっている。初回は触らず、アイコンのモノトーン化+ホバーで
   ブランド色復帰、などの案を別途検討する。
   → **未解決(2026-07-07 時点)**: モック忠実実装(構造全面改修)でも意図的にスコープ外とした
   (依頼に「シェアボタンの改修はスコープ外」と明記されていたため)。新背景色下で見た目が破綻しないことのみ確認済み。
3. **手動テーマトグル**: 導入するなら JS で `data-theme` を付与し、トークンブロックを
   `:root[data-theme="dark"]` / `[data-theme="light"]` に複製する。初回スコープ外とした。
   → **未解決(2026-07-07 時点)**: `:root[data-theme="dark/light"]` セレクタは依然未実装。
   `prefers-color-scheme` のみで運用中。

## 6. 検証方法

**恒久的な検証手順は CLAUDE.md の「Verification Checklist for Visual Changes」に移管した**(実ブラウザで light/dark 両モード、
トップ/記事/タクソノミー+モバイル幅、コントラストは計算で AA 判定、常時グロー3種以内、コンソールエラー/404 ゼロ)。
2026-07 の移行検証は Playwright(chromium、`colorScheme` エミュレーション)+ computed style のモック値照合で実施した。

## 7. 完了条件

Phase 0〜3 はコミット済み、構造全面改修(2026-07-07)も完了し DESIGN.md に反映済み。
残るのは §5.2(シェアボタン)・§5.3(手動テーマトグル)の2点のみ。この2点に結論が出て
DESIGN.md に反映されたら、このファイルを削除する。
