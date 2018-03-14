# Engine Script のサンプル

## 基本形

Engine Script は一つのJavaScript関数として記述します。この関数はリクエストを引数として受け取り、戻り値として返すオブジェクトがレスポンスに使われます。

```
function(request) {
  return {
        status: 200,
        headers: {"Content-Type":"text/plain"},
        body: ["Hello World !!"]
  };
}
```

## リクエストの受取

リクエストは関数の引数で与えられます。以下では関数の引数を request と記述したときに、リクエストに関する各種情報をどのように取得するかサンプル提示しながら説明します。

### リクエストメソッド

request.method　でメソッドが取得できます。

#### GETメソッド以外はエラーとする

```
function(request) {
  if (request.method !== 'GET') {
    return {
      status: 405,
      body: ['Method Not Allowed']
    };
  }
  return {
        status: 200,
        body: ["GET method is fine"]
  };
}
```

### リクエストヘッダ

request.headers　でリクエストヘッダが取得できます。

#### 特定リクエストヘッダの値を返す

```
function(request) {
  var headerVal = request.headers['X-Some-Header'];
  if (!headerVal) {
    return {
      status: 400,
      body: ['X-Some-Header required']
    };
  }
  return {
        status: 200,
        body: ["X-Some-Header value = " + headerVal]
  };
}
```

### リクエストボディ

request.input　でリクエストボディにストリームとしてアクセス可能です。

#### リクエストボディのパース

リクエストボディが文字列であり、サイズが比較的小さいときは以下のようにすべて文字列として読み取ってしまうのが便利です。

```
    var reqString = request.input.readAll();
```

##### x-www-formurlencodedの場合

Personium Engineが提供しているユーティリティ関数を用いてパース可能です。

```
    var params = _p.util.queryParse(reqString);
```

##### JSONの場合

標準のJSONオブジェクトを使ってパースが可能です。

```
    var req = JSON.parse(reqString);
```

#### リクエストボディがバイナリである場合

リクエストボディがバイナリである場合はストリームのまま処理するのがよいでしょう。

```
    var stream = request.input;
```

取得したストリームをファイルに書き出したり、応答で使ったりといったことが可能です。


## レスポンスの返却

### レスポンスのバリエーション

#### HTMLでのレスポンス

```
function(request) {
  return {
        status: 200,
        headers: {"Content-Type":"text/html"},
        body: ["<html><body><h2>Hello World !!</h2></body></html>"]
  };
}
```

#### JSONでのレスポンス

```
function(request) {
  var res = { 
    key: "helloWorld",
    message: "Hello World !!"
  }; 
  return {
        status: 200,
        headers: {"Content-Type":"application/json"},
        body: [JSON.stringify(res)]
  };
}
```

#### ステータスコードを変えてみる

403 Forbidden エラー応答

```
function(request) {
  return {
        status: 403,
        body: []
  };
}
```

### bodyのバリエーション

bodyとして文字列の配列を返すと、それらをつなげた応答となります。

```
function(request) {
  return {
        status: 200,
        headers: {"Content-Type":"text/plain"},
        body: ["Hello", "World"]
  };
}
```

bodyとして返す配列要素はinputStreamを取ることができます。

```
function(request) {
  var is = .... (ファイル取得など)
  return {
        status: 200,
        headers: {"Content-Type":"text/plain"},
        body: [is]
  };
}
```

以下の例では入力されたリクエストボディをそのままレスポンスボディとして返します。

```
function(request) {
    var stream = request.input;
    return {
        status: 200,
        body: [stream]
    };    
}
```

bodyとして返すオブジェクトはforEachメソッドが実装されていることが要件ですので、以下のようなオブジェクトで返すことも可能です。

```
function(request) {
  var bodyObj = {
     min: 0,
     max: 2000,
     forEach: function(f) {
       var i = min;
       while (i < max) {
          f(i + ",");
          i++;
       }
     }
  };
  return {
        status: 200,
        headers: {"Content-Type":"text/plain"},
        body: bodyObj
  };
}
```

### Personium のAPI呼出処理

Engine Script 内では _pというグローバルオブジェクトを介してPersoniumの様々なAPIを呼び出し可能です。

以下の例ではこのScriptが走行するBox内の /img/picture.jpgというファイルに対して クライアントから受け取ったアクセストークンをそのまま使ってアクセスし、取得できた内容をレスポンスボディとして返しています。

```
function(request) {
  var thisBox = _p.as('client').cell().box();
  var picture = thisBox.col('img').get('picture.jpg');
  return {
        status: 200,
        headers: {"Content-Type":"image/jpeg"},
        body: [picture]
  };
}
```

