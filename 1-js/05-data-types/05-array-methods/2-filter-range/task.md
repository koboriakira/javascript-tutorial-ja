importance: 4

---

# 範囲でフィルタする

配列 `arr` を取得し、`a` と `b` の間で要素を探し、それらの配列を返す関数 `filterRange(arr, a, b)` を書いてください。

この関数は配列を変更するべきではありません。新しい配列を返すべきです。

例:

```js
let arr = [5, 3, 8, 1];

let filtered = filterRange(arr, 1, 4);

alert( filtered ); // 3,1 (マッチした値)

alert( arr ); // 5,3,8,1 (修正されていない)
```
