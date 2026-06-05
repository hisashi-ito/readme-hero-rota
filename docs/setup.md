## セットアップ手順

fork してから稼働まで 5 分。

### 1. fork または template として複製

- fork: ページ右上の "Fork" ボタン
- template: 本リポジトリを Settings → General → "Template repository"
  にした上で、相手側から "Use this template" を押してもらう

### 2. clone する

```bash
git clone https://github.com/<あなたのアカウント>/readme-hero-rota.git
cd readme-hero-rota
```

### 3. ヒーロー画像を差し替える

`assets/heroes/` に自分の画像を投入。
対応拡張子: `.png` / `.jpg` / `.jpeg` / `.webp` / `.gif`。
ファイル名は自由(workflow はアルファベット順で扱う)。

推奨枚数: 10〜30枚。少なすぎると rotation がすぐ一周してバレる、
多すぎると `_manifest.txt` の管理がしんどい。

```bash
rm assets/heroes/hero_*.png                    # サンプルプレースホルダを削除
cp /path/to/自分のヒーロー/*.png assets/heroes/
git add assets/heroes/
git commit -m "ヒーロープールを差し替え"
git push
```

### 4. GitHub Actions の権限を有効化

最近作られた repo はデフォルトで GITHUB_TOKEN が read-only に。
rotate workflow は `git push` するので、書込み権限が必須:

1. リポジトリの Settings → Actions → General
2. 下にスクロール Workflow permissions
3. Read and write permissions を選択
4. Save で保存

> "Allow GitHub Actions to create and approve pull requests" は OFF
> のまま で問題ありません(本テンプレは PR を作らず直接 commit)。

### 5. 手動キックで動作確認

1. Actions タブ
2. 左サイドバーから "Rotate README hero" を選択
3. 右上の Run workflow ボタンを押す
4. 数秒で実行完了
5. main ブランチに `"Rotate README hero image [skip ci]"` の bot
   commit が出て、README のヒーローが切り替わってればOK

何も起きない場合は [troubleshooting.md](./troubleshooting.md) を参照。

### 6. cron に任せて毎朝自動更新

デフォルトの cron(`22:00 UTC = 07:00 JST`)で翌朝以降は自動で回ります。
遅延 5〜60 分 は GitHub Actions cron の仕様なので諦めてください。
詳細は [cron-timing.md](./cron-timing.md)。

## 7. (任意)スケジュール時刻を変える

`.github/workflows/rotate-hero.yml` を編集:

```yaml
on:
  schedule:
    - cron: "0 22 * * *"   # ← ここを変える
```

GHA の cron は UTC のみ(タイムゾーン指定不可)。
ローカル時間を UTC に変換して入れる。`:30` 系の方が `:00` より混雑回避できる。
詳しくは [cron-timing.md](./cron-timing.md)。

## 8. (任意)ロゴをヒーローの上か下に置く

HERO マーカーは中身だけを swap するので、外側に他の要素を置いて OK:

```html
<p align="center">
  <img src="./images/あなたのロゴ.png" width="60%" alt="Logo">
</p>
<!-- HERO_START -->
<p align="center">
  <img src="./assets/heroes/hero_1.png" width="100%">
</p>
<!-- HERO_END -->
```

workflow は HERO ブロック内の `<img src>` だけを触るので、
ロゴや subtitle、バッジは触られません。
