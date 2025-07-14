# inuichiba-ffimages
  このリポジトリは、LINE BOT の `Flex Message`、`QRコードなどのイメージ画像` で使用する画像資産を **Cloudflare Pages** から配信するためのものです。
  このリポジトリには最低限の資産以外は入れないポリシーとし、Cloudflare Pages から配信するための画像ファイル(public/)関連と、GitHubにバックアップを保管するための.git関連以外は含めません。

---

## 構成
- gitとCloudflare Pagesに必要なファイル以外入れないこと(最低限のファイル構成で運用するポリシー)

```sh
  public             # 画像ファイル一式
    public/carousel/ # カルーセル画像（最初に出てくる画像。例：施設案内やP&R）
    public/dogrun/   # ドッグラン拡大画像
    public/rules/    # 利用規約拡大画像 
    public/images/   # イメージ画像(QRコードなど)
  
  .git/              # Git が使用するファイル群一式 
  .gitignore         # Git に含めないファイルを記述(ログなど不必要なファイル) 
  .gitattributes     # このリポジトリ内のファイルを、Git がどう扱うかを指定する設定ファイル 

  _headers           # CDNキャッシュを行うファイルを定義(後述) 
 
  README.md          # このファイル(このディレクトリの説明を書いたファイル)
``` 

--- 

## 定義例
```sh
https://inuichiba-ffimages.pages.dev/rules/rules_arrival.jpgのとき

import { getEnv } from "../lib/env.js"; # 相対パスで定義する 
const { baseDir } = getEnv(env);        # 必ず関数内で定義する(遅延実行が必要)
uri: ${baseDir}rules/rules_arrival.jpg  # /までがbaseDirに入る
```

---

## ブラウザで表示させるときの例
 - https://inuichiba-ffimages.pages.dev/rules/rules_arrival.jpg
 
## キャッシュしたくなったときの覚書
 - ファイルパスを `_headres` の中に定義する
    - Windowsの場合 `D:\nasubi\inuichiba-ffimages` 配下の `_headers`
    - Mac/Unix場合  `/Users/yourname/projectname/inuichiba-ffimages` 配下の `_headers`

---

# _headers運用ルール
 - コメントはゴミになるので、削除して_headersに反映すること
 - 本番運用フェーズでアクセス数や帯域が増えたら、
   以下のCloudflare Pages用 _headers を使ってキャッシュ最適化を検討します
 - 現在はCloudflare標準CDNで十分なので、個別制御は行っていませんが、
   評価のため一時的にキャッシュ最適化を行っています
 - （テンプレートは /_headers に準備済み。今はgit対象。説明は以下の通り）

 ✅ 画像ファイルは長期キャッシュ（2ヶ月=5184000秒） immutableで変更不可扱い
   - → 基本更新されない。URL変えたときだけ更新すればいい
   - /*.(jpg|jpeg|png|gif|webp|svg|ico) Cache-Control: public, max-age=5184000, immutable

 ✅ フォントファイルも長期キャッシュ
   - → 基本更新されない。URL変えたときだけ更新すればいい
   - /*.(woff|woff2|ttf|otf) Cache-Control: public, max-age=5184000, immutable

 ✅ JS / CSS は1週間キャッシュ（変更時に無効化しやすい期間）
   - → 週1で更新できる程度のゆるい設定
   - /*.(js|css) Cache-Control: public, max-age=604800

 ✅ JSONも1週間キャッシュ（メニュー定義とか）
   - → メニュータップ時の応答定義など、時々変わるもの
   - /*.(json) Cache-Control: public, max-age=604800

 ✅ APIレスポンスをキャッシュさせない（動的コンテンツは常に最新にする）
   - → 動的な返答はキャッシュさせない
   - /api/* Cache-Control: no-store

 ✅ HTMLは常に最新を取得（Cloudflare推奨）
   - → ページの表示内容は必ず最新にする
   - /* Cache-Control: no-store

---

# inuichiba-ffimages Cloudflare Pages キャッシュ運用ルール
 ✅ 目的

   - 画像 / JS / JSON などを更新した時、確実に新しいファイルを配信する
   - ファイル名を変えなくてもCloudflare Pagesでキャッシュを最新にする

 ✅ 方針

   - ファイル名は変更しない Cloudflare Pagesがデプロイ時にキャッシュを自動更新
   - 変更後はGitでPushする Git Push → 手動デプロイ → キャッシュ更新
   - 即時確認はDevelopment Modeを使う 本番に影響せず、オリジンから直接読ませる
   - Zone IDでのPurgeは使わない Pagesは通常のDNSサイトと別枠（対象外）

 ✅ 運用手順
   - 更新(通常は以下を使う)
      - Windowsの場合  `D:\nasubi\inuichiba-ffscripts\ffimages-upload-deploy.ps1`
      - Mac/Unixの場合 `/Users/yourname/projectname/inuichiba-ffscripts/sh/ffimages-upload-deploy.sh`

 ✅ スクリプトで行われていること
 
```sh
1. 画像やJSファイルなどを修正
2. Gitでコミット & Push
3. npx wranglerコマンド → Cloudflare Pagesが自動で新しいデプロイを行う
4. CDNキャッシュも自動更新
```
```sh
# 画像やJSファイルなどを修正後
git add -A
git commit -m "fix: update images"
git push                  # 初回に git push -u origin main を実行していればこれだけでよい
npx wrangler pages deploy # GitHubに登録され、CDNキャッシュも自動更新
```
   - 注意点
       - 通常はファイル名変更不要
       - **名前を変えずに** 即時確認可能(5~10分程度かかる場合もある)
       - Zone IDでのPurgeは不要、無意味（Pagesは対象外）

