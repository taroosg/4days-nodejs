# 4 days Node.js / day01

## 今回のゴール

- Node.js と Express で簡単な API を体験する．
- 実装のしかたを複数学び，見通しの良いコードの構造を把握する．
- サーバで動作する「おみくじ」と「じゃんけん」を実装する．

## Node.js とは

サーバで JavaScript を動かす環境のこと．
「Node.js」という言語があるわけではないので認識違いに注意．

## Web アプリケーションの仕組み（API を実装する場合）

基本は「リクエスト」と「レスポンス」．PHP など他の言語と同じ．

サーバ側でリクエストを受ける URI（Uniform Resource Identifier の略．URL とほぼ同義）を用意しておき，そこにクライアント（ブラウザなど）からリクエストを送信する．

リクエストを受けると記述されたコードが実行され，データなどが返される（JSON 形式が多い）．

## Express

Express は Node.js を用いて Web アプリケーションを構築するためのフレームワーク．

Node.js は自由度が非常に高く，そのまま書くと人によって千差万別となる．したがって，Web 開発をするときは Express を使用するのがデファクトスタンダードとなっている状態である（他にもフレームワークは存在するが，Express を基にしたものが多い）．

Express は構造が簡易であり，API の実装も非常に簡単である．今回は Express を使っていくつかの API を実装してみる．

## API 実装の準備

Node.js のプロジェクトを実装し，動かしてみる．

### プロジェクト作成

プロジェクトを作成するには，下記のコマンドを実行する．

```bash
$ mkdir express-project && cd express-project
$ npm init
```

`npm`は`node package module`の略であり，Node.js 上で動くパッケージを管理するツールである．Node.js で開発を行う場合はこれを用いることがほとんどである．

ダイアログが出てくるので，答えていく．全部そのまま Enter で OK．

```bash
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help init` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (express-project)
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
About to write to /home/taroosg/Desktop/express-project/package.json:

{
  "name": "express-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}


Is this OK? (yes)
```

完了したらエディタで開く．

エディタで開いたら`package.json`が作成されているので，中身を確認する．

```json
{
  "name": "express-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

| 項目        | 意味合い                                                                   |
| ----------- | -------------------------------------------------------------------------- |
| name        | プロダクト名．                                                             |
| version     | プロダクトのバージョン．                                                   |
| description | プロダクトの説明．                                                         |
| main        | プロダクトをパッケージとして公開する場合に入り口となるファイルを指定する． |
| script      | 開発者が任意に作成するコマンド．                                           |
| auther      | 開発者情報．1 人のみ記述する．                                             |
| license     | ライセンス情報．                                                           |

### Express のインストール

今回は Express のフレームワークを使用するため，下記コマンドで

```bash
$ npm i express
```

実行結果

```bash
+ express@4.17.1
added 50 packages from 37 contributors and audited 50 packages in 1.687s
found 0 vulnerabilities
```

完了したら，再度`package.json`を開き，以下の内容が追加されていることを確認する．

```json
"dependencies": {
  "express": "^4.17.1"
}
```

| 項目         | 意味合い                                   |
| ------------ | ------------------------------------------ |
| dependencies | このプロダクトが依存するパッケージの一覧． |

`dependencies`の中にインストールしたパッケージが追加される仕組み．パッケージとはいろいろな機能をまとめてインストールできるようにしたものであり，様々なものが公開されている．ライブラリとほぼ同義．

また，インストールされたパッケージは同時に作成される`node_modules`ディレクトリ内に保存される．このディレクトリはパッケージ専用なので，自分で触ることは殆ど無い．

`package-lock.json`には実際にインストールしたパッケージのバージョンが記載される．基本的に触らない．

`package.json`にはプロダクトに必要なパッケージがすべて記載される．コマンド`npm install`を実行すると，ここに記載されたパッケージがすべてインストールされる．

### `.gitignore`ファイルの作成

ソースコードを Git で管理することは周知のことであるが，プロジェクト内には Git で管理したくないファイルやディレクトリも存在する．

例えば，上記で作成された`node_modules`ディレクトリにはインストールしたパッケージだけでなく，「パッケージが必要としている別のパッケージ」もインストールされる．そのため，これらすべてを Git で管理するとファイル数が膨大になってしまう．

そのため，`node_modules`ディレクトリ（とその他 Git 管理したくないファイル）を`.gitignore`と呼ばれるファイルにリストアップすることで Git の管理外に措くことができる．

エディタで`.gitignore`を作成する．必ずプロジェクト直下に作成すること．

作成したら以下の内容を記述しよう．`.gitignore`から見た相対パスで Git 管理外にしたいファイルやディレクトリを指定している．

```txt
/node_modules
```

ここから追記

## API の実装 ①（おみくじ初級編）

### 実行用ファイルの作成

実行するための`app.js`ファイルを作成する．エディタから作成すれば OK．

ファイルを作成したら下記の内容を記述する．

```js
const express = require("express");
const app = express();
const port = 3001;

app.get("/", (req, res) => {
  res.send("Hello Node.js!");
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

上記のコードは`app.js`が実行されると，`http://localhost:3001`でサーバが立ち上がることを示している．

また，`http://localhost:3001/`に`GET`でリクエストが来ると，`Hello Node.js!`というレスポンスを返すよう記述している．

### 動作確認

上記を記述したら，ターミナルで以下のコマンドを実行し，サーバを起動する．実行する際には作業ディレクトリに移動しておくこと．

```bash
$ node app.js
```

実行結果．下記のようにサーバが立ち上がれば成功．

```bash
Example app listening at http://localhost:3001
```

立ち上がったら，別のターミナルで下記コマンドを実行し，リクエストとレスポンスが適切に処理されることを確認する．

コマンドを実行してメッセージが返ってくれば成功．

```bash
$ curl localhost:3001
Hello Node.js!
```

`curl`はターミナルから http リクエストを送るコマンド．インストールされていない場合はインストールしておく．

サーバを終了する場合は`ctrl + c`で終了できる．

### URI の追加

`/`以外にも URI と作成してみる．

`app.js`を以下のように編集する．`/omikuji`，`/janken`の 2 つの URI を追加し，各レスポンスを JSON 形式で返すよう変更している．

```js
const express = require("express");
const app = express();
const port = 3001;

// 編集
app.get("/", (req, res) => {
  res.json({
    uri: "/",
    message: "Hello Node.js!",
  });
});

// 追加
app.get("/omikuji", (req, res) => {
  res.json({
    uri: "/omikuji",
    message: "This is Omikuji URI!",
  });
});

// 追加
app.get("/janken", (req, res) => {
  res.json({
    uri: "/janken",
    message: "This is Janken URI!",
  });
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

### 動作確認

記述したら，再度下記コマンドでサーバを立ち上げる．

```bash
$ node app.js
```

別のターミナルから，`curl`コマンドで動作を確認する．

```bash
$ curl localhost:3001/
{"uri":"/","message":"Hello Node.js!"}

$ curl localhost:3001/omikuji
{"uri":"/omikuji","message":"This is Omikuji URI!"}

$ curl localhost:3001/janken
{"uri":"/janken","message":"This is Janken URI!"}
```

簡単であるがこれだけで API を実装することができた．

### 【演習】おみくじ処理の追加

実際におみくじの処理を追加してみよう．

`app.js`の以下の部分におみくじの処理を書いて動作を確認しよう．サーバを起動し，以下のような結果になるように実装する．何回か実行して異なる結果が返ってくれば OK！

```bash
$ curl localhost:3001/omikuji
{"uri":"/omikuji","message":"大吉"}

$ curl localhost:3001/omikuji
{"uri":"/omikuji","message":"凶"}

$ curl localhost:3001/omikuji
{"uri":"/omikuji","message":"中吉"}
```

実装例：

```js
// 省略
app.get("/omikuji", (req, res) => {
  const omikuji = ["大吉", "中吉", "小吉", "凶", "大凶"];
  const min = 0;
  const max = omikuji.length - 1;
  const index = Math.floor(Math.random() * (max - min + 1)) + min;
  res.json({
    uri: "/omikuji",
    message: omikuji[index],
  });
});
// 省略
```

### コマンドの追加

ここまで，下記コマンドでサーバを実行していた．

```bash
$ node app.js
```

これとは別に，プロジェクトに対して自分でコマンドを作成することができる．

コマンドを追加する場合は`package.json`に記述する．下記の内容を追記してみよう．

```json
{
  "name": "express-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "__comment": "↓ここを追記（この行は書かなくてOK）",
    "start": "node app.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

このように記述すると，`npm start`というコマンドを実行すると`node app.js`が実行される．

プロジェクトが複雑になると，場面に応じて様々なコマンドを実行したい場合がある．そのようなときに，統一されたコマンドを準備しておくことで開発をスムーズに進めることができる．

下記を実行するとサーバが立ち上がる．これまでと同じ動作であることを確認しよう．

```bash
$ npm start
```

## API の実装 ②（おみくじ中級編）

簡単な API を実装するだけならば，ここまでの内容で十分である．おみくじの処理部分に適当な記述をすれば問題ないであろう．

しかし，コードの記述量が増えてくると，見通しが悪くなってしまい，保守管理にも支障をきたす．

そこで，処理中の役割毎に別ファイルに記述できるようにディレクトリ構成を変更する（責務の分離，などと呼ばれる）．

### ディレクトリ構造と役割

大きく`routes`，`controllers`，`services`の 3 つに分離する．DB などと組み合わせてデータを扱う場合は他に`model`を用意するが今回は省略する．

各要素の役割は以下の通り．

| 項目        | 意味合い                                                                        |
| ----------- | ------------------------------------------------------------------------------- |
| routes      | URI と実行する処理の対応．                                                      |
| controllers | リクエストパラメータの検証，レスポンス送信．                                    |
| services    | リクエストに対する処理のメインロジック．DB 扱う場合はここに CRUD 処理など記述． |

このような役割分担とするため，以下のようにディレクトリとファイルを作成する．

```bash
.
├── app.js
├── controllers
│   └── omikuji.controller.js
├── node_modules
├── package.json
├── package-lock.json
├── routes
│   └── omikuji.route.js
└── services
    └── omikuji.service.js
```

### 各ファイルの実装

`app.js`を以下のように編集する．

ルーティングは routes 以下のファイルに任せ，指定したファイルを読み込むよう記述．

```js
// app.js
const express = require("express");
const app = express();
const port = 3001;
// おみくじのrouterを読み込む
const omikujiRouter = require("./routes/omikuji.route");

app.get("/", (req, res) => {
  res.json({
    uri: "/",
    message: "Hello Node.js!",
  });
});

// おみくじのルーティングを変更
app.use("/omikuji", (req, res) => omikujiRouter(req, res));

app.get("/janken", (req, res) => {
  res.json({ message: "This is Janken URI!" });
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

`routes/omikuji.route.js`に以下の内容を記述する．

URI と対応するコントローラの処理を記述する．

```js
// routes/omikuji.route.js
const express = require("express");
const router = express.Router();

const OmikujiController = require("../controllers/omikuji.controller");

router.get("/", (req, res) => OmikujiController.getResult(req, res));

module.exports = router;
```

`controllers/omikuji.controller.js`に以下の内容を記述する．

サービスを呼び出して実行したいメソッドを指定し，結果によってレスポンスを指定する．

```js
// controllers/omikuji.controller.js
const OmikujiService = require("../services/omikuji.service");

exports.getResult = async (req, res, next) => {
  try {
    const result = await OmikujiService.getOmikuji({});
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully get Omikuji!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};
```

`services/omikuji.service.js`に以下の内容を記述する．

サービスでは実行したいロジックを書く．ここでおみくじのロジックを実装している．

```js
// services/omikuji.service.js
exports.getOmikuji = async (query) => {
  try {
    const omikuji = ["大吉", "中吉", "小吉", "凶", "大凶"];
    const min = 0;
    const max = omikuji.length - 1;
    const index = Math.floor(Math.random() * (max - min + 1)) + min;
    return { result: omikuji[index] };
  } catch (e) {
    throw Error("Error while getting Omikuji.");
  }
};
```

処理の順序としては以下のようになる．

```txt
リクエスト               レスポンス
   ↓                       ↑
routes/omikuji.route.js    |
   ↓                       |
controllers/omikuji.controller.js
   ↓           ↑
ervices/omikuji.service.js
```

### 動作確認

サーバを立ち上げ，以下のコマンドでリクエストを送信する．

```bash
$ curl localhost:3001/omikuji
```

おみくじの結果が返ってくれば成功．数回実行し，異なる結果が返ってくることを確認しておこう．

```bash
$ curl localhost:3001/omikuji
{"status":200,"result":{"result":"凶"},"message":"Succesfully get Omikuji!"}

$ curl localhost:3001/omikuji
{"status":200,"result":{"result":"大凶"},"message":"Succesfully get Omikuji!"}

$ curl localhost:3001/omikuji
{"status":200,"result":{"result":"中吉"},"message":"Succesfully get Omikuji!"}

$ curl localhost:3001/omikuji
{"status":200,"result":{"result":"大吉"},"message":"Succesfully get Omikuji!"}
```

ユーザに返すレスポンスを変更したい場合はコントローラを，処理の内容を更新したい場合はサービスを書き換えれば良い．このように，各責務を別の場所に置くことでメンテナンスしやすいプロダクトになる．

これでおみくじの実装は完了である．

## API の実装 ③（じゃんけん）

おみくじの実装ができたので，ユーザ側からデータを送信してじゃんけんの結果を返す API を実装してみる．

### アプリケーションの全体像

ユーザはリクエスト時に`post`メソッドで自分の手を送信する．サーバ側はユーザから送信された手に対してランダムで手を出し，「ユーザの手」「サーバの手」「勝敗」をレスポンスで返却する．

下図を参考に，必要なファイルを作成する．

```bash
.
├── app.js
├── controllers
│   ├── janken.controller.js
│   └── omikuji.controller.js
├── node_modules
├── package.json
├── package-lock.json
├── routes
│   ├── janken.route.js
│   └── omikuji.route.js
└── services
    ├── janken.service.js
    └── omikuji.service.js
```

### 各ファイルの実装

`app.js`を以下のように編集する．

じゃんけんのルーティングを読み込む．また，POST メソッドでデータを受け取るためには`body-parser`が必要になるため読み込んでいる．

```js
// app.js
const express = require("express");
const app = express();
// ↓POSTでデータを受け取るために必要
const bodyParser = require("body-parser");
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
const port = 3001;

const omikujiRouter = require("./routes/omikuji.route");

// じゃんけんのルーティングを読み込む
const jankenRouter = require("./routes/janken.route");

app.get("/", (req, res) => {
  res.json({
    uri: "/",
    message: "Hello Node.js!",
  });
});

app.use("/omikuji", (req, res) => omikujiRouter(req, res));
// じゃんけんのルーティングを追加
app.use("/janken", (req, res) => jankenRouter(req, res));

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

`routes/janken.route.js`に以下の内容を記述する．

URI と対応するコントローラの処理を記述する．今回は POST で受け取る点と送信されてきたデータが`req`に入っている点に注意．

```js
// routes/janken.route.js
const express = require("express");
const router = express.Router();

const JankenController = require("../controllers/janken.controller");

router.post("/", (req, res) => JankenController.getResult(req, res));

module.exports = router;
```

`controllers/janken.controller.js`に以下の内容を記述する．

おみくじの場合とほぼ同じ．

```js
// controllers/janken.controller.js
const JankenService = require("../services/janken.service");

exports.getResult = async (req, res, next) => {
  try {
    const result = await JankenService.getJanken(req.body);
    return res.status(200).json({
      status: 200,
      result: result,
      message: "Succesfully get Janken!",
    });
  } catch (e) {
    return res.status(400).json({ status: 400, message: e.message });
  }
};
```

`services/janken.service.js`に以下の内容を記述する．

とりあえず毎回同じ結果を返している．

```js
// services/janken.service.js
exports.getJanken = async (query) => {
  try {
    return { yourHand: query.myhand, comHand: "グー", result: "テスト中" };
  } catch (e) {
    throw Error("Error while getting Janken");
  }
};
```

### 動作確認

サーバを立ち上げて，`curl`コマンドでリクエストを送信する．POST する場合は下記の書式で実行する．

結果が返ってくれば成功．

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"パー"}' localhost:3001/janken

{"status":200,"result":{"yourHand":"パー","comHand":"グー","result":"テスト中"},"message":"Succesfully get Janken!"}
```

サーバに対して`{"myhand":"パー"}`という JSON 形式のデータを POST メソッドで送信している．

### 【演習】じゃんけんの実装

上記で記述した`services/janken.service.js`にじゃんけんのロジックを実装してみよう．

サーバを立ち上げて，以下のようにじゃんけんができれば OK！

```js
$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"グー"}' localhost:3001/janken

{"status":200,"result":{"yourHand":"グー","comHand":"チョキ","result":"Win"},"message":"Succesfully get Janken!"}

$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"チョキ"}' localhost:3001/janken

{"status":200,"result":{"yourHand":"チョキ","comHand":"パー","result":"Win"},"message":"Succesfully get Janken!"}

$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"パー"}' localhost:3001/janken

{"status":200,"result":{"yourHand":"パー","comHand":"チョキ","result":"Lose"},"message":"Succesfully get Janken!"}

```

じゃんけんができたら，グーチョキパー以外の手を送信すると NG なメッセージが返す実装にもチャレンジ！

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"無敵のアレ"}' localhost:3001/janken

{"status":200,"result":{"message":"Invalid hand..."},"message":"Succesfully get Janken!"}
```

実装例：

```js
// services/janken.service.js
exports.getJanken = async (query) => {
  try {
    const hand = ["グー", "チョキ", "パー"];
    const myIndex = hand.indexOf(query.myhand);
    if (myIndex === -1) return { message: "Invalid hand..." };
    const comIndex = Math.floor(Math.random() * 3);
    const resultSheet = [
      ["Draw", "Win", "Lose"],
      ["Lose", "Draw", "Win"],
      ["Win", "Lose", "Draw"],
    ];
    return {
      yourHand: query.myhand,
      comHand: hand[comIndex],
      result: resultSheet[myIndex][comIndex],
    };
  } catch (e) {
    throw Error("Error while getting Janken");
  }
};
```

## おわりに

今回は Node.js を用いて API サーバを実装してみた．

おみくじやじゃんけんの実装を通じて，必要な処理に含まれる責務を分離し，別々のファイルに実装した．

Node.js はシンプルな構成で柔軟な API を構築できる魅力的な技術である．興味を持った方は，じゃんけんの勝率変更や DB 連携などにも挑戦してみると良いだろう．

今回は以上である( `･ω･)b
