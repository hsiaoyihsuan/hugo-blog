---
title: "Laravel 關聯 Model"
date: 2024-01-19
categories:
  - Laravel
tags:
  - Laravel
  - ORM
  - Back-end
thumbnailImagePosition: left
thumbnailImage: images/laravel.svg
---
Laravel 關聯性Model基礎操作：belongs_to, has_one, has_many, many to many

---

## 1. 一對一關係 has_one

假設兩個Model：店主Owner、商店Store，店主擁有一間商店(Owner has one Store)，商店屬於一位店主(Store belongs to Owner)。

```
Owner:
- id
- name

Store:
- id
- name
- owner_id
```

首先要為Store創建欄位`owner_id`，在migration中可以使用`$table->foreignId('owner_id')`建立欄位(foreignId等同於unsignedBigInt)。

`Owner.php`中寫入`hasOne`：

```php
// app/Models/Owner.php
public function store() {
    return $this->hasOne(Store::class);
}
```

`Store.php`中寫入`belonsTo`：

```php
// app/Models/Store.php
public function owner() {
    return $this->belongsTo(Owner::class);
}
```

使用方法：

```php
// 從Owner角度建立Store
// 方法一：使用create
$o1 = Owner::find(1);
$o1->store()->create(['name'=>'store1']);

// 方法二：使用save
$o1 = Owner::find(1);
$s2 = new Store(['name'=?'store2']);
$o1->store()->save($s2);

// 從Store角度建立Owner
$o2 = Owner::find(2);
$s3 = new Store(['name'=>'store3']);
$s3->owner()->associate($o2);
$s3->save();

// 撈取該Owner的Store
$o1->store;
$o1->store->name;

// 撈取該Store的Owner
$s1->owner;
$s1->owner->name;
```

`hasOne`與`belongsTo`都可以獨立設置foreign key或owner key：

```php
public function owner()
{
    return $this->belongsTo(Owner::class, 'foreign_key', 'owner_key');
}
```

```php
public function store()
{
    return $this->hasOne(Store::class, 'foreign_key', 'owner_key');
}
```

## 2. 一對多關係 has_many

假設兩個Model：商店Store、書籍Book，商店擁有多本書籍(Store has many books)，書籍屬於一個商店(Book belongs to Store)。

```
Store:
- id
- name

Book:
- id
- name
- store_id
```

為Book創建欄位`store_id`。

在`Store.php`中寫入`hasMany`：

```php
public function books() {
    return $this->hasMany(Book::class);
}
```

在`Book.php`中寫入`belongsTo`：

```php
public function store() {
    return $this->belongsTo(Store::class);
}
```

使用方式：

```php
// 從Store角度建立Book
// 方法一：使用create
$s1 = Store::find(1);
$s1->books()->createMany([
    ['name'=>'book1'],
    ['name'=>'book2']
]);

// 方法二：使用save
$s1 = Store::find(1);
$s1->books()->saveMany([
    new Book(['name'=>'book4']),
    new Book(['name'=>'book5'])
]);

$books = Book::where('name', 'like', 'book%')->get();
$s1->books()->saveMany($books);

// 從Book角度建立Store
$b1 = new Book(['name'=>'book3']);
$s1 = Store::find(1);
$b3->store()->associate($s1);
$b3->save();

// 撈取該Store的Book
$s1->books;
$s1->books()->where('name', 'like', 'book%')->get();

// 撈取該Book的store
$b1->store;
```

`hasMany`可以獨立設置foreign key和owner key：

```php
return $this->hasMany(Comment::class, 'foreign_key', 'local_key');
```

## 3. 多對多關係 Many to Many

假設兩個Model：書籍Book、作者Author，書籍有多位作者(book has many authors)，作者有撰寫多本書籍(author has many books)，此時會需要中間資料表author_book用來紀錄兩者的關係，要有欄位`book_id`、`author_id`，表格名稱內的Model名稱是依照字母排序。

建立中間資料表author_book：

```bash
php artisan make:migration create_author_book_table

```

```php
Schema::create('author_book', function (Blueprint $table) {
    $table->id();
    $table->foreignId('author_id');
    $table->foreignId('book_id');
});
```

接著在`Book`與`Author`都加上`belongsToMany`：

```php
// Book.php
public function authors(){
    return $this->belongsToMany(Author::class);
}

// Author.php
public function books(){
    return $this->belongsToMany(Book::class);
}
```

一樣可以單獨設定第三方資料表名稱與key名稱：

```php
return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');
```

使用方法：

```php
// 兩筆資料建立關係
// 方法ㄧ：使用save, saveMany
$b1 = Book::find(1);
$a1 = Author::find(1);
$b1->authors()->save($a1);

$authors = Author::whereIn('id', [2, 3])->get();
$b1->authors()->saveMany($authors);

// 方法ㄧ：使用attach, detach
$b1 = Book::find(1);
$a1 = Author::find(1);
$b2->authors()->attach($a2->id); // add relation
$b2->authors()->detach($a2->id); // remove relation

// 撈取資料
$b1->authors;
$a1->books;
```
---

## references
- https://laravel.com/docs/8.x/eloquent-relationships
