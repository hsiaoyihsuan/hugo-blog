---
title: "Rails ORM 常見的CRUD指令"
date: 2023-04-04
categories:
  - Rails
tags:
  - Rails
  - Back-end
  - CRUD
thumbnailImagePosition: left
thumbnailImage: images/RailsLogo.png
---
CRUD = Create Read Update Delete是網站開發中的基本，在此筆記常用到的Rails CRUD指令。
<!--more-->

<!-- {{< toc >}} -->

## Create
- `new`：產生新資料，但不會存檔
- `create`：產生新資料，會直接存檔
- `create!`：與create相同，但過程中發生錯誤時會報錯
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
new_book = Book.new(name: "老人與海", price: 300)
new_book.save

Book.create(name: "Les Misérables", author: "Victor Hugo")
{{< /codeblock >}}

## Read
- `all`：一次抓取所有資料
- `limit()`：抓去特定數量資料
- `first & last`：找到該資料類型中的第一筆 & 最後一筆資料
- `find_by(id: 1)`：找到第一筆符合條件的資料，找不到會回傳`nil`
- `find_by!(id: 1)`：與find_by相同，但是找不到資料會報錯
- `find(1)`：依據id找尋資料，找不到資料會報錯
- `select('name')`：只抓出資料表中的特定欄位（資料表欄位太多時可以節省記憶體空間）
- `where()`：以陣列的形式回傳找到的多筆符合條件的資料，找不到回傳空矩陣
- `find_by_sql()`：使用sql語法查詢（較少使用）
- `order('price AESC')`：依照遞增或遞減順序抓取所有資料
- `order(price: :aesc)`：order的不同寫法
- `find_each`：batch find寫法，預設先抓出1000筆資料，當資料量太多時使用
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
Book.all
Book.limit(5) // 抓取前五筆資料
Book.offset(5).limit(5) // 抓取下五筆資料
Book.first
Book.last
Book.find(id: 10)
Book.find_by(id: 10, author: "Victor Hugo") // 依據多個條件尋找 and
Book.find_by!(id: 9999) //報錯： in `find_by!': Couldn't find Book (ActiveRecord::RecordNotFound)

Book.find(10)
Book.select('id', 'name')
Book.select('id', 'name').find(10) //select搭配其他方法使用
Book.select('type').distinct //distinct會剔除重複的資料
Book.find_by_sql("select * from books where id = 10")
Book.order('price AESC') // AESC遞減，ASC遞增
Book.order(price: :aesc)
Book.order('price AESC').first //抓到價格最低的物件

Book.find_each do |book|
  ...
end

Book.where('price < 500 and price > 250')
Book.where(price: 500, name: '老人與海')
Book.where.not(price: 1000).or Book.where(name: ['老人與海', 'Two walls'])
Book.where(["name = :name and price < :price",{name: "老人與海", price: "1000"}])
Book.where("name LIKE ?","%#{params[:bookname]}%") //部分匹配，抓取書籍名稱中與網頁參數部分相同者 
{{< /codeblock >}}
以下指令可以解決Ruby迴圈執行效率太低的問題：
- `count`：回傳資料總數
- `average()`：回傳該欄位資料的平均
- `sum()`：回傳該欄位資料的總和
- `maximum()`：回傳全部資料中該欄位的最大值
- `minimum()`：回傳全部資料中該欄位的最小值
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
Book.count
Book.sum(:price)
Book.average(:price)
Book.maximum(:price)
Book.minimum(:price)
{{< /codeblock >}}

## Update
- `save`：儲存資料到資料庫中
- `update()`：更新資料到資料庫中
- `update_attribute()`：效果同update
- `update_all()`：更新全部資料（謹慎使用!）
- `increment & decrement`：新增或減少1
- `toggle`：切換boolean值
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
b2 = Book.find(2)
b2.name = "Finance"
b2.save

b2.update(name: "Commerial")
Book.update_all(price: 1)

b2.increment(:price).save // 單價+1
b2.decrement(:price).save // 單價-1

b2.toggle(:is_online).save 
{{< /codeblock >}}

## Delete
- `delete`：刪除資料
- `destroy`：刪除資料，會經歷Callback
- `destroy_all()`：刪除符合條件的資料
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
b1 = Book.first
b1.destroy
b1.delete

Book.delete(1)
Book.destroy_all("price < 400")
{{< /codeblock >}}
