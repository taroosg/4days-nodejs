# 4 days Node.js / day04

## 今回のゴール

- これまで実装した Node.js の API をクライアントから呼び出す．
- サーバとクライアント型の API アプリケーションの動き方の感覚を掴む．
- 生 JS で実装して JS 力の向上を図る．

---

## 準備と方針

### やること

これまでの内容で，大きく 3 つの API を実装した．「おみくじとじゃんけん」「todo リスト」「スクレイピング」である．

今回は，これらの API をクライアント側から呼び出すコードを実装し，Web アプリケーションとして動作させてみる．

リクエストには`axios`ライブラリを使うと記述がしやすいためこれを利用する．`fetch()`関数でも実装可能なので，一通り動かした後でトライしてみるとよいだろう．

### サーバ側の準備

クライアントのアプリケーションからリクエストを実行する場合，ドメインが異なるとサーバ側でアクセスが弾かれる．これを CORS(Cross-Origin Resource Sharing)と呼ぶ．

クライアントからアクセスするには，サーバ側でこれを許可する旨の記述を行っておく必要がある．

前回までに実装したサーバ側アプリケーションを開き，以下のコマンドを実行する．

```bash
$ npm i cors
```

続いて，`app.js`を以下のように編集する．

```js
const express = require("express");
// 🔽 追記 🔽
const cors = require("cors");
const app = express();

const bodyParser = require("body-parser");
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
// 🔽 追加 🔽
app.use(cors());

const port = 3001;
const omikujiRouter = require("./routes/omikuji.route");
const jankenRouter = require("./routes/janken.route");
const scrapingRouter = require("./routes/scraping.route");
const todoRouter = require("./routes/todo.route");

app.get("/", (req, res) => {
  res.json({
    uri: "/",
    message: "Hello Node.js!",
  });
});

app.use("/omikuji", (req, res) => omikujiRouter(req, res));
app.use("/janken", (req, res) => jankenRouter(req, res));
app.use("/scraping", (req, res) => scrapingRouter(req, res));
app.use("/todo", (req, res) => todoRouter(req, res));

app.use((req, res, next) => {
  res.status(404).json({ status: 404, message: "not found..." });
});

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`);
});
```

上記のように記述した場合は，すべてのルーティングに対して CORS を許可したことになる（個別に設定することもできる）．

サーバ側とクライアント側が同じドメインの場合は，これらの設定は不要である（設定しなくてもアクセス可能）．

記述したら，下記コマンドでサーバを立ち上げておく．

```bash
$ npm start
```

### クライアント側のファイルを作成

適当な場所に適当な名前でフォルダを作成し，以下の構造になるようにファイルを作成する．

```bash
.
├── index.html
├── scraping.html
└── todo.html
```

---

## おみくじとじゃんけんの処理を実装

### HTML ファイルの準備

`index.html`に以下の内容を追記する．

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>

  <body>
    <h1>おみくじ&じゃんけん</h1>
    <section>
      <h2>おみくじ</h2>
      <button id="omikuji">おみくじを引く</button>
      <p id="omikuji_result">結果</p>
    </section>
    <section>
      <h2>じゃんけん</h2>
      <button class="janken" value="0">グー</button>
      <button class="janken" value="1">チョキ</button>
      <button class="janken" value="2">パー</button>
      <p id="server_hand">サーバの手</p>
      <p id="janken_result">結果</p>
    </section>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    <script>
      // おみくじの処理
      // じゃんけんの処理
    </script>
  </body>
</html>
```

### おみくじ

コマンドで実行する場合のおみくじのリクエストは下記の通り（動作確認しておこう）．

```bash
$ curl localhost:3001/omikuji
{"status":200,"result":{"result":"凶"},"message":"Succesfully get Omikuji!"}
```

このリクエストをクライアントから`axios`ライブラリを用いて送信する．流れは以下の通り．

1. おみくじボタンをクリックしたらサーバにリクエストを行う．
2. レスポンスが返ってきたら画面に結果を表示する．

`index.html`の`<script></script>`内に以下の内容を追記する．

GET メソッドのリクエストを送信する場合は，`axios.get()`に URL を渡せば良い．

```js
// 🔽 おみくじの処理 🔽
document.getElementById("omikuji").addEventListener("click", async (e) => {
  const requestUrl = "http://localhost:3001/omikuji";
  const omikuji_result = await axios.get(requestUrl);
  document.getElementById("omikuji_result").innerText =
    omikuji_result.data.result.result;
});
```

記述したら動作確認を行う．ブラウザ画面からおみくじボタンをクリックし，結果が表示されれば OK！

うまく動かない場合は適宜`console.log()`で動作を確認しよう．

### じゃんけん

コマンドで実行する場合のじゃんけんのリクエストは下記の通り（動作確認しておこう）．

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"myhand":"パー"}' localhost:3001/janken

{"status":200,"result":{"yourHand":"パー","comHand":"グー","result":"Draw"},"message":"Succesfully get Janken!"}
```

このリクエストをクライアントから`axios`ライブラリを用いて送信する．流れは以下の通り．

1. 自分の手のボタンクリックする．
2. ボタンに割り当てられた value の値を取得する．
3. 取得した値から自分の手の文字列を決定し，POST メソッドでリクエストを送信する．
4. レスポンスを受け取ったら，中身を取り出して画面に表示する．

`index.html`の`<script></script>`内に以下の内容を追記する．

POST リクエストを送信する場合には`axios.post()`を用いる．

第 1 引数にリクエスト先の URL,第 2 引数に送信するデータを渡す．

> 解説
>
> じゃんけんのボタンは複数存在するため，以下の手順でクリックイベントを作成する必要がある．
>
> 1.  `getElementsByClassName`で要素をすべて取得する．
> 2.  スプレッド構文を用いて取得した要素を配列に変換する．
> 3.  `forEach()`を用いて全てにクリックイベントを作成する．

```js
// おみくじの処理
// 省略

// 🔽 じゃんけんの処理 🔽
const jankenHand = ["グー", "チョキ", "パー"];
[...document.getElementsByClassName("janken")].forEach((x) => {
  x.addEventListener("click", async (e) => {
    const requestUrl = "http://localhost:3001/janken";
    const requestData = { myhand: jankenHand[Number(e.target.value)] };
    const jankenResult = await axios.post(requestUrl, requestData);
    document.getElementById("server_hand").innerText =
      jankenResult.data.result.comHand;
    document.getElementById("janken_result").innerText =
      jankenResult.data.result.result;
  });
});
```

記述したら動作確認する．「グー」「チョキ」「パー」それぞれのボタンをクリックし，画面に結果が表示されれば OK．

うまく動かない場合は適宜`console.log()`で動作を確認しよう．

---

## todo リストの実装

サーバ側で todo リストの CRUD 処理を実装した．今回はクライアントのアプリケーションからリクエストを行い，各種処理を動作させる．

### 準備

`todo.html`に以下の内容を記述する．今回は大きく分けて 10 個の処理を作成する必要がある．

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>

  <body>
    <h1>todoリスト</h1>
    <form name="form">
      <input type="text" name="todo" />
      <input type="date" name="deadline" />
      <button type="button" id="submit">送信</button>
    </form>
    <ul id="result"></ul>

    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    <script>
      // データ取得のGETリクエストを送信する関数

      // todoをHTML要素に変換する関数

      // 最新データを取得して画面に表示する関数

      // データ作成をPOSTリクエストを送信する処理

      // データ更新のPUTリクエストを送信する関数

      // 更新ボタンクリックイベントを追加する関数

      // データ削除のDELETEリクエストを送信する関数

      // 削除ボタンクリックイベントを追加する関数

      // 読み込み時にデータ取得実行

      // 送信ボタンクリック時にデータ追加 -> データ取得実行
    </script>
  </body>
</html>
```

### データ取得

まず，すでに保存されている todo の一覧を取得して画面に表示する処理を実装する．

ターミナルで実行する場合のコマンドは以下の通り（動作確認しておく）．

```bash
$ curl localhost:3001/todo

{
  "status": 200,
  "result": [
    {
      "id": "hUqyK0rwVdzGQMWGrVCh",
      "data": {
        "deadline": "2021-02-17T00:00:00.000Z",
        "done": false,
        "created_at": "2021-02-12T08:27:52.390Z",
        "updated_at": "2021-02-12T08:27:52.390Z",
        "todo": "node.js"
      }
    },
    {
      "id": "jmzfkE9XLzCMJ0k6gLtD",
      "data": {
        "done": false,
        "created_at": "2021-02-12T08:27:46.442Z",
        "deadline": "2021-02-24T00:00:00.000Z",
        "updated_at": "2021-02-12T08:27:46.442Z",
        "todo": "next.js"
      }
    }
  ],
  "message": "Succesfully get Todo Data!"
}
```

`todo.html`の`<script></script>`内に以下の内容を記述する．

処理の流れは以下のとおり．

1. サーバからデータを取得する（`getTodo()`）．
2. 取得したデータを HTML 要素に変換する（`createElements()`）．
3. HTML 要素を画面に表示する（`(async () => { ... })()`）．

作成する HTML 要素は更新処理を実装しやすくするため`input`を用いている．フォームの`name`や更新ボタン削除ボタンにはデータを識別するためにドキュメント ID を入れている．

> 解説
>
> - `axios`関連の処理は非同期で実行されるため，`async / await`を用いて実装している．
> - 読み込み時に上記 1 と 2 の処理を実行したいため，3 のような記述が必要になる．

```js
// 🔽 データ取得のGETリクエストを送信する関数 🔽
const getTodo = async () => {
  const requestUrl = `http://localhost:3001/todo`;
  const getResult = await axios.get(requestUrl);
  return getResult.data.result;
};

// 🔽 todoをHTML要素に変換する関数 🔽
const createElements = async (todoData) => {
  return todoData
    .map(
      (x) => `
        <li>
          <form name="${x.id}">
          <p>📝: <input type="text" name="todo" value="${x.data.todo}"></p>
          <p>📆: <input type="date" name="deadline" value="${x.data.deadline.slice(
            0,
            10
          )}"></p>
          <p>✅: <input type="checkbox" name="done" ${
            x.data.done ? "checked" : ""
          }></p>
          <button type="button" class="update_button" value="${
            x.id
          }">更新</button>
          <button type="button" class="delete_button" value="${
            x.id
          }">削除</button>
          </form>
        </li>
      `
    )
    .join("");
};

// 最新データを取得して画面に表示する関数
const showNewDataElements = async () => {
  const todoData = await getTodo();
  const todoElements = await createElements(todoData);
  document.getElementById("result").innerHTML = todoElements;
};

// データ作成をPOSTリクエストを送信する処理

// データ更新のPUTリクエストを送信する関数

// 更新ボタンクリックイベントを追加する関数

// データ削除のDELETEリクエストを送信する関数

// 削除ボタンクリックイベントを追加する関数

// 🔽 読み込み時にデータ取得実行 🔽
(async () => {
  showNewDataElements();
})();

// 送信ボタンクリック時にデータ追加 -> データ取得実行
```

記述したら動作確認する．画面に情報が表示されれば OK！

各種データの構造などは適宜`console.log()`で出力して確認しよう．

### データ追加 -> 最新情報の取得

続いて，データ作成の処理を実装する．コマンドでリクエストを送る場合は以下．

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"todo":"node.js","deadline":"2021-02-17"}' localhost:3001/todo

{
  "status": 200,
  "result": {
    "id": "mA665PVzkTzqxxBx0Msv",
    "data": {
      "todo": "node.js",
      "deadline": {
        "_seconds": 1613520000,
        "_nanoseconds": 0
      },
      "done": false,
      "created_at": {
        "_seconds": 1613117267,
        "_nanoseconds": 915000000
      },
      "updated_at": {
        "_seconds": 1613117267,
        "_nanoseconds": 915000000
      }
    }
  },
  "message": "Succesfully post Firestore Data!"
}
```

`todo.html`の`<script></script>`内を以下のように編集する．

処理の流れは以下のとおり．

1. サーバに POST リクエストを送信する処理を作成する（`postTodo()`）．
2. 送信ボタンのクリックイベントを作成する．
3. 送信ボタンクリック時に，入力フォームの値を取得する．
4. 取得した入力フォームの値をサーバに送信する．
5. データ送信後，最新データを取得して画面に表示する．

> 解説
>
> - フォームの値は`document.フォームの名前.inputの名前.value`で取得できる．

```js
// データ取得のGETリクエストを送信する関数
const getTodo = async () => {
  // 省略
};

// todoをHTML要素に変換する関数
const createElements = async (todoData) => {
  // 省略
};

// 最新データを取得して画面に表示する関数
const showNewDataElements = async () => {
  // 省略
};

// 🔽 データ作成をPOSTリクエストを送信する処理 🔽
const postTodo = async (postData) => {
  const requestUrl = `http://localhost:3001/todo`;
  return await axios.post(requestUrl, postData);
};

// データ更新のPUTリクエストを送信する関数

// 更新ボタンクリックイベントを追加する関数

// データ削除のDELETEリクエストを送信する関数

// 削除ボタンクリックイベントを追加する関数

// 読み込み時にデータ取得実行
(async () => {
  showNewDataElements();
})();

// 🔽 送信ボタンクリック時にデータ追加 -> データ取得実行 🔽
document.getElementById("submit").addEventListener("click", async (e) => {
  const postData = {
    todo: document.form.todo.value,
    deadline: document.form.deadline.value,
  };
  const postReault = await postTodo(postData);
  showNewDataElements();
  alert("created!");
});
```

記述したら動作確認する．フォームに入力して送信ボタンをクリックすると，入力したデータが作成されて画面に表示される．

### データ更新 -> 最新情報の取得

データの更新処理を実装する．更新処理はドキュメント ID と更新データを PUT メソッドで送信する必要がある．

コマンドで実行する場合は以下のとおり．

```bash
$ curl -X PUT -H "Content-Type: application/json" -d '{"todo":"nest.js","deadline":"2021-02-28","done":true}' localhost:3001/todo/yQjmX9lHuRM5WxIUCAjg

{
  "status": 200,
  "result": {
    "data": {
      "todo": "nest.js",
      "deadline": {
        "_seconds": 1614470400,
        "_nanoseconds": 0
      },
      "done": true,
      "updated_at": {
        "_seconds": 1613122454,
        "_nanoseconds": 165000000
      }
    }
  },
  "message": "Succesfully update Todo Data!"
}
```

`todo.html`を以下のように編集する．処理の流れは以下のとおり．

1. データ更新の PUT リクエストを送信する関数を実装する（`updateTodo()`）．
2. HTML 要素作成時に，更新ボタンクリックイベントを追加する（`createUpdateEvent()`）．
3. （上記クリックイベントで）該当する入力フォームの値を取得する（`createUpdateEvent()`）．
4. 取得したデータをドキュメント ID とともにサーバに送信する（`createUpdateEvent()`）．
5. 更新処理が完了したら最新データの一覧を取得して画面を更新する（`createUpdateEvent()`）．

> 解説
>
> - PUT メソッドの場合，サーバ側では`req.params`で ID を，`req.body`で更新データを受け取る．
> - ID は`uri/id`の形で埋め込み，更新データは POST メソッドと同様に第 2 引数で送信できる．
> - 更新ボタンのクリックイベントは，HTML 要素が作成された後に作成する必要がある．そのため関数（`createUpdateEvent()`）にまとめ，一覧表示のタイミングで実行するように記述する．

```js
// データ取得のGETリクエストを送信する関数
const getTodo = async () => {
  // 省略
};

// todoをHTML要素に変換する関数
const createElements = async (todoData) => {
  // 省略
};

// 最新データを取得して画面に表示する関数
const showNewDataElements = async () => {
  const todoData = await getTodo();
  const todoElements = await createElements(todoData);
  document.getElementById("result").innerHTML = todoElements;
  // 🔽 追加 🔽
  createUpdateEvent();
};

// データ作成をPOSTリクエストを送信する処理
const postTodo = async (postData) => {
  // 省略
};

// 🔽 データ更新のPUTリクエストを送信する関数 🔽
const updateTodo = async (documentId, postData) => {
  const requestUrl = `http://localhost:3001/todo/${documentId}`;
  return await axios.put(requestUrl, postData);
};

// 🔽 更新ボタンクリックイベントを追加する関数 🔽
const createUpdateEvent = () => {
  [...document.getElementsByClassName("update_button")].forEach((x) => {
    x.addEventListener("click", async (e) => {
      const documentId = e.target.value;
      const updateData = {
        todo: document[e.target.value].todo.value,
        deadline: document[e.target.value].deadline.value,
        done: document[e.target.value].done.checked,
      };
      const updateResult = await updateTodo(documentId, updateData);
      showNewDataElements();
      alert("updated!");
    });
  });
};

// データ削除のDELETEリクエストを送信する関数

// 削除ボタンクリックイベントを追加する関数

// 読み込み時にデータ取得実行
(async () => {
  showNewDataElements();
})();

// 送信ボタンクリック時にデータ追加 -> データ取得実行
document.getElementById("submit").addEventListener("click", async (e) => {
  // 省略
});
```

記述したら動作確認する．適当に入力値を変更し，更新ボタンをクリックしてアラートが出れば OK！

リロードしても上で更新した値になっている．

### データ削除 -> 最新情報の取得

削除ボタンクリック時に該当するデータを削除する処理を実装する．コマンドで実行する場合は以下のとおり．

```bash
$ curl -X DELETE localhost:3001/todo/yQjmX9lHuRM5WxIUCAjg

{
  "status": 200,
  "result": {
    "id": "yQjmX9lHuRM5WxIUCAjg"
  },
  "message": "Succesfully delete Todo Data!"
}
```

`todo.html`に以下の内容を記述する．削除の処理はドキュメントの ID さえわかれば実行できるため実装は容易である．処理の流れは以下のとおり．

1. DELETE メソッドでリクエストを送信する関数を作成する（`deleteTodo()`）．
2. 削除ボタンのクリックイベントを作成する（`createDeleteEvent()`）．
3. クリック時にドキュメント ID を取得する（`createDeleteEvent()`）．
4. サーバに DELETE メソッドのリクエストを送信する（`createDeleteEvent()`）．
5. データ削除完了後に最新データを取得する（`createDeleteEvent()`）．

更新処理と同様に，HTML 要素が作成された後に作成する必要がある．そのため`showNewDataElements()`内で`createDeleteEvent()`を実行している．

```js
// データ取得のGETリクエストを送信する関数
const getTodo = async () => {
  // 省略
};

// todoをHTML要素に変換する関数
const createElements = async (todoData) => {
  // 省略
};

// 最新データを取得して画面に表示する関数
const showNewDataElements = async () => {
  const todoData = await getTodo();
  const todoElements = await createElements(todoData);
  document.getElementById("result").innerHTML = todoElements;
  createUpdateEvent();
  // 🔽 追加 🔽
  createDeleteEvent();
};

// データ作成をPOSTリクエストを送信する処理
const postTodo = async (postData) => {
  // 省略
};

// データ更新のPUTリクエストを送信する関数
const updateTodo = async (documentId, postData) => {
  // 省略
};

// 更新ボタンクリックイベントを追加する関数
const createUpdateEvent = () => {
  // 省略
};

// 🔽 データ削除のDELETEリクエストを送信する関数 🔽
const deleteTodo = async (documentId) => {
  const requestUrl = `http://localhost:3001/todo/${documentId}`;
  return await axios.delete(requestUrl);
};

// 🔽 削除ボタンクリックイベントを追加する関数 🔽
const createDeleteEvent = () => {
  [...document.getElementsByClassName("delete_button")].forEach((x) => {
    x.addEventListener("click", async (e) => {
      const documentId = e.target.value;
      const deleteResult = await deleteTodo(documentId);
      showNewDataElements();
      alert("deleted!");
    });
  });
};

// 読み込み時にデータ取得実行
(async () => {
  showNewDataElements();
})();

// 送信ボタンクリック時にデータ追加 -> データ取得実行
document.getElementById("submit").addEventListener("click", async (e) => {
  // 省略
});
```

記述したら動作確認する．削除ボタンをクリックし，該当するデータが削除されれば OK！

これで todo リストの CRUD 処理は完了である．

---

## スクレイピングの実装

todo リストの実装を通して記述した内容を元に，スクレイピング部分も実装してみよう．

スクレイピングの API で実装するのは以下 2 つの処理．

- 現在保存されているスクレイピングしたデータを画面に表示する．
- 最新データ取得ボタンクリック時に，新たなデータを取得して保存し，画面を最新の情報に更新する．

### 準備

まずは，HTML の構造を準備する．`scraping.html`を以下のように編集する．

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>

  <body>
    <h1>スクレイピング</h1>
    <section>
      <h2>新規スクレイピング</h2>
      <button type="button" id="start">スクレイピングする</button>
    </section>
    <section>
      <h2>保存されているスクレイピングのデータ</h2>
      <div id="result"></div>
    </section>

    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    <script>
      // 保存されているスクレイピングのデータを取得する関数

      // スクレイピングのデータをHTML要素に変換する関数

      // 最新データを取得して画面に表示する関数

      // 新しいスクレイピングをリクエストする関数

      // 読み込み時にデータを取得して表示する処理

      // クリック時にスクレイピング -> 最新データを取得する処理
    </script>
  </body>
</html>
```

### データ表示

現在保存されているスクレイピングで取得したデータを表示してみよう．動き方は todo リストのデータ取得と同様である．

ターミナルで実行する場合は以下のコマンドになる．

```bash
$ curl localhost:3001/scraping/read

{
  "status": 200,
  "result": [
    {
      "id": "NUHcdv9quiDme44b2j1q",
      "data": {
        "created_at": "2021-02-12T16:07:09.649Z",
        "data": [
          # データたくさん
        ]
      }
    }
  ],
  "message": "Succesfully post Scraping Data!"
}
```

`scraping.html`を以下のように編集する．取得したデータの中に配列で更にデータが入っている．そのため，`map()`内で更に`map()`する必要がある．

このような場合は構造が把握しづらいため，まず console でデータの形を確認したほうが良い．

```js
// 🔽 保存されているスクレイピングのデータを取得する関数 🔽
const getScrapingData = async () => {
  const requestUrl = `http://localhost:3001/scraping/read`;
  const getResult = await axios.get(requestUrl);
  return getResult.data.result;
};

// 🔽 スクレイピングのデータをHTML要素に変換する関数 🔽
const createElements = async (scrapingData) => {
  return scrapingData
    .map(
      (x) => `
        <details>
          <summary>${x.data.created_at}</summary>
          <ul>
            ${x.data.data
              .map(
                (y) => `
              <li>
                <p><a href="${y.url}" target="_blank">🔗 ${y.title}</a></p>
              </li>
             `
              )
              .join("")}
          </ul>
        </details>
      `
    )
    .join("");
};

// 🔽 最新データを取得して画面に表示する関数 🔽
const showNewDataElements = async () => {
  const scrapingData = await getScrapingData();
  const scrapingElements = await createElements(scrapingData);
  document.getElementById("result").innerHTML = scrapingElements;
};

// 新しいスクレイピングをリクエストする関数

// 🔽 読み込み時にデータを取得して表示する処理 🔽
(async () => {
  showNewDataElements();
})();

// クリック時にスクレイピング -> 最新データを取得する処理
```

記述したら動作確認する．保存されているデータが画面に表示されれば OK！

### データ追加 -> 最新データの取得

ボタンクリック時にスクレイピングを実行し，データ保存 -> 最新データの取得を実装する．

こちらも todo リストの create の処理と同様の動きである．ターミナルでのリクエストは以下．

```bash
$ curl localhost:3001/scraping/create

{
  "status": 200,
  "result": {
    "id": "NUHcdv9quiDme44b2j1q",
    "data": {
      "data": [
        # データたくさん
      ],
      "created_at": {
        "_seconds": 1613146029,
        "_nanoseconds": 649000000
      }
    }
  },
  "message": "Succesfully post Scraping Data!"
}
```

`scraping.html`を以下のように編集する．

```js
// 保存されているスクレイピングのデータを取得する関数
const getScrapingData = async () => {
  // 省略
};

// スクレイピングのデータをHTML要素に変換する関数
const createElements = async (scrapingData) => {
  // 省略
};

// 最新データを取得して画面に表示する関数
const showNewDataElements = async () => {
  // 省略
};

// 🔽 新しいスクレイピングをリクエストする関数 🔽
const newScraping = async () => {
  const requestUrl = `http://localhost:3001/scraping/create`;
  const scrapingResult = await axios.get(requestUrl);
  return scrapingResult.data.result;
};

// 読み込み時にデータを取得して表示する処理
(async () => {
  showNewDataElements();
})();

// 🔽 クリック時にスクレイピング -> 最新データを取得する処理 🔽
document.getElementById("start").addEventListener("click", async () => {
  const scrapingResult = await newScraping();
  showNewDataElements();
  alert("done!");
});
```

実行してスクレイピングが動作し，画面のコンテンツが増えていれば OK！

---

## まとめ

今回は，前回までに実装した API に対してクライアント側の処理を実装した．

API 側で URI を設定してデータの形状も整えていたが，実際に画面に表示するためには HTML 要素に変換する必要がある．

Web アプリケーションとしては避けては通れない部分であり，うまく JavaScript のライブラリやフレームワークを利用することも視野に入るだろう．

また，これらの処理を実装しないために，既存のプラットフォーム（LINE や Twitter，Discord など）の使用も有効だ．

サーバとクライアントの実装を通して，Web アプリケーションの動きや実装のイメージを持っていただければ幸いだ．

今回は以上である( `･ω･)b
