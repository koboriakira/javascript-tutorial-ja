# コンストラクタ 演算子 new

通常 `{...}` 構文で1つのオブジェクトを作ることが出来ます。しかし、しばしば多くの似たようなオブジェクトを作る必要があります、例えば複数のユーザやメニューアイテムなどです。

それは、コンストラクタ関数と `"new"` 演算子を使うことで実現できます。

[cut]

## コンストラクタ 関数 

コンストラクタ関数は技術的には通常の関数です。それには2つの慣習があります:

1. それらは先頭が大文字で名前付けされます。
2. それらは `"new"` 演算子を使ってのみ実行されるべきです。

例:

```js run
function User(name) {
  this.name = name;
  this.isAdmin = false;
}

*!*
let user = new User("Jack");
*/!*

alert(user.name); // Jack
alert(user.isAdmin); // false
```

`new User(...)` として関数が実行されたとき、次のようなステップになります:

1. 新しい空のオブジェクトが作られ、 `this` に代入されます。
2. 関数本体を実行します。通常は `this` を変更し、それに新しいプロパティを追加します。
3. `this` の値が返却されます。

つまり、`new User(...)` は次のようなことをします:

```js
function User(name) {
*!*
  // this = {};  (暗黙)
*/!*

  // this へプロパティを追加
  this.name = name;
  this.isAdmin = false;

*!*
  // return this;  (暗黙)
*/!*
}
```

なので、`new User("Jack")` の結果は次と同じオブジェクトです:

```js
let user = {
  name: "Jack",
  isAdmin: false
};
```

さて、もし他のユーザを作りたい場合、`new User("Ann")`, `new User("Alice")` と言ったように呼ぶことができます。毎回リテラルを使うよりはるかに短く、また簡単で読みやすいです。

それがコンストラクタの主な目的です -- 再利用可能なオブジェクト作成のコードを実装すること。

もう一度注意しましょう -- 技術的にはどんな関数もコンストラクタとして使うことができます。つまり: どの関数も `new` で実行することができ、それは上のアルゴリズムで実行されるでしょう。"先頭が大文字" は共通合意であり、それは関数が `new` で実行されることを明確にするためです。

````smart header="new function() { ... }"
1つの複雑なオブジェクトの作成に関する多くのコード行がある場合、コンストラクタ関数でそれをラップすることができます。このように:

```js
let user = new function() {
  this.name = "John";
  this.isAdmin = false;

  // ...ユーザ作成のための他のコードは複雑なロジック、
  // 文やローカル変数などを持つかもしれません。
};
```

コンストラクタはどこにも保存されず、単に作られて呼び出されただけなので2度は呼び出せません。なので、このトリックは将来再利用することなく、単一のオブジェクトを構成するコードをカプセル化することを目指しています。
````

## 二重構文コンストラクタ: new.target 

関数の中では、`new.target` プロパティを使うことで、それが `new` で呼ばれたかそうでないかを確認することができます。

通常の呼び出しでは空であり、 `new` で呼び出された場合は関数と等しくなります:

```js run
function User() {
  alert(new.target);
}

// new なし:
User(); // undefined

// new あり:
new User(); // function User { ... }
```

これは、 `new` と通常両方の構文が同じように動作するようにするために使用できます:

```js run
function User(name) {
  if (!new.target) { // new なしで実行した場合
    return new User(name); // ...new を追加します
  }

  this.name = name;
}

let john = User("John"); // new User へのリダイレクト
alert(john.name); // John
```

このアプローチは、構文をより柔軟にするためにライブラリの中で使われることがあります。恐らくどこへでもこれを使うのは良いことではありません。なぜなら `new` の省略は、何をしているのかを少し不明確にします。`new` があれば、新しいオブジェクトが作られることを知ることができ、それは良いことです。

## コンストラクタからの返却 

通常、コンストラクタは `return` 文を持ちません。それらのタスクは全ての必要なことを `this` の中に書くことで、それが自動的に結果になります。

しかし、もし `return` 文があった場合、そのルールはシンプルです:

- もし `return` がオブジェクトと一緒に呼ばれた場合、`this` の代わりにそれを返します。
- もし `return` がプリミティブと一緒に呼ばれた場合、それは無視されます。

言い換えると、オブエジェクトの`return` はそのオブジェクトを返し、それ以外のケースでは `this` が返却されます。

例えば、ここで `return` は オブジェクトを返却することで、`this` を上書きします:

```js run
function BigUser() {

  this.name = "John";

  return { name: "Godzilla" };  // <-- オブジェクトを返す
}

alert( new BigUser().name );  // Godzilla, オブジェクトを取得 ^^
```

また、これは空の `retruen` (もしくはプリミティブをこの後に置くことができます)の例です

```js run
function SmallUser() {

  this.name = "John";

  return; // 実行が終了し, this を返す

  // ...

}

alert( new SmallUser().name );  // John
```

通常、コンストラクタは `return` 文を持ちません。ここでは、主に完全性のためにオブジェクトを返す特殊な動作について説明します。

````smart header="丸括弧の省略"
ところで、もし引数を取らない場合は、`new` の後の丸括弧を省略することもできます。

```js
let user = new User; // <-- 括弧なし
// これと同じ
let user = new User();
```

丸括弧の省略は "良いスタイル" ではありませんが、仕様では許可されます。
````

## コンストラクタの中のメソッド 

オブジェクトを作るときにコンストラクタ関数を使用することで、大きな柔軟性を得ることができます。コンストラクタ関数はオブジェクトがどのように組み立てられるか、その中に何を置くかを定義するパラメータを持っています。

もちろん、`this` にプロパティだけでなく、メソッドも同様に追加することができます。

例えば、下の `new User(name)` は `name` と メソッド `sayHi` を持つオブジェクトを作ります:

```js run
function User(name) {
  this.name = name;

  this.sayHi = function() {
    alert( "My name is: " + this.name );
  };
}

*!*
let john = new User("John");

john.sayHi(); // My name is: John
*/!*

/*
john = {
   name: "John",
   sayHi: function() { ... }
}
*/
```

## サマリ 

- コンストラクタ関数、もしくは簡潔にコンストラクタは通常の関数ですが、大文字から始まる名前を持つ、と言う共通の合意があります。
- コンストラクタ関数は `new` を使ってのみ呼び出されるべきです。このような呼び出しは、最初に空の `this` を作成し、最後に追加された `this` を返すことを意味します。

複数の似たようなオブジェクトを作るときにコンストラクタ関数を使うことができます。

JavaScript は多くの組み込み言語オブジェクトのコンストラクタを提供しています: 日付のための `Date`, セットのための `Set`、そしてその他私たちが学ぶ予定のものがあります。

```smart header="オブジェクト, 我々は戻ってきます!"
このチャプターでは、オブジェクトとコンストラクタについての基礎のみをカバーしています。これらは、次のチャプターでデータ型と関数についてより深く学ぶために不可欠です。

それを学んだ後、チャプター　<info:object-oriented-programming> では、オブジェクトに戻り、継承やクラスを含めそれらを詳細に説明します。
```
