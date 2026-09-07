# site/ の公開手順（GitHub Pages）

このディレクトリの中身（`index.html`・`privacy.html`・`report.html`・`filters/`）を
GitHub Pagesで公開するための手順です。公開先リポジトリの作成にはGitHubアカウントの
操作が必要なため、ここまでは自動化していません。以下を社長が実行してください。

## 事前に決めておくこと

1. **公開先リポジトリ名**（例: `yokete-filters`）。個人アカウント配下でもOrganization
   配下でも構いません。

問い合わせ先（`privacy.html`・`report.html`内の連絡先）は
`eatsafe.japan@gmail.com`で確定済みのため、両ファイルに直接記載してあります。
置き換え作業は不要です。

## 手順

1. GitHubで新しい公開リポジトリを作成する（例: `yokete-filters`）。READMEなどの
   初期ファイルは無しで作成して構いません。

2. このディレクトリ（`apps/adblock-jp/site/`）の中身を、手順1で作った
   リポジトリのルート（またはリポジトリ内の`docs/`フォルダなど、GitHub Pages
   の公開元に指定する場所）にコピーする。`index.template.html`は生成用の
   雛形なので公開リポジトリには含めなくてよい（`index.html`だけが公開ページ
   本体）。

   ```bash
   # 例: 別ディレクトリに新規リポジトリを用意している場合
   cp apps/adblock-jp/site/index.html      /path/to/yokete-filters/
   cp apps/adblock-jp/site/privacy.html    /path/to/yokete-filters/
   cp apps/adblock-jp/site/report.html     /path/to/yokete-filters/
   cp -r apps/adblock-jp/site/filters      /path/to/yokete-filters/
   ```

3. リポジトリにコミット・pushする。

   ```bash
   cd /path/to/yokete-filters
   git add .
   git commit -m "ヨケテ フィルタと検証結果ページを公開"
   git push
   ```

4. GitHubのリポジトリ設定 → **Pages** を開き、公開元（Source）を
   「Deploy from a branch」、ブランチを`main`（コピー先が`docs/`フォルダの
   場合はフォルダも`/docs`に指定）にして保存する。

5. 数分待ってから、表示された公開URL（例:
   `https://<ユーザー名>.github.io/yokete-filters/`）にアクセスし、
   `index.html`が表示されることを確認する。

6. フィルタが実際に取得できることを確認する。

   ```bash
   curl -sI https://<公開URL>/filters/ads.json | head -3
   ```

   `HTTP/2 200` が返ってくればOK。

7. 公開URLが確定したら、`apps/adblock-jp/ios/Shared/RuleUpdater.swift`の
   取得先URL（`baseURL`の既定値）を、確定した`https://<公開URL>/filters/`に
   合わせて更新する（現在の既定値は`https://mjryoryo.github.io/yokete-filters/filters/`という
   仮のURLです）。

## 更新のたびに行うこと（2回目以降）

パイプラインを再実行してルール・検証結果ページを更新したときは、次を実行して
生成物を作り直してから、公開リポジトリへコピー・pushしてください。

```bash
cd apps/adblock-jp/pipeline
.venv/bin/python -u -m yokeru.runner
PYTHONPATH=. .venv/bin/python -c "
from pathlib import Path
from yokeru.publish import publish
publish(Path('out'), Path('../site'))
"
```
