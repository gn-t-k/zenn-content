# Zenn × GA4 × 検索キーワード運用：一次情報調査

- 調査日: 2026-11-11
- 対象: Zenn 公式ドキュメント（zenn.dev / info.zenn.dev）、Google 公式ヘルプ（GA4 / Search Console / Google Ads / Google Trends）、Google Search Central
- 方針: 一次情報で確認できた事実には出典リンクを付けます。zenn.dev の HTML や JS バンドルを直接読んで確かめたことは「実地確認」と書きます。筆者の推論は「推論」と明記します。

## 結論サマリ

1. **Zenn の個人アカウントには GA4 の測定ID を設定できます。** 2020-11-19 から使える機能で、「アカウント設定」から登録します。プランの条件はなく、`https://zenn.dev/ユーザー名` で始まる URL へのアクセスが計測されます（[Zenn リリース 2020-11-19](https://info.zenn.dev/2020-11-19)）。公開ページには `G-` で始まる ID を設定しているユーザーが実際にいるので、GA4 の ID が受け付けられることも確認できました（実地確認）。
2. **GA4 で記事ごとに見られるのはアクセスの量と流入元までです。どの検索キーワードで来たかは見られません。** Google 検索のクエリは Search Console 側のデータで、GA4 に取り込むには「Search Console プロパティの確認済み所有者」であることが前提です（[GA4 Search Console 連携](https://support.google.com/analytics/answer/10737381?hl=en)）。
3. **Zenn の記事について Search Console の所有権確認をする現実的な方法は見当たりません。** Zenn 公式にも Search Console 連携の案内はありません。URL プレフィックスプロパティの確認方法（HTML ファイル、HTML タグ、GA、GTM、DNS）はどれも、ページの `<head>`、サイトのルート、ドメインの DNS のいずれかを自分で編集できることが前提で、Zenn のユーザーにはその手段がありません（[確認方法](https://support.google.com/webmasters/answer/9008080?hl=en)）。GA 方式も成立しない可能性が高いと考えますが、実際には試していません（未確認事項を参照）。**そのため、Zenn でキーワード単位の評価を直接行うことはできません。** 代わりに使える指標は、GA4 の「ランディングページ × セッションの参照元/メディア」で見る Organic Search 流入の推移と、手動での検索確認です。
4. **公開後に変えられるもの:** タイトル、本文、トピック（5つまで）、絵文字は、記事を更新すれば変えられます（[Zenn CLI ガイド](https://zenn.dev/zenn/articles/zenn-cli-guide)）。**slug（URL）は変えられません。** slug を変えると別の記事として作成されます（[slug とは](https://zenn.dev/zenn/articles/what-is-slug)）。`published_at` も一度設定したら変更できません。記事ページには公開日と並んで本文の更新日が表示されます（実地確認）。
5. **Zenn の記事 HTML には `meta name="description"` と `og:description` がありません**（実地確認）。著者が触れる検索向けの要素は、実質的に**タイトル（`<title>`・`og:title`・`h1` のすべてに使われる）と本文**だけです。Google はスニペットを主に本文から作るので（[スニペット](https://developers.google.com/search/docs/appearance/snippet)）、冒頭の段落が事実上の説明文になると考えられます（推論）。

---

## 1. Zenn は GA4 の測定ID 設定に対応しているか

### 個人アカウント

- 2020-11-19 のリリースで「ユーザーがご自身のページに Google Analytics を設定できるように」なりました。`https://zenn.dev/ユーザー名` で始まる URL へのアクセスがあると、登録したトラッキングID で計測されます。ID は「アカウント設定」から登録します（[Zenn リリース 2020-11-19](https://info.zenn.dev/2020-11-19)）。
- 有料プランが必要だという記述はありません。リリース文の対象は「ユーザー」全般です（同上）。
- GA4（`G-` で始まる測定ID）への対応について、Zenn が公式に告知した記事は見つかりませんでした。ただし公開記事ページに埋め込まれたユーザー情報には `"gaTrackingId":"G-XXXXXXX"` という形の値が実際に入っていました（複数の公開記事で実地確認）。少なくとも GA4 の ID は登録でき、計測にも使われています。
- 具体的な画面の操作（「アカウント設定 → トラッキングIDを設定」など）は、Zenn 公式ドキュメントでは確認できませんでした。ログインが必要な画面なので、今回は実物も見ていません。

### Publication

- Publication 単位の GA 連携は **Publication Pro（有料プラン）限定**です。2024-11-26 から、Publication ダッシュボードの「外部連携」で設定します。計測対象は「Publication のトップページと、Publication へ紐づけられた記事ページ」で、`page_view` などを計測できます（[Zenn リリース 2024-11-26](https://info.zenn.dev/2024-11-26-publication-pro-google-analytics)）。
- 記事ページの JS では、**著者ユーザーの gaTrackingId と Publication の gaTrackingId をそれぞれ別のフックに渡して計測しています**（実地確認：記事ページのバンドルに `J({gaTrackingId: <ユーザー>.gaTrackingId})` と `R({gaTrackingId: publication?.gaTrackingId})` が並んでいます）。このコードを読む限り、Publication に紐づけた記事でも著者個人の GA に page_view が送られます（推論。Publication 記事で実際に送信されることは確認していません）。

### 本・スクラップ・プロフィール

- リリース文の書き方からすると、`zenn.dev/ユーザー名` 以下（本・スクラップ・プロフィールを含む）はすべて計測対象です（[Zenn リリース 2020-11-19](https://info.zenn.dev/2020-11-19)）。
- 実地確認できたのは、**記事ページ**と**プロフィールページ**のバンドルが同じ計測フック（`J`）を呼んでいることまでです。本とスクラップのページのコードは確認していません。

### Zenn 内蔵の統計

- GA とは別に、Zenn には「ユーザー向け統計ダッシュボード（β）」があります（2023-06-01〜）。記事を1つ以上公開していれば使えて、閲覧数や執筆状況をグラフで見られます。データは翌日に反映されます。本やスクラップしか公開していない場合は使えません（[Zenn リリース 2023-06-01](https://info.zenn.dev/2023-06-01-stats-dashboard-beta)）。

---

## 2. Zenn の GA4 連携が実際に送っているデータ

### 送信の仕組み（実地確認：記事ページの JS バンドル）

記事ページの JS バンドルにある計測フックの処理は、要約すると次のとおりです。

```js
// 個人用（cookiePrefix "zn_usr", cookieExpires 86400）／Publication 用（"zn_pub", 2592000）
router.events.on("routeChangeStart", () => { window[`ga-disable-${id}`] = true });
setTimeout(() => {
  window[`ga-disable-${id}`] = false;
  gtag("config", id, { send_page_view: false, cookie_domain: "none",
                       cookie_prefix: "zn_usr", cookie_expires: 86400 });
  gtag("event", "page_view", { page_path: asPath, send_to: id });
}, 1000);
```

ここから言えること（事実部分はコードそのもの、影響の部分は推論）:

- **送っている明示イベントは `page_view` だけです。** 自動の page_view は `send_page_view: false` で止めていて、ページを表示するたびに（SPA の画面遷移を含む）自前で `page_view` を1回送ります。
- **表示後 1 秒経ってから送ります。** ページ遷移が始まると `ga-disable` で計測を止めます。推論すると、1 秒以内に離脱した閲覧は数えられません。
- **個人用の cookie の有効期限は 86400 秒（1日）です。** GA4 の `_ga` cookie はユーザーを区別するためのもので、標準の有効期限は 2 年です（[GA4 cookie の使用](https://support.google.com/analytics/answer/11397207?hl=en)）。推論すると、Zenn の個人用計測では翌日以降に戻ってきた読者が「新規ユーザー」として数え直される可能性が高く、**ユーザー数・新規ユーザー数・リピーターの指標は信頼できません。** 比較的信頼できるのは、表示回数と、セッション単位の流入元です。
- GA4 側の「拡張計測機能」（scroll、外部リンクのクリックなど）は、データストリームの設定で有効にした場合に gtag.js が自動で送るイベントです（[拡張計測機能](https://support.google.com/analytics/answer/9216061?hl=en)）。Zenn の構成でこれらが実際に送られるかは確かめていません（未確認）。この機能には「ブラウザの履歴イベントに基づくページ変更」で page_view を送る設定もあるため（同上）、Zenn が自前で送る page_view と重複して数えられるかどうかも未確認です。
- `page_path` は GA4 の標準パラメータではありません（GA4 の page_view は `page_location` が基本です）。GA4 のレポートでページパスがどう表示されるかは、実際のプロパティで確かめる必要があります（推論・未確認）。

### 記事ごとに GA4 で見られるもの（Google 公式の定義）

- **表示回数・ページ別の閲覧状況**: 「ページとスクリーン」レポートでは、セッションの最初のページかどうかに関係なく、閲覧されたページのデータを見られます（[ページとスクリーン レポート](https://support.google.com/analytics/answer/12926732?hl=en)）。
- **ランディングページ**: 「ランディングページ」レポートでは、訪問者が最初に着地したページとその人数を見られます。セカンダリディメンションに「セッションの参照元/メディア」を追加すると、ランディングページごとの流入元が分かります（[ランディングページ レポート](https://support.google.com/analytics/answer/12931766?hl=en)）。
- **流入元**: 「トラフィック獲得」レポートはセッション単位で、新規・リピートのセッションがどこから来たかを示します（[トラフィック獲得 レポート](https://support.google.com/analytics/answer/12923437?hl=en)）。
- **チャネルの定義**: Organic Search は「オーガニック検索結果の広告以外のリンクから来た」流入で、参照元が検索サイトのリストに合致するか、メディアが `organic` のものです。Organic Social はソーシャルサイトの広告以外のリンク、Referral は他サイトのリンク、Direct は参照元が `(direct)` のものです（[デフォルト チャネル グループ](https://support.google.com/analytics/answer/9756891?hl=en)）。
- 国、デバイス、エンゲージメント時間などは GA4 の標準ディメンションと指標で見られます。ただし、cookie を 1 日で切る Zenn の構成で、エンゲージメント時間などがどこまで正確に取れるかは検証していません（未確認）。

### 既知・推定される制約

- **計測されるのは自分の `zenn.dev/ユーザー名` 以下のページだけです**（[Zenn リリース 2020-11-19](https://info.zenn.dev/2020-11-19)）。Zenn のトップやトピック一覧など、他の Zenn ページの閲覧は自分の GA には入りません。
- **Zenn 内からの流入の帰属について（推論・未確認）**: Zenn は SPA なので、読者が Zenn トップから自分の記事へ画面遷移すると、そのときに初めて自分の GA タグが初期化されます。このときセッションの参照元が `zenn.dev` の Referral になるのか、Direct や元の外部参照元になるのかは、コードを読むだけでは判断できません。Zenn 内の回遊（Trending やトピックページ経由）を GA4 で正確に切り分けられる保証はありません。

---

## 3. Search Console を Zenn の記事に使えるか

### Zenn 側の状況

- Zenn 公式の「使い方」一覧（[zenn.dev/manual](https://zenn.dev/manual)）、FAQ（[zenn.dev/faq](https://zenn.dev/faq)）、お知らせ・リリース一覧（[info.zenn.dev](https://info.zenn.dev/)）を確認しましたが、**Search Console の連携や所有権確認に関する記述は見つかりませんでした。** 外部サービスとの計測連携として公式に案内されているのは、Google Analytics（個人と Publication Pro）だけです。
- 記事ページの `<head>` に、著者が任意のタグ（`google-site-verification` など）を入れる手段は、公式ドキュメントでは案内されていません。

### Google 側の要件と照らし合わせ

- `zenn.dev/ユーザー名/` のようなパスを範囲にするには **URL プレフィックスプロパティ**を使います（ドメインプロパティはパスを指定できません）。URL プレフィックスプロパティは「このプレフィックスで始まる URL すべて」を含みます（[プロパティの追加](https://support.google.com/webmasters/answer/34592?hl=en)）。
- 確認方法とそれぞれの前提（[所有権の確認](https://support.google.com/webmasters/answer/9008080?hl=en)）を、Zenn の状況と並べると次のとおりです。
  - **HTML ファイル**: サイトのルートディレクトリにファイルをアップロードできることが前提です。Zenn のユーザーにはできません。
  - **HTML タグ**: ホームページの `<head>` にタグを挿入できることが前提です。Zenn のユーザーにはできません。
  - **Google アナリティクス**: ページの `<head>` に analytics.js か gtag.js のスニペットがあり、それが「提供されたとおり、変更せずに」入っていることと、GA で編集権限を持つことが前提です。Zenn の `<head>` には gtag.js の読み込みはありますが、ユーザーの測定ID を指定する `config` は表示から 1 秒後に JS から実行されます（実地確認）。推論すると、この方式では確認が通らない可能性が高いです。実際には試していません（未確認）。
  - **Google タグ マネージャー**: 自分がコンテナの公開・管理権限を持ち、そのコードがページに入っていることが前提です。Zenn のユーザーにはできません。
  - **DNS（ドメイン名プロバイダ）**: zenn.dev の DNS を編集できることが前提です。Zenn のユーザーにはできません。

### GA4 単体ではキーワードが分からないことの確認

- GA4 で「検索クエリ」と関連指標を見られるのは、**Search Console プロパティをリンクした場合の「オーガニック検索クエリ」レポート**です。リンクを作るには GA プロパティの編集者ロールと、**Search Console プロパティの確認済み所有者**であることが必要です。ウェブのデータストリームにリンクできる Search Console プロパティは 1 つだけです。また Search Console の指標は、Search Console のディメンションと、GA のランディングページ、デバイス、国としか組み合わせられません（[GA4 Search Console 連携](https://support.google.com/analytics/answer/10737381?hl=en)）。
- Search Console の検索パフォーマンスレポートでは、クリック数、表示回数、CTR、平均掲載順位を、クエリ、ページ、国、デバイス、検索での見え方、日付の切り口で見られます（[検索パフォーマンス レポート](https://support.google.com/webmasters/answer/7576553?hl=en)、[Search Console の概要](https://developers.google.com/search/docs/monitor-debug/search-console-start)）。
- 「GA4 の標準レポートにはオーガニック検索キーワードがない」と明言した Google 公式ページは、今回の調査では見つかりませんでした。ただし上の連携ヘルプが、クエリデータの出どころを Search Console として説明しています。GA4 単体でクエリ単位の評価はできない、という結論は一次情報と矛盾しません（明言している一次情報は未確認）。

---

## 4. Search Console なしで「狙ったキーワードで届いているか」を見る代替手段

一次情報で裏付けられるもの:

- **GA4 のランディングページ × セッションの参照元/メディア**: 記事ごとに、Organic Search（`google / organic` など）から何セッション入ってきたかを見られます（[ランディングページ レポート](https://support.google.com/analytics/answer/12931766?hl=en)、[チャネル定義](https://support.google.com/analytics/answer/9756891?hl=en)）。分かるのは「検索から来ているか」までで、「どの語で来たか」は分かりません。
- **`site:` 演算子**: 指定したドメインや URL プレフィックスの検索結果を出せます。ただし、インデックス済みの URL がすべて返るとは限らず、クエリなしの `site:` 検索は順位付けもしません（[site: 演算子](https://developers.google.com/search/docs/monitor-debug/search-operators/all-search-site)）。インデックスされているかどうかの大まかな確認にとどまります。
- **手動検索**: 検索結果は時間、場所、デバイス、検索履歴によって変わります（[検索パフォーマンス レポート](https://support.google.com/webmasters/answer/7576553?hl=en)）。自分で検索して見た順位は、ほかの人が見る順位と同じとは限りません。
- **変更の効果が出るまでの時間**: 変更によっては数時間で反映されますが、数か月かかるものもあります。効果を判断するまで数週間待つよう案内されています（[SEO スターターガイド](https://developers.google.com/search/docs/fundamentals/seo-starter-guide)）。タイトルリンクへの反映には数日から数週間かかります（[タイトルリンク](https://developers.google.com/search/docs/appearance/title-link)）。

推論（一次情報で裏付けた運用手順ではありません）:

- 公開前に狙う語を決めておき、公開から数週間後に、GA4 で「その記事の Organic Search セッション数」と、シークレットウィンドウでの手動検索の結果を記録する、という形が現実的です。Organic Search がほぼゼロなら、タイトルと冒頭段落を見直します。それでも動かなければ、狙う語自体を見直します。
- 個人用計測ではユーザー数の指標が信頼できないため（第2節）、比べるときは「表示回数」や「セッション数」を使います。

---

## 5. 公開後に変更できるもの・Zenn が出力するメタ情報

### 変更の可否

- **記事の更新**: Markdown ファイルを編集してプッシュすれば反映されます。slug が同じでないと別の記事として作成されます（[Zenn CLI ガイド「記事の更新」](https://zenn.dev/zenn/articles/zenn-cli-guide)）。Front Matter の `title`、`emoji`、`topics`、`type` を公開後に変えてはいけないという記述はないので、更新の手順で変えられると読めます。ただし「タイトルを変更できます」と明言した記述も見つかりませんでした。
- **トピック**: 5 つまで指定できます（同上）。
- **slug**: 「slug は一度 zenn.dev 上で作成されたら変更できません（slug を変えると別の投稿として作成されます）」と書かれています。形式は半角英小文字、数字、ハイフン、アンダースコアの 12〜50 字です（[slug とは](https://zenn.dev/zenn/articles/what-is-slug)）。URL に検索キーワードを含めたい場合は、**公開前に**決めておく必要があります。
- **公開日時（`published_at`）**: 指定できるのは一度だけで、設定済みの値は変更できません（[Zenn CLI ガイド](https://zenn.dev/zenn/articles/zenn-cli-guide)）。
- **反映の遅延**: zenn.dev への反映には時間がかかります。ログインしていない状態では、古いキャッシュがしばらく表示されることがあります（同上）。
- **更新日の表示**: 記事ページには公開日（`publishedAt`）と並んで本文の更新日（`bodyUpdatedAt`）が表示されます（実地確認：[slug とは](https://zenn.dev/zenn/articles/what-is-slug) のページで「2020/09/06 に公開」と「2022/03/16」が表示されていました）。どの変更が更新日に反映されるか（Front Matter だけを変えた場合など）は未確認です。

### 記事 HTML の SEO 関連要素（実地確認：2 本の公開記事の SSR HTML）

- `<title>`: 記事タイトルそのまま（「| Zenn」などの接尾辞はありません）
- `<link rel="canonical">`: 記事 URL（`https://zenn.dev/<user>/articles/<slug>`）
- `og:title`: 記事タイトル、`og:url`: 記事 URL、`og:type`: article、`og:site_name`: Zenn、`og:image`: タイトル入りの自動生成画像
- `h1`: 記事タイトル
- **`meta name="description"` と `og:description` はありません。** 代わりに `meta name="zenn:description"` があり、中身は「〇〇さんによる記事」という固定の文言です。
- 構造化データ（`application/ld+json`）はありません。

ここから言えること:

- タイトルは `<title>`、`og:title`、`h1` のすべてに使われます。Google はこれらをタイトルリンクの材料にしています（[タイトルリンク](https://developers.google.com/search/docs/appearance/title-link)）。**検索向けに効く主な変更点はタイトルです。**
- meta description を書く場所がないので、スニペットは本文から作られる前提になります（Google はスニペットを「主にページのコンテンツ自体から」作ります。[スニペット](https://developers.google.com/search/docs/appearance/snippet)）。冒頭で記事の要点と狙う語に触れておくのが妥当です（推論）。

---

## 6. 公開前のキーワード調査に使える一次情報のツール

### Google Trends

- 各データポイントを、その地域と期間の総検索数で割ったうえで、0〜100 に正規化して表示します。値は相対的な人気度で、検索回数そのものではありません。データは Google 検索のサンプルで、人気のある語のデータしか表示されず、検索数の少ない語は 0 になります。統計的なノイズも含まれます（[Google Trends データの FAQ](https://support.google.com/trends/answer/4365533?hl=en)）。
- 推論すると、Trends は「候補の語どうしの相対比較」や「季節変動・上昇傾向の確認」に向いていて、絶対的な検索数の見積もりには向きません。

### Google 広告 キーワードプランナー

- キーワード候補の提案、月間検索数の推定、入札単価の推定、予測を提供します。Google 広告の「ツール」メニューから使い、キーワード候補などの基本機能を使うには、支払い情報を入力してアカウントの設定を完了する必要があります。検索数が非常に少ない語やセンシティブな語は表示されません（[キーワードプランナーの使い方](https://support.google.com/google-ads/answer/7337243?hl=en)）。
- 「月間平均検索数」は、キーワードとその類似パターンが検索された平均回数で、値は丸められています（[キーワードプランナーの予測について](https://support.google.com/google-ads/answer/3022575?hl=en)）。
- Google の案内ページでは「無料のリソース」と紹介されています（[Google 広告：キーワードプランナーの使い方](https://business.google.com/us/resources/articles/using-google-ads-keyword-planner/)）。
- 広告を出稿していない、または出稿額が少ないアカウントでは検索数が「1万〜10万」のような範囲で表示される、という説明は Google のヘルプでは確認できませんでした（未確認）。

### Zenn 側のシグナル

- トピックページ（例: [zenn.dev/topics/typescript](https://zenn.dev/topics/typescript)）には、そのトピックの Articles、Books、Scraps の件数と、Trending、Alltime、Recent のタブが表示されます（実地確認：調査時点で TypeScript は Articles 14279 件）。人気トピックの一覧は [zenn.dev/topics](https://zenn.dev/topics) にあります。トピックはフォローできます（[Zenn リリース 2024-06-03](https://info.zenn.dev/2024-06-03-topic-following-available)）。フォロワー数が表示されるかどうかは、SSR の HTML では確認できませんでした（未確認）。
- Trending は「さまざまな指標によるスコアリング」で決まり、Like の数が多くても載るとは限りません。詳しいロジックは公開されていません（Zenn FAQ「私の記事が Trending に表示されない理由は？」。[zenn.dev/faq/why-not-in-trending](https://zenn.dev/faq/why-not-in-trending)。ページの本文は JS で描画されるため、FAQ ページのバンドルに入っている本文で内容を確認しました）。
- 2026-05-27 にサイト内検索へ「意味検索」が追加されました（[Zenn リリース 2026-05-27](https://info.zenn.dev/2026-05-27-semantic-search)）。Zenn 内の検索で見つかるかどうかは、Google 検索とは別の経路です。

### Google Search Central の指針（要点）

- 「ユーザーがあなたのコンテンツを見つけるために検索しそうな語を考える」。初心者と専門家では使う語が違うことがあります。Google は語が完全に一致しなくてもページとクエリを結びつけられます（[SEO スターターガイド「読者の検索語を予測する」](https://developers.google.com/search/docs/fundamentals/seo-starter-guide)）。
- 良いタイトルは、ページ固有で、明確かつ簡潔で、内容を正確に表すものです（同上）。キーワードを繰り返し詰め込むのは避けます（[タイトルリンク](https://developers.google.com/search/docs/appearance/title-link)）。
- Google は keywords メタタグを使いません。同じ語を過度に繰り返すと、スパムに関するポリシーに抵触し得ます（[SEO スターターガイド](https://developers.google.com/search/docs/fundamentals/seo-starter-guide)）。
- 良い meta description は短く、ページ固有で、要点を含むものです（同上）。ただし Zenn では設定できません（第5節）。

---

## 未確認事項

1. **GA 方式で Search Console の所有権確認が通るか**: Zenn は gtag.js を `<head>` で読み込んでいますが、測定ID の `config` は JS で後から実行されます。Google の要件（スニペットを変更せずに `<head>` に置く）を満たさない可能性が高いと推論していますが、試していません。
2. **GA4 の ID に対応したことの Zenn 公式告知**: 告知は見つかりませんでした。公開ページで `G-` の ID が使われていることだけを確認しています。
3. **アカウント設定画面の具体的な操作手順と表示文言**: ログインが必要なため、公式の画面は確認していません。
4. **本・スクラップページでの計測**: リリース文の範囲からすると対象ですが、コードで確認できたのは記事ページとプロフィールページだけです。
5. **Publication に紐づけた記事で、著者個人の GA にも送信されるか**: コード上はそう読めますが、実際の送信は確認していません。
6. **拡張計測機能（scroll など）が送られるか、履歴変更による page_view が重複して数えられるか**: 未検証です。
7. **`page_path` パラメータが GA4 のレポートでどう扱われるか**: 未検証です。
8. **Zenn 内の回遊（Trending やトピックページ経由）が GA4 でどの参照元として記録されるか**: 未検証です（SPA 遷移のため Referral になるとは限りません）。
9. **cookie の有効期限 1 日がユーザー数などの指標に与える実際の影響の大きさ**: コード上の事実と Google の既定値（2 年）との比較からの推論です。
10. **「GA4 の標準レポートにはオーガニック検索キーワードがない」と明言した Google 公式ページ**: 見つかりませんでした（Search Console 連携ヘルプから間接的に支持される、という状態です）。
11. **キーワードプランナーで検索数が範囲表示になる条件（出稿の有無や額）**: Google のヘルプでは確認できませんでした。
12. **トピックページにフォロワー数が表示されるか**: SSR の HTML では確認できませんでした。
13. **Front Matter だけを変更した場合に本文の更新日が変わるか**: 未確認です。
14. **タイトル変更の可否の明示**: 更新手順上は変更できると読めますが、「タイトルを変更できる」と明言した公式記述は見つかりませんでした。
