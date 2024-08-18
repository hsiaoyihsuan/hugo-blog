---
title: "Rails N+1問題 with joins, includes, preload and eager_load"
date: 2023-06-08
categories:
  - Rails
  - SQL
tags:
  - Rails
  - Back-end
  - CRUD
  - SQL
thumbnailImagePosition: left
thumbnailImage: images/RailsLogo.png
---
資料庫的操作是影響網站效能的重要因素，而在存取資料時幾乎一定會遇到 N+1 問題。本篇文章將簡要探討 includes 的使用時機，以及與其相似的 preload 和 eager_load 方法。此外，我們還會介紹在處理多表格操作時常用的 joins。

---

## 1. 造成 N+1 問題的原因
我們先想像一個情境，你正在開發商家的訂單系統，其中有兩個 Model，分別是`Customer`與`Order`，`Customer`記載顧客資料，並且每個顧客擁有多筆訂單`Order`
![customerOrders](/images/customerOrders.png)
```ruby
# customer.rb
class Customer < ApplicationRecord
  has_many :orders
end

# order.rb
class Order < ApplicationRecord
  belongs_to :customer
end
```

現在我們想要呈現4位顧客的個人資訊以及他們各自的所有訂單，你可能會這樣寫：
![fourCustomerOrders](/images/fourCustomerOrders.png)
```ruby
Customer.limit(4).each{|customer| puts customer.orders}
```
Rails ORM 產生的 SQL 指令如下：
```sql
SELECT 'customers'.* FROM 'customers' LIMIT 4
SELECT 'orders'.* FROM 'orders' WHERE 'orders'.'customer_id'=1
SELECT 'orders'.* FROM 'orders' WHERE 'orders'.'customer_id'=2
SELECT 'orders'.* FROM 'orders' WHERE 'orders'.'customer_id'=3
SELECT 'orders'.* FROM 'orders' WHERE 'orders'.'customer_id'=4
```
從中我們可以觀察到總共產生了5個 SQL queries。首先，第一個查詢擷取了4位顧客的資料，然後又產生了額外的4個查詢來擷取每個顧客的所有訂單。這就是典型的N+1問題，這裡的 N 等於4。當需要擷取的顧客數量增加時，資料庫的效能就會顯著地下降。

## 2. 解決 N+1問題
在 Rails ActiveRecord 的官方手冊中提到我們可以使用`:includes`、`:preload`、`:eager_load`來解決 N+1 問題，其本質精神都是"eager load"，透過一次性預先加載相關聯的資料來代替頻繁的向資料庫發送查詢指令。
實務上使用`:includes`即可代替`:preload`和`:eager_load`，Rails會根據使用者的查詢方式產生與`:preload`或`:eager_load`相同的 SQL 查詢指令，因此這裡會將三者放在一起討論。

### 2.1 使用 :includes 和 :preload
絕大多數情況下，`:includes`會使用與`:preload`相同的 SQL 查詢，最終產生2條queries：
1. 存取主資料表的資料
2. 加載關聯的數據

```ruby
Customer.includes(:orders).limit(4).each{|customer| puts customer.orders}
```
```sql
SELECT 'customers'.* FROM 'customers' LIMIT 4
SELECT 'orders'.* FROM 'orders' WHERE 'orders'.'customer_id' IN (1, 2, 3, 4)
```
在這種情況下，ORM 指令可以從`:includes`代換成`:preload`，其效果是相同的：
```ruby
Customer.preload(:orders).limit(4).each{|customer| puts customer.orders}
```

### 2.2 使用 :includes 和 :eager_load
當我們的查詢會附帶條件子句時（ex: `where`、`order`），此時就會需要使用`:eager_load`或`:includes`，而無法使用`:preload`了。與`:preload`不同，此種情境下的`:includes`和`:eager_load`只會產生一條 SQL query，並且是透過 `LEFT OUTER JOIN`進行資料庫的操作。
```ruby
Customer.includes(:orders).where(orders:{status: 'paid'}).limit(4).each{|customer| puts customer.orders}.references(:orders)
or
Customer.eager_load(:orders).where(orders:{status: 'paid'}).limit(4).each{|customer| puts customer.orders}
```
注意上述的`:includes`雖然與`:eager_load`基本相同，但`:includes`還必須額外加上`references`來強迫連結兩個資料表。
```sql
SELECT 'customers'.'id' AS t0_r0, 'customers'.'name' AS t0_r1, 'orders'.'id' AS t1_r0, 'orders'.'status' AS t1_r1
FROM 'customers'
LEFT OUTER JOIN 'orders'
ON 'orders'.'customer_id' = 'customers'.'id'
WHERE 'orders'.'status' = 'paid' AND 'customers'.'id' IN (1, 2, 3, 4)
```
可以觀察到`:eager_load`透過`LEFT OUTER JOIN`在`Customer`和`Order`之間建立了一個中間資料表，用以建構模型的輸出。
![customerLeftJoin](/images/customerLeftJoin.png)

讀到到這裡，我們已經了解以上三個方法是如何解決典型的 N+1 問題了，其實也沒有想像中複雜，那接著可以討論另一個也很常提及的`:joins了`。

## 3. How about :joins
Rails中的`:joins`是透過 SQL 的`INNER JOIN`來實作，可以將兩個表格依照條件進行連結，進而建立出一個新的中間資料表以提供使用者進一步分析。前面我們在闡述如何解決 N+1 問題時並沒有提到使用 `:joins`，原因是`:joins`不會真的將關聯的資料取出。即使使用`:joins`，在後續存取資料時，仍然可能導致 N+1 問題的發生。例如以下的程式碼：
```ruby
Customer.joins(:orders).each{|customer| puts customer.orders}
```
```sql
SELECT 'customers'.* FROM 'customers' INNER JOIN 'orders' ON 'customers'.'id' = 'orders'.'customer_id'
SELECT 'orders'.* FROM 'orders' WHERE 'orders'.'customer_id' = 1
SELECT 'orders'.* FROM 'orders' WHERE 'orders'.'customer_id' = 2
SELECT 'orders'.* FROM 'orders' WHERE 'orders'.'customer_id' = 3
SELECT 'orders'.* FROM 'orders' WHERE 'orders'.'customer_id' = 4
```
可以看到 N+1 queries 還是發生了，並沒有因為使用了`:joins`而解決，因此還是要使用上述的`:includes`、`:preload`或`:eager_load`。
### 使用 :joins 的時機
既然`:joins`不能解決 N+1 問題，那什麼時候才會用到這個方法呢？

這個問題沒有絕對的答案，不過常見的使用時機是：當我們只是想要篩選的結果，或是觀察多個表格間的性質時就可以使用`:joins`。由於`:joins`僅進行表格連接，而不會實際存取物件本身，所以它的執行效能通常比`:includes`等方法更高。以下就是一個使用例子：
```ruby
# 計算四位顧客所有已付款(paid)訂單的數量
Customer.joins(:orders).where(orders: {status: 'paid'}).limit(4).count
```
```sql
SELECT COUNT(*)
FROM (
  SELECT 1 AS one
  FROM 'customers' INNER JOIN 'orders'
  ON 'orders'.'customer_id' = 'customers'.'id'
  WHERE 'orders'.'status' = 'paid' LIMIT 4
)
```

---

References:
- https://medium.com/@jinghua.shih/rails-activerecord-%E6%95%88%E8%83%BD%E5%84%AA%E5%8C%96-%E4%B8%8A-%E9%97%9C%E8%81%AF%E6%9F%A5%E8%A9%A2-75ca79f510b3ruby-on-rails-switch-from-sqlite3-to-postgres-590009645c25
- https://medium.com/gusto-engineering/a-visual-guide-to-using-includes-in-rails-700a91cd3095
- https://guides.rubyonrails.org/active_record_querying.html#group
