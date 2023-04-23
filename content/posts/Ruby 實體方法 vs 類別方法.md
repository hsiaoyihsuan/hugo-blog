---
title: "Ruby 實體方法 vs 類別方法"
date: 2023-04-23
categories:
  - Ruby
tags:
  - Ruby
  - Rails
  - Back-end
thumbnailImagePosition: left
thumbnailImage: images/RubyLogo.png
---

快速了解 Ruby 中 實體方法(Instance method) 與 類別方法(Class method)的差異。

<!--more-->

<!-- {{< toc >}} -->

## 方法 Method

Ruby 世界的方法(Mehod)可以對應到其他程式語言中的函式(function)，將一些行為定義在方法中，藉由帶入引數並執行方法的過程來簡化程式的撰寫，以及達到程式碼重複利用的效果。Ruby 中的方法由`def`開頭，並且以小寫、蛇式的方式命名，例如下方範例中，簡單定義了一個方法並於外部呼叫，方法執行結束前會回傳最後ㄧ行的執行結果。
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
def calculate_bmi(weight_kg, height_m)
  weight_kg / height_m ** 2
end
p calculate_bmi(60, 1.75) # 19.591836734693878
{{< /codeblock >}}

## 實體方法 ＆ 類別方法
實體(Instance)是由類別(Class)為模板所產生的獨立物件，他可以使用定義在類別中的實體方法，但是不能使用定義在類別中的類別方法。同樣的，類別本身可以使用類別方法，
但是不能使用實體方法。

簡單來說，實體方法與類別方法的使用對象不同，實體方法 for 實體，類別方法 for 類別，雖然方法都是定義在類別中，但是兩者是不能混用的。
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
class Student
  def initialize(name)
    @name = name
  end

  def self.say_classroom_name
    p "All students are in Room 311"
  end

  def say_my_name
    p "My name is #{@name}"
  end
end

student01 = Student.new("Lisa")
student02 = Student.new("Kevin")

student01.say_my_name
student02.say_my_name

Student.say_classroom_name

>>>>>>>>輸出：
"My name is Lisa"
"My name is Kevin"
"All students are in Room 311"
>>>>>>>>

student01.say_classroom_name 
>>>>>>>>>>錯誤訊息：
undefined method `say_classroom_name' for #<Student:0x00000001002ea440 @name="Lisa"> (NoMethodError)
>>>>>>>>>

Student.say_my_name 
>>>>>>>>>>錯誤訊息：
undefined method `say_my_name' for Student:Class (NoMethodError)
>>>>>>>>>>
{{< /codeblock >}}
實體方法與類別方法在實作上的差異是：類別方法會在方法名稱前加上`self`，`self`在class中會指向class本身，可以想成加上`self`就是指名這個方法只能給類別所使用。

上述的程式碼中，say_classroom_name是類別方法，只能由Student這個類別使用，say_my_name是實體方法，只能給各個實體所使用，另一方想使用不屬於自身的方法，系統會跑出**找不到該方法**的錯誤訊息。

在Rails中其實很常使用到類別方法，例如在透過Model對資料庫的資料進行操作時：`User.all #撈出所有使用者資料`

## Self 的位置
要特別注意一點，類別方法的`self`是放在方法名稱前，如果`self`是放在方法中（如下方範例），此時的`self`就不是指向class物件而是實體本身了，因此下方的who_am_I?依然是一個實體方法。
{{< codeblock "archives.rb" "ruby" "http://underscorejs.org/#compact" "archives.rb" >}}
class Student
  ...

  def who_am_I?
    p self
  end
end

student01 = Student.new("Lisa")
student01.who_am_I?
>>>>>>>>輸出：
 #<Student:0x0000000102569f70 @name="Lisa">
>>>>>>>>>

{{< /codeblock >}}

