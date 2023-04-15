---
title: "Rails 使用 PostgreSQL"
date: 2023-04-15
categories:
  - Rails
tags:
  - Rails
  - Back-end
  - PostgreSQL
  - SQL
thumbnailImagePosition: left
thumbnailImage: images/RailsLogo.png
---
Rails 預設的資料庫是 Sqlite，如欲使用較為專業的開源資料庫 **PostgreSQL**，可參照此說明筆記進行操作。
<!--more-->

<!-- {{< toc >}} -->
## 安裝PostgreSQL on Mac
1. 使用 Homebrew套件管理工具安裝： `$ brew install postgresql`（預設安裝postgreSQL v14）
2. 安裝後重新啟動`$ brew services restart postgresql`
3. 重新啟動Terminal
4. 查看版本`$ postgres --version`或`$ psql --version`

---

## 在 Rails 專案建立 PostgreSQL 資料庫
### Pre-requisites：
- 確認已經在本機中安裝PostgreSQL
- 本機中必須保持運行PostgreSQL，可透過`$ brew service list`查看運行狀況。
 :::info
postgresql@14 started hsuan ~/Library/LaunchAgents/homebrew.mxcl.postgresql@14.plist
 :::
 相關啟動PostgreSQL操作指令：
1. 第一種方法是手動啟動和停止PostgreSQL，每次需要輸入相應的指令。
```bash!
psql -D /usr/local/var/postgres start
psql -D /usr/local/var/postgres stop
```
2. 使用 Homebrew 服務管理工具啟動和停止PostgreSQL。此方法將 PostgreSQL 服務作為系統常駐運行的服務，可在在系統啟動後自動啟動。
```bash!
$ brew services start postgresql
$ brew services stop postgresql
$ brew services restart postgresql
```

### Step1：安裝pg gem
- 方法1：建立新專案時就選擇使用 PostgreSQL：`$ rails new myapp --database=postgresql`
- 方法2：單純new新專案，接著安裝pg gem： `$ bundle add pg`+`$ bundle install`
方法1較為推薦，可以讓rails自動幫妳生成相關設定，如果是使用方法2，則要將原先在Gemfile中的`gem sqlite3`手動移除，只保留`gem pg`（通常一個專案僅需要一個資料庫）
### Step2：設定資料庫 adapter
接著設定 Rails 專案的 config/database.yml（如果是使用--database=postgresql，Rails會自動設定好這部分的檔案）。
如果原本是sqlite，可以使用指令切換資料庫來產生database.yml`$ rails db:system:change --to postgresql`，或是自行手動改寫檔案內容。
database.yml
```yaml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch('RAILS_MAX_THREADS') { 5 } %>

development:
  <<: *default
  database: YOUR_PROJECT_NAME_WITH_PG_development

test:
  <<: *default
  database: YOUR_PROJECT_NAME_WITH_PG_test

production:
  <<: *default
  database: YOUR_PROJECT_NAME_WITH_PG_production
  username: YOUR_PROJECT_NAME_WITH_PG
  password: <%= ENV['YOUR_PROJECT_NAME_WITH_PG_DATABASE_PASSWORD'] %>
```
### Step3：建立專案資料庫
有三種狀況：
- 狀況一：還沒建立資料庫，可以從rails專案建立對應的資料庫 (database.yml必須先設定好)：進入到rails console後使用指令`$ rails db:create`來自動建立資料庫，資料庫名稱會命名為`<專案名稱>_development`
- 狀況二：還沒建立資料庫，透過`$ psql`自行建立資料庫（相關指令於文章下方），資料庫名稱需與database.yml的設定相符
- 狀況三：已經有原先的資料庫，並且有 `schema` 和 `seed`需要進行資料轉移，可以執行：`$ rails db:setup` 其效果等於 `db:create`+`db:schema:load`+`db:seed`

### Step4：執行 migration 啟動伺服器
1. 執行`$ rails db:migrate`
2. 執行`$ rails server`

---

## 資料庫終端機其餘操作
**解除安裝：**`$ brew uninstall postgres`

**查看所有資料庫資訊：**`$ pqsl -list` ，postgresSQL安裝後的默認資料庫是`postgres`

**建立或刪除資料庫**：
```bash!
$ createdb <new_database> # 新增資料庫
$ dropdb <database> # 刪除資料庫
```

**進入資料庫：**`$ psql <資料庫名稱>`
```
psql (14.7 (Homebrew))
Type "help" for help.
<資料庫名稱>=#
```
```bash!
\q = exit
\l = list
```
## 其他：
圖形界面軟體推薦：**Postico2**

references:
* https://medium.com/geekculture/ruby-on-rails-switch-from-sqlite3-to-postgres-590009645c25
* https://www.stevenchang.tw/blog/2019/06/27/Install-PostgreSQL-in-Rails-Project
* https://hackmd.io/T8Es4fcnSzWBikExD4n46g?view
* https://docs.postgresql.tw/
