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
Polymorphic使用說明
<!--more-->

<!-- {{< toc >}} -->
---

## 1. 使用情境
使用Polymophic


## 2. 使用方法
同樣以Comment這個model舉例，建立Polymorphic model的指令：
`rails g model Comment content:text commentable:references{polymorphic}`
{{< codeblock "ruby" >}}
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
{{< /codeblock >}}


執行`rails db:migrate`在資料庫產生資料表與更新`schema`。
{{< codeblock "ruby" >}}
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
{{< /codeblock >}}
此時，查看`Comment`的模型內容：
{{< codeblock "ruby" >}}
# app/models/comment.rb
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end
{{< /codeblock >}}

Polymorphic會產生兩個欄位，`commentable_id`和`commentable_type`，`commentable_id`用以指向關聯模型的id，`commentable_type`用以記錄關聯模型的名字，例如`Post`、`Product`，
這邊假定已經有兩個`Model`，`Post`和`Product`，要為這些模型添加`has_many`或`has_one`的關聯：
{{< codeblock "ruby" >}}
# app/models/product.rb
class Product < ApplicationRecord
  has_many :comments, as: :commentable
end

# app/models/post.rb
class Post < ApplicationRecord
  has_one :comment, as: :commentable
end

{{< /codeblock >}}

## 3. 使用指令
{{< codeblock "ruby" >}}
product1 = Product.create(content: "product1")
comment1 = Comment.create(content: "nice product1", commentable: product1)
comment2 = Comment.create(content: "good product1", commentable: product1)
product1.comments
# => [#<Comment:0x000... id: 1, content: "nice product1", commentable_type: "Product", commentable_id: 1, created_at: xxx, updated_at: xxx>, #<Comment:0x000... id: 2, content: "good product1", commentable_type: "Product", commentable_id: 1, created_at: xxx, updated_at: xxx>]

post1 = Post.create(content: "post1")
comment2 = Comment.create(content: "nice post1", commentable: post2)
post1.comment
# => #<Comment:0x000... id: 3, content: "nice post1", commentable_type: "Post", commentable_id: 1, created_at: xxx, updated_at: xxx>

{{< /codeblock >}}


references:
- https://guides.rubyonrails.org/association_basics.html#polymorphic-associations
