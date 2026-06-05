## readme-hero-rota

> README のヒーロー画像を毎日自動で切り替える GitHub Actions テンプレート。
> `HERO_START`/`HERO_END` マーカー + cron slot 選定ガイドつき、forkable。

<!-- HERO_START -->
<p align="center">
  <img src="./assets/heroes/hero_1.png" width="80%">
</p>
<!-- HERO_END -->

毎日 JST 07:00(UTC 22:00)に GitHub Actions の cron が走り、
[`assets/heroes/`](./assets/heroes/) からランダムで新しいヒーロー画像を選び(現在表示中の画像は除外)、commit & push します。

自分の携わっているプロジェクトで、ヒーロー画像が毎日変わって嬉しいと感じる方が
どこまでいらっしゃるかは分かりませんが、PC の日替わり壁紙のような感覚で
ご利用いただければ。


### このテンプレートで実現できること

- 毎日決まった時刻(デフォルト JST 07:00)に README のヒーロー画像を自動切替
- 現在表示中の画像を除外して random pick(連日同じ画像にならない)
- `<!-- HERO_START -->` / `<!-- HERO_END -->` マーカー方式 — 既存の `width`、`alt`、ラッパ要素は保持
- 依存性ゼロ(Python stdlib のみ、追加 install なし、workflow 全体で 90 行未満)
- 手動キック対応(`workflow_dispatch`)— Actions タブから即時実行可
- 他 workflow を巻き込まない設計(`[skip ci]` + `paths-ignore` ガイド)
- Node.js 24 切替(2026-06-16 デフォルト化)対応済み

### クイックスタート(fork して動かす場合)

1. 本リポジトリを fork する(または "Use this template")
2. `assets/heroes/` にヒーロ画像として利用したい画像を投入する — `.png` / `.jpg` / `.jpeg` /
   `.webp` / `.gif` 対応。ファイル名は自由(`sorted()` で並ぶだけ)
3. 権限設定: `Settings → Actions → General → Workflow permissions → Read and write permissions` をチェック[☑]
4. 動作確認:
   Actions タブ → "Rotate README hero" → Run workflow を押す。数秒で bot commit が出れば OK
6. 翌朝 JST 07:00(UTC 22:00)から cron が自動で回します

### 仕組み

```
README.md
├── <!-- HERO_START -->
│     <img src="./assets/heroes/<現在>.png" ...>
└── <!-- HERO_END -->

.github/workflows/rotate-hero.yml
└── 1. リポジトリを checkout
    2. assets/heroes/ から「現在の画像以外」をランダム選択
    3. HERO ブロック内の <img src> だけを差し替え
    4. github-actions[bot] で commit + push
```

Python は workflow にインライン(stdlib only)。追加 install ゼロ。

### カスタマイズ

| 何を | 場所 | やり方 |
|---|---|---|
| 発火時刻を変える | `.github/workflows/rotate-hero.yml` の `cron:` | UTC 限定。[docs/cron-timing.md](./docs/cron-timing.md) 参照 |
| 画像幅を変える | README の `<img ...>` タグ | `width="80%"` を編集 |
| 画像ディレクトリ | workflow 内の Python | `Path("assets/heroes")` を編集 |
| 手動キック | Actions タブ | "Run workflow" ボタン(`workflow_dispatch` 有効済) |
| CI 二重起動を抑止 | 他 workflow の yaml | `paths-ignore` を追加(後述) |

### [TIPS] cron タイミングの注意

GitHub Actions の cron は best-effort UTC(混雑時間帯で遅延しがち)。
スロット選定の目安:

| slot | グローバル状況 |
|---|---|
| UTC 14:00–19:00 | 米国 + 欧州が両方アクティブ(混雑ゾーン)|
| UTC 22:00(JST 07:00、本リポジトリのデフォルト) | 米国 EOD + 欧州 就寝(穏やかなウィンドウ)|

米国と欧州の業務時間が両方落ち着いている UTC 帯を選ぶとよい。
詳細と確認コマンドは [docs/cron-timing.md](./docs/cron-timing.md)。

### CI を無駄に回さないための二段防御

bot commit が走るたびに test job が回らないように:

#### 1. `[skip ci]` を commit message に

本テンプレの commit message は:

```
Rotate README hero image [skip ci]
```

ほとんどの CI サービスはこのマーカーを尊重して build を skip します。

#### 2. 他 workflow に `paths-ignore`

保険として、test workflow 側でも README / 画像変更を無視:

```yaml
on:
  push:
    paths-ignore:
      - "README.md"
      - "assets/heroes/**"
      - ".github/workflows/rotate-hero.yml"
```

### ドキュメント

- [docs/setup.md](./docs/setup.md) — fork してから稼働まで 5 分の手順
- [docs/cron-timing.md](./docs/cron-timing.md) — cron slot 選びのガイド
- [docs/troubleshooting.md](./docs/troubleshooting.md) — よくある問題と解決

### 📝 ライセンス

MIT — [LICENSE](./LICENSE) 参照。

`assets/heroes/` のサンプル画像はプレースホルダです。fork したら自分の画像に差し替えてください(著作権はあなたが用意した画像の出典に従う)。
