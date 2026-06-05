## トラブルシューティング

### workflow は動くが commit が 403 で失敗する

> `remote: Permission to <owner>/<repo>.git denied to github-actions[bot]`

Workflow 権限が read-only になってます。

対処: リポジトリの Settings → Actions → General → Workflow permissions
→ Read and write permissions に切替。

workflow yaml にも `permissions: contents: write` を書いていますが、
これは「repo 設定が許す上限の範囲で要求できる」もので、
repo 側が read-only なら yaml の指定は効きません(下限のみ調整可)。

### "HERO_START / HERO_END block not found" と出る

Python スクリプトは README.md 内の literal な `<!-- HERO_START -->`
と `<!-- HERO_END -->` マーカーを探しています。
両方の綴りが正確に存在することを確認してください。

### "No hero images found in assets/heroes" と出る

ディレクトリが空、または対応してない拡張子ばかりです。
対応拡張子: `.png`, `.jpg`, `.jpeg`, `.webp`, `.gif`。

`assets/heroes/` に最低1枚画像を入れて再実行してください。

### cron がスケジュール時刻に発火しない

GHA cron は best-effort UTC。詳しくは
[cron-timing.md](./cron-timing.md)。

短くまとめると:

- 通常で 5〜60 分の遅延 は想定範囲
- UTC 14:00–19:00 は混雑するので避ける
- 直前に workflow を変更 した場合、初回はスキップされやすい
- 60 日以上 inactive な repo は cron が止められている

困ったら 手動 dispatch(Actions タブ → "Run workflow")で即時実行。

### bot commit のたびに CI が走ってしまう

二段防御で対処:

1. 本テンプレの bot commit は `[skip ci]` をメッセージに含んでます
   → 多くの CI サービスがこれを尊重して build をスキップ
2. test workflow 側にも `paths-ignore` を追加:

```yaml
on:
  push:
    paths-ignore:
      - "README.md"
      - "assets/heroes/**"
      - ".github/workflows/rotate-hero.yml"
```

これで `[skip ci]` を尊重しないツールに対しても安全。

## 同じヒーローが2日連続で出る

スクリプトは現在の画像を除外してから random pick するため、
プールに2枚以上ある限り連続は起きません。

唯一の例外: プールが1枚しかない場合 → 候補なしフォールバック →
結局同じ画像 → `git diff --quiet` で "No changes" で終了。

画像を増やしてください(推奨 10 枚以上)。

### 既存の `<img>` の他属性が消える

スクリプトは HERO ブロック内の 最初の `<img>` タグの `src` 属性
だけを書き換えます。`width`、`alt`、クラス、ラッパ要素 は保持されます。

それでも他属性が消える場合は、HERO ブロックのスニペットを添えて issue
を立ててください。

### 複数の HERO ブロックを使いたい

スクリプトは 最初の HERO ブロックだけを swap します。複数 region
で別々に回したい場合は workflow の Python ロジックを分岐させてください
(`block_re.search` を `findall` に変えて各マッチを個別処理する等)。

### fork ごとに違うスケジュールにしたい

`workflow_dispatch:` は常に on なので、各 fork は手動 trigger 可能。
スケジュールを変えたい場合は、fork 側で
`.github/workflows/rotate-hero.yml` の `cron:` 行を編集してください。

### Node.js 20 の deprecation 警告が出る

本テンプレは既に `actions/checkout@v5`(Node 24 native)を使い、
workflow env に `FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: "true"` を
セットしています。通常は警告は出ません。

警告が出る場合は、fork でバージョンを下げてないか確認してください。
