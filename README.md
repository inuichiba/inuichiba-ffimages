inuichiba-ffimages
このリポジトリは、LINE BOTのFlex Messageで使用する画像資産をCloudflare Pagesから配信するためのものです。

構成
public/carousel/ … カルーセル画像（最初に出てくる画像。例：施設案内やP&R）
public/dogrun/ … ドッグラン拡大画像
public/rules/ … 利用規約拡大画像
public/images/ … イメージ画像(QRコードなど)
public/menu/ … リッチメニュー画像
定義例
"D:\nasubi\inuichiba-ffimages\public\rules\rules_arrival.jpg"のとき
const { baseDir } = require("../lib/env.js");
uri: ${baseDir}rules/rules_arrival.jpg ※publicまでがbaseDirに入る
ブラウザで表示させるとき
https://inuichiba-ffimages.pages.dev/rules/rules_arrival.jpg
キャッシュしたくなったときの覚書
ファイルパス："D:\nasubi\inuichiba-ffimages\_headers"
-------------------------
_headers運用ルール
-------------------------
コメントはゴミになるので、削除して_headersに反映すること
本番運用フェーズでアクセス数や帯域が増えたら、
以下のCloudflare Pages用 _headers を使ってキャッシュ最適化を検討します
現在はCloudflare標準CDNで十分なので、個別制御は行っていませんが
評価のため一時的にキャッシュ最適化を行っています
（テンプレートは /_headers に準備済み。今はgit対象。説明は以下の通り）
✅ 画像ファイルは長期キャッシュ（2ヶ月=5184000秒） immutableで変更不可扱い
→ 基本更新されない。URL変えたときだけ更新すればいい
/*.(jpg|jpeg|png|gif|webp|svg|ico) Cache-Control: public, max-age=5184000, immutable

✅ フォントファイルも長期キャッシュ
→ 基本更新されない。URL変えたときだけ更新すればいい
/*.(woff|woff2|ttf|otf) Cache-Control: public, max-age=5184000, immutable

✅ JS / CSS は1週間キャッシュ（変更時に無効化しやすい期間）
→ 週1で更新できる程度のゆるい設定
/*.(js|css) Cache-Control: public, max-age=604800

✅ JSONも1週間キャッシュ（メニュー定義とか）
→ メニュータップ時の応答定義など、時々変わるもの
/*.(json) Cache-Control: public, max-age=604800

✅ APIレスポンスをキャッシュさせない（動的コンテンツは常に最新にする）
→ 動的な返答はキャッシュさせない
/api/* Cache-Control: no-store

✅ HTMLは常に最新を取得（Cloudflare推奨）
→ ページの表示内容は必ず最新にする
/* Cache-Control: no-store

-----------------------------------------------
inuichiba-ffimages Cloudflare Pages キャッシュ運用ルール
✅ 目的

画像 / JS / JSON などを更新した時、確実に新しいファイルを配信する
ファイル名を変えなくてもCloudflare Pagesでキャッシュを最新にする
✅ 方針

内容 方針
ファイル名は変更しない Cloudflare Pagesがデプロイ時にキャッシュを自動更新
変更後はGitでPushするだけ Git Push → 自動デプロイ → キャッシュ更新
即時確認はDevelopment Modeを使う 本番に影響せず、オリジンから直接読ませる
Zone IDでのPurgeは使わない Pagesは通常のDNSサイトと別枠（対象外）
✅ 運用手順 🔹 更新 - 画像やJSファイルを修正 - Gitでコミット & Push - npx wranglerコマンド - Cloudflare Pagesが自動で新しいデプロイを行う - CDNキャッシュも自動更新

git add .
git commit -m "fix: update images"
git push
npx wrangler pages deploy


🔹 注意点

通常はファイル名変更不要
「名前を変えずに」即時確認可能
Zone IDでのPurgeは不要、無意味（Pagesは対象外）