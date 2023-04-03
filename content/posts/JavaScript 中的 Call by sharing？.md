---
title: "JavaScript 中的 Call by sharing "
date: 2023-04-03
categories:
  - JavaScript
tags:
  - JavaScript
  - Front-end
thumbnailImagePosition: left
thumbnailImage: images/js-garbriel.jpg
---
Call by value、Call by reference在一般的程式語言中是常見的基本觀念，但在JavaScript中多了一項Call by sharing，這裡做個簡單的說明。

<!--more-->

<!-- {{< toc >}} -->

## 1. Data Type

JavaScript 的資料類別可以分為：

- 基本型別（Primitive）：string、number、boolean、null、undefined
- 物件型別（Object）：object、symbol

基本型別會以純值的形式存在，而物件型別則是由零或不同型別的值所組成。

這邊之所以提及 Data Type 是因為資料類別會決定 call by value、call by reference、call by sharing 的運作時機。

## 2. 變數的真實樣貌

在提到 call by value 之前，我們先簡單講解平常看到的「變數」在程式中的真實樣貌。

當我們宣告一個變數時，如：`let num = 1234`，程式會自動分配一份記憶體空間，空間內部可以存放我們想要的資料，並且給予該空間一個名字。
以這段程式碼為例，`num`便是記憶體空間的名字（variable name），`123`是變數初始化後內部所存放的資料（variable value），另外還有我們平常看不到，用來標注空間所在位置的記憶體地址（memory address）。
{{< image classes="fancybox fig-100" src="images/js-garbriel.jpg" thumbnail="/images/variable_structure.jpeg" >}}

## 3. 基本型別 ＆ 物件型別

變數作為一個資料容器，當內部儲存的數值是基本型態時進行比較：
{{< codeblock "javasript" "js" "http://underscorejs.org/#compact" "js" >}}
let a = "Dog";
let b = "Dog";
let c = "Cat";
// 變數之間進行數值比較
console.log(a === b); //true
console.log(b === c); //false
{{< /codeblock >}}
基本型態下兩者的比較結果可想而知，比較式會取出兩變數內的「值」進行比較，但是當變數內部儲存的值是物件型態時，比較結果會有所不同：
{{< codeblock "javasript" "js" "http://underscorejs.org/#compact" "js" >}}
let a = {value: "Dog"};
let b = {value: "Dog"};
console.log(a === b); //false
{{< /codeblock >}}
明明變數 a, b 間的數值相同，但比較結果卻是false，
其實這就是 call by value 和 call by reference 之間的差異了。

## 4. Call by Value 
當變數內的數值是基本型別時，JavaScript 使用 call by value 的方式對變數內的值進行更新與傳遞。
{{< codeblock "javasript" "js" "http://underscorejs.org/#compact" "js" >}}
let a = 10
let b = a
console.log(`a=${a}, b=${b}`) // a=10, b=10

b = 20
console.log(`a=${a}, b=${b}`) // a=10, b=20
{{< /codeblock >}}
當 b 被初始化時，程式碼會分配一塊新的記憶體位置，並「複製」變數 a 內部的數值到新的記憶體空間中，因此儘管兩者的值是一樣的，兩個變數是實際上仍是指向完全不同的記憶體區塊，所以當變數 b 的數值更動時並不會影響到變數 a 內部的數值。
{{< image classes="fancybox fig-100" src="/images/passedByValue.png" thumbnail="/images/passedByValue.png" >}}


## 5. Call by reference
當變數內的數值是物件型別時，JavaScript 使用 call by reference 的方式對變數內的值進行更新與傳遞。
物件型別的資料與基本型別不同，可以彈性的進行擴充、修改與刪除，因此其記憶體空間的分配方式與單一變數不同，其實是多個不相鄰的記憶體空間串接而成，就像火車般由一節一節的車廂組成，我們可以隨意的新增、刪除車廂，或改變車廂的連接順序，變數所指向的不是車廂內的數值，而是火車的車頭，當我們要取用某節車廂的數值時，都要先透過車頭去找到對應的車廂，再取出其中的數值。
在下面的程式碼中，變數 a 與變數 b，都指向了同一個記憶體區塊，也就是同一輛火車頭，因此當其中的數值被修改時，透過 a 或 b 都會得到被修改後的結果（因為取得的數值都是來自同一節車廂）。
{{< codeblock "javasript" "js" "http://underscorejs.org/#compact" "js" >}}
let a = {value: 10}
let b = a
console.log(a === b) // true，a與b所指向的記憶體位置相同
console.log(`a.value=${a.value}, b.value=${b.value}`) // a.value=10, b.value=10

b.value = 20 // 透過 b 對其中一個資料區塊的數值進行修改
console.log(`a.value=${a.value}, b.value=${b.value}`) // a.value=20, b.value=20
{{< /codeblock >}}
{{< image classes="fancybox fig-100" src="/images/reference_train.png" thumbnail="/images/reference_train.png" >}}

如此便可以解釋先前的程式碼了：
{{< codeblock "javasript" "js" "http://underscorejs.org/#compact" "js" >}}
let a = {value: "Dog"};
let b = {value: "Dog"};
console.log(a === b); //false，a 與 b 指向的是完全不同的記憶體地址（兩者是不同的火車頭）
{{< /codeblock >}}

## 6. Call by sharing
Call by sharing可以說是call by reference中的特例，特點在於：當物件作為function的參數傳入時，如果對該物件在function內部中重新賦值，外部的物件的內容是不會被修改的。

範例1：function內部對物件進行修改
{{< codeblock "javasript" "js" "http://underscorejs.org/#compact" "js" >}}
obj1 = {value: 100}
function changeValue(obj) {
    obj.value = 300;
}
changeValue(obj1);
console.log(`obj1.value = ${obj1.value}`); // obj1.value = 300 數值被修變了
{{< /codeblock >}}

範例2：function內部對物件進行重新賦值
{{< codeblock "javasript" "js" "http://underscorejs.org/#compact" "js" >}}
obj1 = {value: 100}
function giveNewValue(obj) {
    obj = { value: 300 };
}
giveNewValue(obj1);
console.log(`obj1.value = ${obj1.value}`); // obj1.value = 100 數值不變
{{< /codeblock >}}

常見的 array 也是屬於物件型別，因此也符合 call by sharing 的運作規則：
{{< codeblock "javasript" "js" "http://underscorejs.org/#compact" "js" >}}
let array1 = [0, 1, 2];
let array2 = [0, 1, 2];

function changeArrValue(arr) {
    arr.push(4);
}
function giveNewArrValue(arr) {
    arr = [0, 0, 0, 0, 0, 0];
}

changeArrValue(array1); // 修改矩陣
console.log(`array1 = ${array1}`); // array1 = [0,1,2,4] 數值被修變

giveNewArrValue(array2); // 矩陣重新賦值
console.log(`array2 = ${array2}`); // array2 = [0,1,2] 數值依然不變
{{< /codeblock >}}
