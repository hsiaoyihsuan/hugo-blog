---
title: "Rails 基礎 ORM 的 SQL 語法"
date: 2023-06-04
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
儘管Rails開發者可以透過 ActiveRecord 簡單地操作資料庫資源，但仍然需要了解基本的 SQL 語法。以下是常見 ORM 指令的翻譯，可以幫助你真正理解背後的運作原理。

較複雜的資料庫關係語法（例如：includes、join）將不在條列於本文中，我們將在另一篇文章中探討這些內容。
<!--more-->

<!-- {{< toc >}} -->
---

## Create
- create:
使用 create 創建資料，或透過 new 產生物件之後再使用 save，以上狀況所產生的SQL語法如下。
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
Book.create(name: "學習Rails", author: "Eddie", intro: "學習Rails好幫手", price: 100)
or
Book.new(name: "學習Rails", author: "Eddie", intro: "學習Rails好幫手", price: 100)
Book.save
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
INSERT INTO books(name, author, intro, price, updated_at) VALUES ('學習Rails', 'Eddie', '學習Rails好幫手', 100, '2023...')
{{< /codeblock >}}

---
## Read

- **all:**
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
Book.all
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
SELECT * FROM books
{{< /codeblock >}}

---
- **select or pluck:**
select 和 pluck 最終得到的資料形式不同，這裡不多做討論，只要知道Rails進一步的處理是在從資料庫抓取資料後，兩者前期的SQL語法沒有不同
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
Book.select("name")
Book.pluck("name")
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
SELECT name FROM books
{{< /codeblock >}}

---
- **find or find_by:**
find 與 find_by 本質上都是使用 WHERE 進行查詢，但是額外使用 LIMIT 限制最終僅有一筆資料回傳
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
Book.find(2)
Book.find_by(name: "老人與海")
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
SELECT * FROM books WHERE id = 2 LIMIT 1
SELECT * FROM books where name = '老人與海' LIMIT 1
{{< /codeblock >}}

---
- **where:**
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
# where
Book.where(name: "老人與海")

# where NOT
Book.where.not(name: "老人與海")
Book.where.not(id: [1, 2])

# where OR
Book.where("price > 200").or Book.where(id: 2)

# where AND
Book.where("price > 200 and name = 老人與海")
Book.where(price: 1000, name: "老人與海")
Book.where(id: [1, 2])
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
-- where
SELECT * FROM books WHERE (name = '老人與海')

-- where NOT
SELECT * FROM books WHERE (name != '老人與海')
SELECT * FROM books WHERE id NOT IN (1, 2)

-- where OR
SELECT * FROM books WHERE (name = '老人與海' OR id = 2)

-- where AND
SELECT * FROM books WHERE (price > 1000 AND name = '老人與海')
SELECT * FROM books WHERE (price = 1000 AND name = '老人與海')
SELECT * FROM books WHERE id IN (1, 2)
{{< /codeblock >}}

---
- **like:**
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
Book.where("name like ?", "%與%")
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
SELECT * FROM books WHERE (name like '與')
{{< /codeblock >}}

---
- **order:**
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
Book.order(price: :DESC)
Book.where(id: [1, 2]).order(price: :ASC)
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
SELECT * FROM books ORDER BY price DESC
SELECT * FROM books WHERE id IN (1, 2) ORDER BY price ASC
{{< /codeblock >}}

---
- **first & last:**
first 和 last 都會先進行排序，再依照數量透過 LIMIT 抓取資料。
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
Book.first
Book.last
Book.first(3)
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
SELECT * FROM books ORDER BY id ASC LIMIT 1
SELECT * FROM books ORDER BY id DESC LIMIT 1
SELECT * FROM books ORDER BY id ASC LIMIT 3
{{< /codeblock >}}

---
## Update
- **update:**
該筆資料更新時，updated_at 也會順帶一起更新
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
book4 = Book.find 4
book4.update(name: "Book4", price: 444)
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
UPDATE books SET name = 'Book4', price = 444, updated_at = "2023..." WHERE id = 4
{{< /codeblock >}}

---
- **update_all:**
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
books = Book.where(id: [1, 2])
books.update_all(author: "unknown")
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
UPDATE books SET author = 'unknown' WHERE id IN (1, 2)
{{< /codeblock >}}

---
## Delete 
- **destroy:**
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
book = Book.find 4
book.destroy
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
DELETE FROM books where id = 4
{{< /codeblock >}}

---
- **destroy_all:**
與 destroy 相同，依照 id 將每筆資料刪除
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
books = Book.all
books.destroy_all
{{< /codeblock >}}
{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
DELETE FROM books WHERE id = 1
DELETE FROM books WHERE id = 2
...
DELETE FROM books WHERE id = 100
{{< /codeblock >}}

---

## 進階：sum, count, average, maximum, minimum
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
Book.sum(:price)
Book.count
Book.average(:price)
Book.maximum(:price)
Book.minimum(:price)
{{< /codeblock >}}

{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
SELECT SUM(price) FROM books
SELECT COUNT(*) FROM books
SELECT AVG(price) FROM books
SELECT MAX(price) FROM books
SELECT MIN(price) FROM books
{{< /codeblock >}}

---

## 結語

在了解 Rails 的 ORM 如何轉譯成 SQL 語法之後，我們可以更深入地理解 ActiveRecord 的運作原理，並避免寫出冗余的語句。

以下是一個例子，它實際上對資料庫進行了兩次查詢，建議將程式碼整理成一行，以改善效能。

錯誤示範：
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
books = Book.where("price > 100")
books.order(price: :ASC)
{{< /codeblock >}}

{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
SELECT * FROM books WHERE price > 100
SELECT * FROM books WHERE price > 100 ORDER by price ASC
{{< /codeblock >}}

優化後寫法：
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
books = Book.where("price > 100").order(price: :ASC)
{{< /codeblock >}}

{{< codeblock "archives.sql" "sql" "http://underscorejs.org/#compact" "archives.sql" >}}
SELECT * FROM books WHERE price > 100 ORDER by price ASC
{{< /codeblock >}}

