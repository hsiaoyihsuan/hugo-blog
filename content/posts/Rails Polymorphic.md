---
title: "Rails Polymorphic"
date: 2023-10-12
categories:
  - Rails
tags:
  - Rails
  - Back-end
  - Polymorphic
thumbnailImagePosition: left
thumbnailImage: images/RailsLogo.png
---
如何使用 Polymorphic Associations 多型關聯。

## 1. 使用情境
想像你正在設計「評論」功能的資料庫架構，使用者可以在幾乎任何地方留下評論，例如產品、貼文、活動等，此時你會想到使用一對多關係，為這三個 Model 設計出`ProductComment`、`PostComment`、`EventComment`，但此時你發現這三個資料表的欄位幾乎一模一樣，如果分成三個 Model 顯得相當冗餘，此種情景就相當適合使用**Polymorphic 多型關聯**來簡化資料庫的設計，使用Polymophic可以使模型在同一個關聯上屬於多個模型。

## 2. 使用方法
同樣以Comment這個model舉例，建立Polymorphic model的指令：
`rails g model Comment content:text commentable:references{polymorphic}`

觀察一下產生的 migration 檔：
```ruby
# 產生的migration檔案，references版本
class CreateComments < ActiveRecord::Migration[7.0]
  def change
    create_table :comments do |t|
      t.text :content
      t.references :commentable, polymorphic: true, null: false

      t.timestamps
    end
  end
end
# 產生的migration檔案，較複雜的版本
class CreateComments < ActiveRecord::Migration[7.0]
  def change
    create_table :comments do |t|
      t.string  :name
      t.bigint  :commentable_id
      t.string  :commentable_type
      t.timestamps
    end

    add_index :comments, [:commentable_type, :commentable_id]
  end
end
```


接著執行`rails db:migrate`在資料庫產生資料表與更新`schema`。
```ruby
# schema
ActiveRecord::Schema[7.0].define(version: 2023_10_12_155217) do
  create_table "comments", force: :cascade do |t|
    t.text "content"
    t.string "commentable_type", null: false
    t.integer "commentable_id", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["commentable_type", "commentable_id"], name: "index_comments_on_commentable"
  end
  ...
end
```
查看`Comment`的模型內容：
```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end
```

此時會產生兩個欄位，`commentable_id`和`commentable_type`，`commentable_id`用以指向關聯模型的id，`commentable_type`用以記錄關聯模型的名字，例如`Post`、`Product`。

接著建立與 Comment 的關聯。
```ruby
# app/models/product.rb
class Product < ApplicationRecord
  has_many :comments, as: :commnetable
end

# app/models/post.rb
class Post < ApplicationRecord
  has_many :comments, as: :commnetable
end

# app/models/event.rb
class Event < ApplicationRecord
  has_many :comments, as: :commnetable
end

```

## 3. 資料庫指令
建立與 Commnet 的指令很多種，可以間單的透過`commentble`，或是直接寫入`commentable_id`、`commentable_type`。

```ruby
# 使用commentable
product = Product.create(content: "product")
# => Product Create (1.4ms)  INSERT INTO "products" ("content", "created_at", "updated_at") VALUES (?, ?, ?)  [["content", "product"], ["created_at", "2023-10-16 14:09:23.854703"], ["updated_at", "2023-10-16 14:09:23.854703"]]

Comment.create(content: "nice product", commentable: product)
# => Comment Create (0.6ms)  INSERT INTO "comments" ("content", "commentable_type", "commentable_id", "created_at", "updated_at") VALUES (?, ?, ?, ?, ?)  [["content", "nice product"], ["commentable_type", "Product"], ["commentable_id", 1], ["created_at", "2023-10-16 14:11:57.300843"], ["updated_at", "2023-10-16 14:11:57.300843"]]

# 直接寫入comment_id & comment_type
Product.create(content: "product2")
Comment.create(content: "nice product2", commentable_id: 2, commentable_type: "Product")
```


references:
- https://guides.rubyonrails.org/association_basics.html#polymorphic-associations
