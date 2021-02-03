# 4 days Node.js / day02

## 今回のゴール

- Web スクレイピングの仕組みや注意点を理解する．
- 複数の方法でスクレイピングを体験し，使い方のイメージを掴む．
- 取得したデータの処理を行い，配列力を高める．

## Web スクレイピングとは

> ウェブスクレイピング（英: Web scraping）とは、ウェブサイトから情報を抽出するコンピュータソフトウェア技術のこと。ウェブ・クローラー[1]あるいはウェブ・スパイダー[2]とも呼ばれる。 通常このようなソフトウェアプログラムは低レベルの HTTP を実装することで、もしくはウェブブラウザを埋め込むことによって、WWW のコンテンツを取得する。
> (Wikipedia より)

## Web スクレイピングを行うときの注意点

スクレイピングの対象となる Web サイトには著作権が存在する．また，利用規約が存在する場合もあるため，対象の Web サイトでこれらの記述内容を確認する必要がある．

また，対象となる Web サイトのサーバに過度な負担をかけることも問題となる．過度なスクレイピングを行ってサービスが停止し，逮捕された事件も存在する．

[参考記事](https://qiita.com/nezuq/items/c5e827e1827e7cb29011)

コードを書く際には，ターゲットとなる Web サイトの HTML 構造をよく把握することが大切である．取り出したい情報がどの要素に含まれているのかを知っておく必要がある．

当然，Web サイトが更新されたタイミングでタグ構成などが変更された場合は動作しないのでソースコードの見直しが必要になる．

## スクレイピング初級編（HTML ソースの取得）

スクレイピングには複数の方法があるが，まずはターゲットとなる Web サイトの HTML ソースコードを取得する方法を実装する．

### 処理の流れと必要なファイルの準備

下記のようにファイルを作成する．

```bash
.
├── app.js
├── controllers
│   ├── janken.controller.js
│   ├── omikuji.controller.js
│   └── scraping.controller.js
├── package.json
├── package-lock.json
├── routes
│   ├── janken.route.js
│   ├── omikuji.route.js
│   └── scraping.route.js
└── services
    ├── janken.service.js
    ├── omikuji.service.js
    └── scraping.service.js
```

それぞれのファイルについて，下記のようにコードを編集する．

```js
// app.js
const express = require("express");
const app = express();

const bodyParser = require("body-parser");
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

const port = 3001;
const omikujiRouter = require("./routes/omikuji.route");
const jankenRouter = require("./routes/janken.route");
const scrapingRouter = require("./routes/scraping.route");

app.get("/", (req, res) => {
  res.json({
    uri: "/",
    message: "Hello Node.js!",
  });
});

app.use("/omikuji", (req, res) => omikujiRouter(req, res));
app.use("/janken", (req, res) => jankenRouter(req, res));
app.use("/scraping", (req, res) => scrapingRouter(req, res));

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

```js
// routes/scraping.route.js
const express = require("express");
const router = express.Router();

const ScrapingController = require("../controllers/scraping.controller");

router.get("/gethtml", (req, res) => ScrapingController.getHtml(req, res));

module.exports = router;
```

```js
// controllers/scraping.controller.js
const ScrapingService = require("../services/scraping.service");

exports.getHtml = async (req, res, next) => {
  try {
    const result = await ScrapingService.getHtml(req.query);
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully get HTML!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};
```

```js
// services/scraping.service.js
exports.getHtml = async (query) => {
  try {
    return query;
  } catch (e) {
    throw Error("Error while getting HTML.");
  }
};
```

今回は`GET`メソッドでリクエストを行っている．

各ファイルに，他のレイヤーと連携する処理を記述し，`query`の取得と固定のレスポンスでデータの受け渡しを行っている．

### 動作確認

下記コマンドで動作確認する．下記のようにレスポンスが返ってくれば OK．

```bash
$ curl localhost:3001/scraping/gethtml?message=test

{
  "status": 200,
  "result": {
    "message": "test"
  },
  "message": "Succesfully get HTML!"
}
```

### HTML ソースを取得する処理

ここから，実際に HTML ソースを取得する処理を実装する．方針は以下の通り．

- ユーザはリクエストに「ターゲットとなる URL」を`url=https://example.com`の形式で含めて送信する．
- サーバ側で，該当する URL のソースコードを取得し，レスポンスに含めて返却する．
- 要はいつもブラウザでやっていることを Node.js でやっているだけである．

Node.js から http リクエストを行うため，`node-fetch`パッケージをインストールする．

http リクエストを行う方法は複数あるが，多分これが最も簡単．JavaScript の http リクエストである`fetch`関数を Node.js でも使えるようにしたもの．

```bash
$ npm i node-fetch
```

実行結果

```bash
+ node-fetch@2.6.1
added 1 package from 1 contributor and audited 51 packages in 1.796s
found 0 vulnerabilities
```

一連の流れはサービスに記述しよう．

```js
// services/scraping.service.js
const fetch = require("node-fetch");

exports.getHtml = async (query) => {
  // `fetch`関数でhttpリクエストを送信
  const response = await fetch(query.url);
  // `text`で本文取得
  const html = await response.text();
  try {
    return html;
  } catch (e) {
    throw Error("Error while getting HTML.");
  }
};
```

リクエストを送って実行してみる（URL は自由で OK）．

```bash
$ curl localhost:3001/scraping/gethtml?url=https://gsacademy.jp/fukuoka/

{
  "status": 200,
  "result": "ここにHTMLのソースコードが入る",
  "message": "Succesfully get HTML!"
}
```

ソースコードを取得できれば成功だ．

しかし，これではソースコード全体を取得しただけなので，必要な情報をうまく利用することは難しい．

そこで，次は必要な情報のみ利用するような処理を実装する．

### ソースから必要な情報を抜き出す

HTML から必要な情報を抜き出す場合は，まずターゲットとなる Web サイトの HTML 構造をよく確認する必要がある．

今回の処理では予めターゲットとなる Web サイトを決めておく．今回は`https://gigazine.net/`からニュースのタイトルと個別記事の URL を取得する．

ルートとコントローラのファイルに以下の内容を追記する．

```js
// routes/scraping.route.js
const express = require("express");
const router = express.Router();

const ScrapingController = require("../controllers/scraping.controller");

router.get("/gethtml", (req, res) => ScrapingController.getHtml(req, res));
// ↓追加
router.get("/get-recent-contents", (req, res) =>
  ScrapingController.getRecentContents(req, res)
);

module.exports = router;
```

```js
// controller/scraping.controller.js
const ScrapingService = require("../services/scraping.service");

exports.getHtml = async (req, res, next) => {
  // 省略
};

// ↓追加
exports.getRecentContents = async (req, res, next) => {
  try {
    const result = await ScrapingService.getContents({});
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully get recent contents!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};
```

サービスのファイルに，コンテンツを取得する処理を記述する．流れは以下の通り．

- 前項と同様に HTML のソースコードを取得する．
- 必要なデータ（今回はニュースの URL とタイトル）の HTML 構造を確認しておき，必要な部分のみ抜き出す．

文字列抽出の流れは以下の通り．一つずつ出力しながら確認する．

- 今回は個別のニュースコンテンツが`<h2>`で表記されているのでここからスタートする．
- 正規表現を用いて，HTML のソースコード（文字列）から`<h2><a`で始まって`</h2>`で終了する文字列を配列として抽出する（[参考](https://iwb.jp/javasctipt-html-regexp-match/)）．
- 必要なタグ部分の配列を得られたら，`map()`メソッドを用いて各配列要素の URL とタイトルを抽出する（このあたりはただの配列処理）．
- 今回は`split()`を用いたが，他にも方法はあるので好みで問題ない．
- URL とタイトルの配列が得られたら，最後に情報が欠けているものを`filter()`で除外している．

```js
// services/scraping.service.js
const fetch = require("node-fetch");

exports.getHtml = async (query) => {
  // 省略
};

// ↓追加
exports.getRecentContents = async (query) => {
  try {
    const url = "https://gigazine.net/";
    const response = await fetch(url);
    const html = await response.text();
    const contentsElement = html.toString().match(/<h2><a(?: .+?)?>.*?<\/h2>/g);
    const contentsArray = contentsElement
      .map((x) => {
        return {
          url: x.split(" ")[1].split("=")[1].split('"')[1],
          title: x.split("<span>")[1]?.split("</span>")[0],
        };
      })
      .filter((x) => x.url && x.title);
    return contentsArray;
  } catch (e) {
    throw Error("Error while getting recent contents.");
  }
};
```

リクエストを送って動作確認する．URL とタイトルを含んだ JSON データが得られれば OK．ブラウザで確認しても同様の結果を得られる．

```bash
$ curl localhost:3001/scraping/get-recent-contents

{
  "status": 200,
  "result": [
    {
      "url": "https://gigazine.net/news/20210203-instagram-recently-deleted/",
      "title": "Instagramが削除された写真を簡単に復活できる「Recently Deleted」機能をスタート"
    },
    {
      "url": "https://gigazine.net/news/20210203-coronavirus-variant-new-mutation/",
      "title": "感染力の強い新型コロナウイルスの英国型変異株がワクチンの有効性を低下させる変異を獲得しつつある"
    },
    "...以下たくさん"
  ],
  "message": "Succesfully get recent contents!"
}
```

これで HTML から必要な部分のみ抽出して JSON データを返す API が完成した．

ターゲットとなる Web サイトによって HTML 構造が異なるため，ブラウザの検証ツールなどを用いて構造を確認しながら実装することが大事である．

## スクレイピング中級編（ユーザ操作の自動化）

前項で Web サイトから必要な情報を抽出し，JSON データにまとめる実装を行った．

しかし，はじめから必要なデータが HTML 上に揃っている状況ばかりではない．例えば，ユーザ操作によって追加の情報が表示される場合はどうだろうか．

このような場合は，Node.js 上からブラウザを操作し，ページの状態を更新しなければならないため，別のアプローチでスクレイピングを行う必要がある．

今回は Node.js のパッケージである`puppeteer`を用いて，ブラウザの操作を自動化する処理を実装する．

`puppeteer`は，ヘッドレスブラウザを内蔵しており，Node.js からブラウザを起動して，決められた操作を実行することができる．今回のようなスクレイピング以外にも，プロダクトの E2E テストなどにも利用可能だ．

### パッケージの準備

`puppeteer`をインストールする．

```bash
$ npm i puppeteer

+ puppeteer@6.0.0
added 59 packages from 75 contributors and audited 110 packages in 50.827s

8 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

### 必要なファイルの準備

ルートとコントローラに必要な処理を記述する．

ルートとコントローラのファイルに以下の内容を追記する．

```js
// routes/scraping.route.js
const express = require("express");
const router = express.Router();

const ScrapingController = require("../controllers/scraping.controller");

router.get("/gethtml", (req, res) => ScrapingController.getHtml(req, res));
router.get("/get-recent-contents", (req, res) =>
  ScrapingController.getRecentContents(req, res)
);
// ↓追加
router.get("/get-many-contents", (req, res) =>
  ScrapingController.getManyContents(req, res)
);

module.exports = router;
```

```js
// controller/scraping.controller.js
const ScrapingService = require("../services/scraping.service");

exports.getHtml = async (req, res, next) => {
  // 省略
};

exports.getRecentContents = async (req, res, next) => {
  // 省略
};

// ↓追加
exports.getManyContents = async (req, res, next) => {
  try {
    const result = await ScrapingService.getManyContents({});
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully get many contents!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};
```

```js
// services/scraping.service.js
const fetch = require("node-fetch");
// ↓追加
const puppeteer = require("puppeteer");

exports.getHtml = async (query) => {
  // 省略
};

exports.getRecentContents = async (query) => {
  // 省略
};

exports.getManyContents = async (query) => {
  try {
    const url = "https://gigazine.net/";
    return { message: "OK!" };
  } catch (e) {
    throw Error("Error while getting many contents.");
  }
};
```

記述したら動作確認する．`{"message":"OK!"}`が返ってくれば OK．

```bash
$ curl localhost:3001/scraping/get-many-contents

{
  "status": 200,
  "result": {
    "message": "OK!"
  },
  "message": "Succesfully get many contents!"
}
```

### ブラウザの起動とその操作

では，早速 puppeteer を用いてブラウザを操作してみる．

まずは，ブラウザを開いて指定したページにアクセスし，ブラウザを閉じる動作を実装する．

細かい処理内容はコメント参照．

```js
// services/scraping.service.js
exports.getManyContents = async (query) => {
  try {
    const url = "https://gigazine.net/";

    // `headless`を`true`にするとヘッドレスモードで動作．本番は`true`にしておこう
    // `devtools`を`true`にすると検証画面を開いた状態で動作．
    // 実行時の動作が速くて確認できないときは`slowMo`を設定するとゆっくりになる．
    const options = {
      headless: false,
      devtools: true,
      slowMo: 500,
      args: ["--no-sandbox"],
    };

    // ブラウザを立ち上げる
    const browser = await puppeteer.launch(options);

    // ブラウザのタブを開く
    const page = await browser.newPage();

    // ユーザーエージェントを適当に設定
    await page.setUserAgent(
      `Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36`
    );

    // リクエストのフィルタリングを有効化する
    await page.setRequestInterception(true);

    // リクエストイベントを受け取る
    page.on("request", (request) => {
      if (["image", "font", "script"].indexOf(request.resourceType()) !== -1) {
        // 重いやつらはリクエストをブロックする
        request.abort();
      } else {
        // その他の要素はリクエストを許可する
        request.continue();
      }
    });

    // url指定して移動
    await page.goto(url);

    // ブラウザを閉じる
    await browser.close();

    return { message: "OK!" };
  } catch (e) {
    throw Error("Error while getting many contents.");
  }
};
```

動作を確認する．コマンドを実行すると，自動的にブラウザが立ち上がり，読み込んだ後閉じられる．

レスポンスは前項から変化なし．

```bash
$ curl localhost:3001/scraping/get-many-contents

{
  "status": 200,
  "result": {
    "message": "OK!"
  },
  "message": "Succesfully get many contents!"
}
```

### ユーザ操作の自動化

ブラウザの起動ができたら操作を自動化してみる．

今回の web ページでは，画面下部の「さらに前の記事を見る >>」部分をクリックすることで，次のコンテンツのページにアクセスすることができる．

そこで，この部分をクリックしてページを移動する動作を自動化する．最初のページを 1 ページめとして，3 ページ目まで移動したらブラウザを閉じることにする．

```js
// services/scraping.service.js
// url指定して移動
await page.goto(url);

// ↓ここから追加
const pages = 3;
for (let i = 0; i < pages; i++) {
  // 必要なコンテンツが表示されるまで待つ
  await page.waitForTimeout(() => {
    // 画面下部までスクロールする（今回はしなくてもいい）
    window.scrollTo(0, document.body.scrollHeight);
    const contents = document.querySelectorAll("h2");
    return contents.length > 0;
  });
  // 最終ページ以外なら次へボタンをクリック
  if (i < pages - 1) {
    await Promise.all([
      page.waitForNavigation({ timeout: 20000, waitUntil: "domcontentloaded" }),
      page.click("#nextpage"),
    ]);
  }
}
// ↑ここまで追加

// ブラウザを閉じる
await browser.close();
return { message: "OK!" };
```

### データ取得と整形

ページの移動ができたので，移動する毎に必要な情報を取得してみる．

今回も個別記事のタイトルと URL を取得するため，`<h2>`要素の中の`<a>`要素を指定して中身を取得する．

要素の指定は`querySelectorAll`などと同様なので，わかりやすい（↓ コードの変数`query`部分）．

`$$eval()`を実行すると`query`に該当する要素が`anchors`に配列形式で入ってくる．あとはただの配列処理だ．

各ページのコンテンツを配列にまとめ，`manyContents`に追加している．`manyContents`は 2 次元配列になるため，`return`時に`flat()`して 1 次元配列に調整している．

```js
// services/scraping.service.js
// url指定して移動
await page.goto(url);

const pages = 3;
// ↓追加：取得データ格納用の配列
const manyContents = [];
for (let i = 0; i < pages; i++) {
  await page.waitForTimeout(() => {
    window.scrollTo(0, document.body.scrollHeight);
    const contents = document.querySelectorAll("h2");
    return contents.length > 0;
  });
  // ↓追加：探したい要素の指定
  const query = "h2 a";
  const currentPageData = await page.$$eval(query, (anchors) =>
    anchors
      .map((anchor) => ({
        url: anchor.href,
        title: anchor.title,
      }))
      .filter((x) => x.url && x.title)
  );
  // ↓追加：配列に追加する処理
  manyContents.push(currentPageData);
  if (i < pages - 1) {
    await Promise.all([
      page.waitForNavigation({ timeout: 20000, waitUntil: "domcontentloaded" }),
      page.click("#nextpage"),
    ]);
  }
}
await browser.close();
// ↓追加：取得したデータを返す処理
return manyContents.flat();
```

コマンドを実行して動作確認しよう．コンテンツの情報が返ってくれば OK！

```bash
$ curl localhost:3001/scraping/get-many-contents

{
  "status": 200,
  "result": [
    {
      "url": "https://gigazine.net/news/20210203-nike-go-flyease/",
      "title": "ナイキが手を使わずに着脱できるスニーカー「GO FlyEase」を発表"
    },
    {
      "url": "https://gigazine.net/news/20210203-courts-of-space/",
      "title": "宇宙での紛争を解決するための「宇宙裁判所」設立、裁判官養成や紛争解決ガイドの作成もスタート"
    },
    "...以下たくさん"
  ],
  "message": "Succesfully get many contents!"
}
```

やったね！これでブラウジングの自動化は完璧だ！

本番環境にデプロイするときは`options`の`headless`を`true`にしておこう！

## おわりに

今回は Node.js を用いた Web スクレイピングを実装した．

Web スクレイピングというと自動でブラウザを操作するようなイメージだが，実際の方法は様々である（極論すれば人力のコピペもスクレイピングだ）．

取得したいデータによっては，ブラウザの自動化を行うことなく http リクエストと配列や文字列処理だけで十分実装できる場合もある．

Web サイトの構成が変更され，処理内容を見直すことが必要な場合もある．そのため，目的に応じてできるだけ容易な実装を選択することが大切だ．

（利用規約や法律など気をつけつつ）良きスクレイピングライフを！

今回は以上である( `･ω･)b
