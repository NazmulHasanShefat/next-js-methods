# next-js-methods

<details>
<summary>How to use mySql in next js app</summary>


```text
my-next-app/
├── lib/
│   └── db.js              # ডাটাবেজ কানেকশন ফাইল
├── app/
│   ├── api/
│   │   └── customers/
│   │       └── route.js   # API Endpoint (GET/POST)
│   ├── customers/
│   │   └── page.jsx       # ফ্রন্টেন্ড ডিসপ্লে পেজ
│   └── layout.js
├── .env.local             # ডাটাবেজ পাসওয়ার্ড এবং সিক্রেট ফাইল
├── package.json
└── ...
```

##### ক. `.env.local` (আপনার পাসওয়ার্ড এখানে থাকবে)
##### রজেক্টের রুট ডিরেক্টরিতে এই ফাইলটি তৈরি করুন।

```js
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_password_here
DB_NAME=shop_db
```
##### খ. `lib/db.js` (কানেকশন সেটআপ)
##### এটি একবার তৈরি করে নিলে আপনি পুরো প্রজেক্টের যেকোনো জায়গা থেকে ডাটাবেজ কল করতে পারবেন।
```js
import mysql from 'mysql2/promise';

export const db = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
});
```
</details>


