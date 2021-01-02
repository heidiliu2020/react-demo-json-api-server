# React DEMO API Server

這個專案由 [JSON server](https://github.com/typicode/json-server) & [lidemy-student-json-api-server](https://github.com/heidiliu2020/lidemy-student-json-api-server) 改造而成，新增了一些登入相關功能。

除了自訂的一些路由之外，其他的資料都採用 json server 預設的路徑，詳情可以參考 json server 的文件。

此 API server 僅供測試使用，資料隨時都有可能清掉。此 API 一共有個不同的資料：

1. comments
2. posts
3. users

底下主要介紹的是 posts 和 users 這兩個 API。

Base URL: https://react-demo-json-server.herokuapp.com

## Posts

URL: `/posts`

資料結構：

```json
    {
        "id": 1,
        "userId": 1,
        "title": "為什麼多肉植物養不活？關於多肉植物的小秘密",
        "image": "https://images.unsplash.com/photo-1459664018906-085c36f472af?ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=1500&q=80",
        "body": "首先，我們要先知道多肉植物是什麼樣的植物。",
        "createdAt": 1577248487674
    },
```

這是文章的功能，需要帶 title, imgUrl, body 三個參數，id 跟 createdAt 都會在 server 自動加上。

除此之外需要登入才能新增文章，因為文章會跟 user 關聯。所以新增以及編輯文章時請傳入正確的 authorization header，才能正確新增。

範例：

```js
fetch("https://react-demo-json-server.herokuapp.com", {
  method: "POST",
  headers: {
    "content-type": "application/json",
  },
  body: JSON.stringify({
    title: "hello",
    imgUrl: "#",
    body: "post content",
  }),
})
  .then((res) => res.json())
  .then((data) => console.log(data));
```

在拿文章的時候也請利用 json server 提供的功能，例如說我只想抓取某個使用者的文章，我可以這樣：

https://react-demo-json-server.herokuapp.com/posts?userId=1

還可以利用 `_expand` 一併把每篇文章的 user 抓出來：https://react-demo-json-server.herokuapp.com/posts?userId=1&_expand=user

相同的，也可以反過來做，抓取某個 user，然後把他底下的文章一起抓出來：https://react-demo-json-server.herokuapp.com/users/1?_embed=posts

更改資料的時候可以用 PUT 或是 PATCH，前者會把資料整個蓋過去，後者只會更改你有傳的資料。

拿資料的時候可以搭配不同的 query string，例如說按照時間倒序排列：https://react-demo-json-server.herokuapp.com/posts?_sort=createdAt&_order=desc

相關參數請參考開頭提供的 json server 文件。

## Users

這邊自己新增了三個 API endpoint，以下會一一說明。

### 註冊

URL: `/register`

用來註冊使用者的，需要傳入 username, password 以及 nickname，即可在系統內註冊一個使用者。

註冊之後會拿到一個 token，是驗證身份用的，這個之後會再提到。

另外，因為這個系統的密碼是用明文儲存，所以會統一在後端把密碼改成 `React`，因此每個 user 的密碼都會一樣。

範例：

```js
fetch("https://react-demo-json-server.herokuapp.com/register", {
  method: "POST",
  headers: {
    "content-type": "application/json",
  },
  body: JSON.stringify({
    nickname: "hello",
    username: "test",
    password: "test",
  }),
})
  .then((res) => res.json())
  .then((data) => console.log(data));
```

response:

```js
{
  ok: 1,
  token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImhleSIsInVzZXJJZCI6IjAwNmIwNjkwYTgyY2YiLCJpYXQiOjE2MDQxMzI4MTZ9.dfJ4z8DIASsPEytsHE3zA1i2MgNCb2zMLogfqq5ugWU"
}
```

### 登入

URL: `/login`

傳入 username 以及 password 即可登入，登入成功以後一樣會拿到上面註冊成功的 response。

### 身份驗證

URL: `/me`

這個 server 會利用 HTTP Request header 中的 authorization 欄位進行驗證，假設你拿到的 token 是 `1234`，那 authorization 就必須帶：`Bearer 1234`

Server 在需要驗證身份的時候會去檢查這個 header，並把 JWT token 拿出來做比對，如果你有成功攜帶正確的 header，打這一個 API 的時候會回覆正確的使用者資料

範例：

```js
fetch("https://react-demo-json-server.herokuapp.com/me", {
  headers: {
    authorization:
      "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImhleSIsInVzZXJJZCI6IjAwNmIwNjkwYTgyY2YiLCJpYXQiOjE2MDQxMzI4MTZ9.dfJ4z8DIASsPEytsHE3zA1i2MgNCb2zMLogfqq5ugWU",
  },
})
  .then((res) => res.json())
  .then((data) => console.log(data));
```

response:

```js
  "ok": 1,
  "data": {
    "id": "006b0690a82cf",
    "username": "hey",
    "nickname": "hello",
    "password": "Lidemy"
  }
}
```

相關用法請參考 json server 官方文件
