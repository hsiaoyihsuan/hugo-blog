---
title: "JavaScript 矩陣, 字串, 數字 實用語法集"
date: 2023-03-22
categories:
  - JavaScript
tags:
  - JavaScript
  - Front-end
thumbnailImagePosition: left
thumbnailImage: images/javascript.svg
---

大家在新學習一門程式語言時，想必都會經歷一段不斷刷題練習的時期，在練題及翻閱官方文件的過程中除了更加熟悉該語言，同時也累積了不少好用的語法及函式，藉由這個筆記，希望可以幫助大家統整 JS 中實用的語法與操作，筆記主要著重在矩陣、數字、字串間的使用方法與關係，那些過於複雜或冷門的就不收錄了。

## 1. String 使用方法

**字串轉矩陣 Convert string to array:**

在確認是否轉換成功時要使用`isArray`，不能使用`typeof`(`typeof `回傳結果是`number`)

```javascript

// ES6 spread syntax ...
[...str]
ex: let arr = [..."123"] // ["1","2","3"]

// split()
str.split()
ex: "123".split("") // ["1","2","3"]
ex: "123".split() // ["123"]

// Array.from()
Array.from(str)
ex: Array.from("123") // ["1","2","3"]

```

**字串中取出字串 Extract string from string:**
```javascript

//slice， startIndex ~ endIndex，endIndex 不包括在內，回傳新 string，原 string 不變
//str.slice(startIndex, endIndex);
ex: "01234".slice(1, 3); // "12"
ex: "01234".slice(3, 1000); // "34"
ex: "01234".slice(3); // "34"
ex: "01234".slice(-100, 3); // "12"

// 類似矩陣 index 的取用法
str[index];
ex: "012"[1]; // "1"
// 超出範圍不會壞，在迴圈中的控制中就不太需要擔心超出範圍的問題
ex: "012"[-100]; // undefined
ex: "012"[100]; // undefined
```

---

**合併字串 Merge strings:**

```javascript

// backtick ``，較推薦的使用法
`${str1}${str2}`ex: let str1="Hi", str2="my love!"
   `${str1}, ${str2}` // "Hi, my love!"

// + operator
str1 + str2
ex: "Hi" + ", " + "my love!" // "Hi, my love!"

// Array.concat
// 使用法寫在 Convert Array to String 區域

```

---

**字串替換 String replacement:**

```javascript

// replace，替換從頭遇到的第一個遇到的字串，回傳新字串，原字串不變
str.replace(pattern, replacement)
ex:
let str = "cat dog cat duck"
str.replace("cat", "KITTY") // "KITTY dog cat duck"
str.replace(/cat/, "KITTY") // "KITTY dog cat duck"

// replaceAll，替換全部符合的字串，回傳新字串，原字串不變
str.replaceAll(pattern, replacement)
str.replaceAll("cat", "KITTY") // "KITTY dog KITTY duck"
str.replaceAll(/cat/g, "KITTY") // "KITTY dog KITTY duck"

// trim，修剪字串前後的空白
str.trim()
ex:" Hello world! ".trim() // "Hello world!"
```

---

**刪除或改變特定字元 Delete/Change char in string:**

javascirpt 中的字串是不能直接更動的（immutable），因此若要做到刪除、改變特定字元，就只能另外產生一個新的字串。
無效操作示範： `"hello"[0] = "H"`

可以使用 replace、slice 等函式產生新字串，或是將字串轉換成矩陣，經過處理後再接合成新字串，此外，如果需要反覆做出該操作，也可以考慮自己寫成一個新的函式。

```javascript

const replaceAt = function (index, replacement) {
return this.slice(0, index) + replacement + this.slice(index + 1);
};
```

---

**字串迴圈 String loop:**

可以對字串使用一般的 while、for 迴圈，但考量到字串無法直接修改，還是建議先轉換成矩陣後再使用矩陣內建的迴圈方法，操作上較為靈活。

---

**其他字串操作 Other string manipulations:**

```javascript

// toUpperCase、toLowerCase 轉大小寫
str.toUpperCase();
str.toLowerCase();
ex: "Hello!".toUpperCase(); // "HELLO!"
ex: "Hello!".toLowerCase(); // "hello!"

// padStart、padEnd 填充字串的長度
str.padStart(targetLength, padString);
str.padEnd(targetLength, padString);
ex: "123".padStart(8, "->"); // "->->-123"
ex: "Hi".padEnd(5, "~"); // "Hi~~~"

// repeat 重複字串
str.repeat(count);
ex: "Happy!".repeat(3); // "Happy!Happy!Happy!"

// charCodeAt 印出 UTF-16 Code
str.charCodeAt(index);
ex: "Apple".charCodeAt(0); // 65
```

---

## 2. Number 使用方法

**數字轉字串 Number to string:**

```javascript

//toString or String() 閱讀上比較直覺的好寫法
num.toString();
String(num);
ex: (3.14).toString(); // "3.14"
ex: String(3.14); // "3.14"

// 利用自動轉型，不太推薦此方法，但是在某些特定狀況可以順勢使用
"" + num;
ex: "" + 3.14; // "3.14"
```

---

**每ㄧ位數字轉矩陣 Number to array by digit:**

要先將數字轉換成字串再轉換成矩陣

```javascript
Array.from(num.toString(), Number); // 順便轉回數字
Array.from(num.toString());
ex: Array.from((3.14).toString(), Number); // [3, NaN, 1, 4]
ex: Array.from((3.14).toString()); // ["3", ".", "1", "4"]
ex: Array.from(3.14); // [] 轉換失敗
```

---

**數字運算 Number computing:**

```javascript
//power 次方
num1 ** num2;
Math.pow(num1, num2);
ex: 3 ** 4; // 3 的 4 次方 = 81
ex: Math.pow(3, 4); // 81

// 進位相關操作
// 四捨五入
Math.round(num);
num.toFixed(digits); // 會轉換成字串
// 無條件捨去
Math.floor(num);
// 無條件進位
Math.ceil(num);

// Math 要針對特定位數的進位操作有局限性，常見操作如下：
// 小數點後 2 位
let num = 3.14159;
console.log(Math.round(num * 100) / 100); // 3.14
console.log(Math.ceil(num * 100) / 100); // 3.15
console.log(Math.floor(num \* 100) / 100); // 3.14

// \* % operator，基礎實用的運算子
ex: (總秒數) => 小時, 分鐘, 秒;
let hours = Math.floor(seconds / 60 / 60);
let minutes = Math.floor((secondes / 60) % 60);
let rest_seconds = Math.floor(seconds % 60);
```

---

**轉換成數字是否成功 Conversion to number:**

使用`Number`是最為簡單且直覺的，但是在確認是否轉換成功時要使用`isNaN`，不能使用`typeof`(`typeof NaN`回傳結果是`"number"`)

```javascript
Number([1, 2, 3]); // NaN
Number([1]); // 1
Number("3.14"); // 3.14
Number("$100"); // NaN

isNaN(Number("$100")); // true
```

---

## 3. Array 使用方法

**矩陣迴圈 Array loop:**

array 自帶的迴圈已經可以應付大多數的使用時機，可讀性與方便性也比一般的 for、while 迴圈要來得好，可以說是一定要熟悉的語法。
記住：

> 如果在迴圈外面定義了一個變數，然後在迴圈裡面不斷的用它，八成都有更好的寫法

篇幅有限，矩陣自帶迴圈的用法與原理這裡不多贅述。

```javascript

arr.forEach((element,index,array)=>{})
arr.map((element,index,array)=>{}) // return new array
arr.filter((element,index,array)=>{}) // return new array
arr.reduce((accumulator,currentValue,currentIndex,array)=>{},initialValue) // return one value
```

---

**找尋矩陣中的元素 Find within array:**

```javascript

// find()，從頭找尋第一個符合條件元素，回傳元素
arr.find((element, index, array) => {})
ex: [0,50,2,70,1].find((element) => element > 10) // 50

// findIndex()，從頭找尋第一個符合條件元素，回傳元素的指標
arr.findIndex((element, index, array) => {})
ex: [0,50,2,70,1].findIndex((element) => element > 10) // 1

// findLast()，從尾找尋第一個符合條件元素，回傳元素
arr.findLast((element, index, array) => {})
ex: [0,50,2,70,1].findLast((element) => element > 10) // 70

// indexOf()，從頭找尋第一個符合條件的元素，回傳指標
arr.indexOf(element, fromIndex)
ex:
let arr = ["cat", "dog", "duck", "dog"]
arr.indexOf("dog") // 1
arr.indexOf("dog", 2) // 3
arr.indexOf("lion") // -1 (沒找到)

// includes ，找尋矩陣中是否有符合條件的元素，回傳真假值
arr.includes(element, fromIndex)
ex: ["cat", "dog"].includes("dog") // true
ex: ["cat", "dog"].includes("lion") // false
```

---

**矩陣轉字串 Convert array to string:**

```javascript

// join，回傳新 string，原 array 不變
// 產生具有某些符號規律字串時的好工具
arr.join();
ex: ["ab", "cd"].join(); //"ab,cd"
ex: ["ab", "cd"].join(""); // "abcd"
ex: ["2023", "01", "01"].join("-"); // "2023-01-01"

//toString() or String()，回傳新 string，原 array 不變
arr.toString();
ex: ["Hi", "baby!"].toString(); //"Hi,baby!"
ex: ["Hi", ["my", "love!"]].toString(); //"Hi,my,love!"
ex: String(["Hi", ["my", "love!"]]); //"Hi,my,love!"
```

---

**合併矩陣 Merge arrays:**

```javascript

// ES6 spread syntax ... 漂亮且直覺的寫法
[...arr1, ...arr2];
ex: [...[0, 1, 2], ...[3, 4, 5]]; //[0,1,2,3,4,5]

// concat(concatenate)，回傳新矩陣，原矩陣不變
arr1.concat(arr2);
ex: [1, 2].concat([3, 4]); // [1,2,3,4]

// push + ...
arr1.push(...arr2);
ex: [1, 2].push(...[3, 4]); // [1,2,3,4]
```

---

**矩陣元素的插入與刪除 Insertion and deletion of elements in array:**

除了 ES6 的 spread syntax ...，以及常見的 push、pop、Unshift、shift 以外，splice 的應用機會也是很多，使用時要注意 splice 是會改變原矩陣的！。

```javascript

splice(start, deleteCount, item1, item2, itemN)

ex:
const months = ["Jan", "March", "April", "June"];

months.splice(1, 0, "Feb"); // Inserts at index 1
console.log(months); // ["Jan", "Feb", "March", "April", "June"]

months.splice(4, 1, "May"); // Replaces 1 element at index 4
console.log(months); // ["Jan", "Feb", "March", "April", "May"]
```

---

**矩陣元素排列 Sort array:**

sort 原先是依據字串做排列，如果要做出依照數字大小排列的效果，就必須使用 function 作為引數。

```javascript

let arr = [20, 1, 50, 33, 0, 100];
arr.sort();
console.log(arr); // [ 0, 1, 100, 20, 33, 50 ]
arr.sort((a, b) => a - b);
console.log(arr); // [ 0, 1, 20, 33, 50, 100 ]
arr.sort((a, b) => b - a);
console.log(arr); // [ 100, 50, 33, 20, 1, 0 ]
```

---

**矩陣複製 Array duplication:**

矩陣的複製牽涉到 JavaScript 數值傳遞的原理，除了常見的 call by value、call by reference，還有特殊的 call by sharing，這會是一項需要另外探討的問題，這裡不做詳細討論，不過至少要知道`let newArr = oldArr`不是正確的複製方法。

---

**其他矩陣操作 Other array manipulations:**

```javascript

// 分解巢狀矩陣
[[], [[]]].flat(depth);
ex: [0, 1, [2, [3, 4]]].flat(2); // [0,1,2,3,4]
ex: [0, 1, [2, [3, 4]]].flat(Infinity); // [0,1,2,3,4]

// isArray 辨別是不是矩陣，使用 typeof 不太準確
Array.isArray(value);
ex: Array.isArray([1, 2]); // true
ex: Array.isArray("1,2"); // false
```

---

## 4. 操作範例 Demonstration

Q: 找出價格大於$200 的商品，並依據價格排列

```javascript

let food = [
  {name: "cake", price: 100},
  {name: "fruit", price: 300},
  {name: "water", price: 5},
];
let products = [
  {name: "car", price: 30000},
  {name: "book", price: 500},
  {name: "pencil", price: 10},
];
let highPrice = [...food, ...products].filter((e) => e.price > 200);
highPrice.sort((e1, e2) => e1.price - e2.price);

console.log(highPrice);
Output:
[
  { name: 'fruit', price: 300 },
  { name: 'book', price: 500 },
  { name: 'car', price: 30000 }
]
```

---

Q: 每個單字的開頭大寫

```javascript

let str = "cat dog duck lion rabbit snake";
let splitStr = str.split(" ").map((e) => e[0].toUpperCase() + e.slice(1));
str = splitStr.join(" ");
console.log(str); // Cat Dog Duck Lion Rabbit Snake
```

---

Q: 顯示產品項目的種類

```javascript

let products = ["book", "car", "car", "phone", "book", "computer"];
console.log(
products.filter((element, index, array) => array.includes(element, index + 1) === false)
);
// [ 'car', 'phone', 'book', 'computer' ]
```
