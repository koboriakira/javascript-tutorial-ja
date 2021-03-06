# 座標

要素を移動させるには、座標についてよく知っている必要があります。

ほとんどの JavaScript メソッドは２つの座標系のいずれかを扱います:

1. ウィンドウ(もしくは別のビューポート)の 上/左 を基準にします
2. ドキュメントの 上/左 を基準とします

違いを理解し、どのタイプがどこにあるかを理解することは重要です。

[cut]

## ウィンドウ座標: getBoundingClientRect 

ウィンドウ座標はウィンドウの左上端から始まります。

メソッド `elem.getBoundingClientRect()` はプロパティを持つオブジェクトとして `elem` に対するウィンドウ座標を返します。:

- `top` -- 要素の上端の Y-座標,
- `left` -- 要素の左端の X-座標,
- `right` -- 要素の右端の X-座標,
- `bottom` -- 要素の下端の Y-座標.

このようになります:

![](coords.png)


ウィンドウ座標はドキュメントのスクロールアウト部分を考慮せず、ウィンドウの左上端から計算されたものになります。

言い換えると、ページをスクロールするとき、要素は上下に移動し、*そのウィンドウ座標は変わります*。これはとても重要です。


```online
ボタンをクリックしてウィンドウ座標を見てみてください:

<input id="brTest" type="button" value="このボタンの  button.getBoundingClientRect() を表示する" onclick='showRect(this)'/>

<script>
function showRect(elem) {
  let r = elem.getBoundingClientRect();
  alert("{top:"+r.top+", left:"+r.left+", right:"+r.right+", bottom:"+ r.bottom + "}");
}
</script>

もしページをスクロールすると、ボタン位置は代わり、ウィンドウ座標も同様に変わります。
```

また:

- 座標は少数の場合があります。それは正常で、内部的にはブラウザは計算のためにそれを使用します。私たちは `style.position.left/top` へ設定するときにそれらを丸める必要はありません。ブラウザで少数は問題ありません。
- 座標は負の値になる場合があります。例えば、ページが下にスクロールされ、`elem` の上端がウィンドウの上にある場合、`elem.getBoundingClientRect().top` は負の値になります。
- Chromeような一部のブラウザでは、結果 `getBoundingClientRect` にプロパティ `width` と `height` も追加します。減算することでもそれらを取得することは可能です: `height=bottom-top`, `width=right-left`。

```warn header="座標 右/下 は CSS プロパティとは異なります"
ウィンドウ座標と CSSの配置を比較すると、`position:fixed` との明らかな類似点があります -- ビューポートを基準とした位置も同じです。

しかし、CSS では `right` プロパティは右端からの距離を意味し、`bottom` は -- 下端からの距離です。

上の図を見た時、JavaScriptではそうでないことが分かります。すべてのウィンドウ座標は左上隅から数えられます。
```

## elementFromPoint(x, y) 

`document.elementFromPoint(x, y)` の呼び出しは、ウィンドウ座標 `(x, y)` で最もネストされた要素を返します。

構文は次の通りです:

```js
let elem = document.elementFromPoint(x, y);
```

例えば、下のコードは今ウィンドウの中央にある要素のタグを強調表示し、出力します。:

```js run
let centerX = document.documentElement.clientWidth / 2;
let centerY = document.documentElement.clientHeight / 2;

let elem = document.elementFromPoint(centerX, centerY);

elem.style.background = "red";
alert(elem.tagName);
```

ウィンドウ座標を使うので、要素は現在のスクロール位置に応じて異なります。

````warn header="ウィンドウ外の座標の場合、`elementFromPoint` は `null` を返します。"
メソッド `document.elementFromPoint(x,y)` は `(x,y)` が可視領域にある場合にのみ動作します。

もしある座標が負の値またはウィンドウの幅/高さを超えている場合、`null` を返します。

ほとんどの場合、このような振る舞いは問題ではありませんが、それを心に留めておく必要があります。

これは、それをチェックしない場合に発生する可能性のある典型的なエラーです:

```js
let elem = document.elementFromPoint(x, y);
// 座標がウィンドウ外の場合、elem = null
*!*
elem.style.background = ''; // エラー!
*/!*
```
````

## position:fixed を用いる 

多くの場合、何かを配置するために座標を必要とします。CSS ではビューポートを基準として要素を配置するために、`left/top` (または `right/bottom`) と一緒に `position:fixed` を使います。

私たちは、`getBoundingClientRect` を使って要素の座標を取得し、その近くに何かを表示することができます。

例えば、下の関数 `createMessageUnder(elem, html)` は `elem` の下にメッセージを表示します。:

```js
let elem = document.getElementById("coords-show-mark");

function createMessageUnder(elem, html) {
  // メッセージ要素の作成
  let message = document.createElement('div');
  // ここでは、スタイルにCSSクラスを使う方が良いです
  message.style.cssText = "position:fixed; color: red";

*!*
  // 座標の設定, "px" を忘れないこと!
  let coords = elem.getBoundingClientRect();

  message.style.left = coords.left + "px";
  message.style.top = coords.bottom + "px";
*/!*

  message.innerHTML = html;

  return message;
}

// 使用例:
// ドキュメント上に５秒間追加します
let message = createMessageUnder(elem, 'Hello, world!');
document.body.append(message);
setTimeout(() => message.remove(), 5000);
```

```online
実行するにはボタンをクリックしてください:

<button id="coords-show-mark">id="coords-show-mark" のボタン, この下にメッセージが現れます</button>
```

このコードを変更して、メッセージを左、右、下に表示したり、"フェードイン" するための CSS アニメーションを適用することができます。私たちは要素のすべての座標とサイズを知っているので簡単にできます。

しかし、重要な点に注意してください: ページがスクロールされたとき、メッセージはボタンから離れていきます。

理由は明らかです: メッセージ要素は `position:fixed` に依存しているので、ページがスクロールしている間、ウィンドウの同じ場所にい続けます。

変更するためには、ドキュメントベースの座標を使い、`position:absolute` をを使う必要があります。

## ドキュメント座標 

ドキュメント相対座標は、ウィンドウではなくドキュメントの左上端から始めます。

CSS では、ウィンドウ座標は `position:fixed` に対応する一方、ドキュメント座標は `position:absolute` に似ています。

`position:absolute` と `top/left` を使うことで、ドキュメント上の特定の場所に何かを置くことができます。なので、ページのスクロール時にそこに残ることができます。しかし、最初に正しい座標が必要です。

分かりやすくするために、ウィンドウ座標 `(clientX,clientY)` とドキュメント座標 `(pageX,pageY)` を呼び出します。

ページがスクロールされていないとき、ウィンドウ座標とドキュメント座標はまったく同じです。それらのゼロの点も一致します:

![](document-window-coordinates-zero.png)

そして、スクロールをすると、`(clientX,clientY)` が変わります。なぜなら、それらはウィンドウに相対的なためです。しかし、`(pageX,pageY)` は同じままです。

これは縦スクロール後の同じページです:

![](document-window-coordinates-scroll.png)

- 要素は今ウィンドウの上部にあるので、ヘッダー `"From today's featured article"` の `clientY` は `0` になりました。
- 水平スクロールをしなかったため、`clientX` は変わりませんでした。
- 要素の `pageX` と `pageY` 座標は依然として同じです。なぜなら、それらはドキュメントに相対的だからです。

## ドキュメント座標の取得 

要素のドキュメント座標を取得するための標準メソッドはありません。しかし、簡単に書けます。

２つの座標系は次の式で繋がれます:
- `pageY` = `clientY` + ドキュメントのスクロールアウトした垂直部分の高さ
- `pageX` = `clientX` + ドキュメントのスクロールアウトした水平部分の幅

関数 `getCoords(elem)` は `elem.getBoundingClientRect()` からウィンドウ座標を取り、それらに現在のスクロールを加えます。:

```js
// 要素のドキュメント座標を取得
function getCoords(elem) {
  let box = elem.getBoundingClientRect();

  return {
    top: box.top + pageYOffset,
    left: box.left + pageXOffset
  };
}
```

## サマリ 

ページ上に任意の点は座標を持っています:

1. ウィンドウに相対的 -- `elem.getBoundingClientRect()`
2. ドキュメントに相対的 -- `elem.getBoundingClientRect()` + 現在のページスクロール

ウィンドウ座標は `position:fixed` と一緒に使用するのが賢明で、ドキュメント座標は `position:absolute` と上手くやります。

どちらの座標系も "長所" と "短所" を持っており、CSS の `position` `absolute` と `fixed` のように、どちらか一方が必要なときがあります。
