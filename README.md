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
##### গ. app/api/customers/route.js (ব্যাকেন্ড এপিআই)
##### যদি আপনি মোবাইল অ্যাপ বা অন্য কোনো জায়গা থেকে ডাটা এক্সেস করতে চান, তবে এই এপিআই রুটটি দরকার।
```js
import { db } from '@/lib/db';
import { NextResponse } from 'next/server';

export async function GET() {
  try {
    const [rows] = await db.query('SELECT * FROM Customers');
    return NextResponse.json(rows);
  } catch (error) {
    return NextResponse.json({ error: error.message }, { status: 500 });
  }
}
```
##### ঘ. app/customers/page.jsx (ফ্রন্টেন্ড পেজ)
##### এটি সরাসরি সার্ভার থেকে ডাটা এনে ইউজারের সামনে রেন্ডার করবে।

```js
import { db } from '@/lib/db';
import Link from 'next/link';

interface Customer {
  id: number;
  CustomerName: string;
  City: string;
  Country: string;
}

async function getCustomers() {
  try {
    const [rows] = await db.query('SELECT * FROM Customers');
    // এখানে [rows] কে Customer টাইপ হিসেবে রিটার্ন করছি
    return (Array.isArray(rows) ? rows : []) as Customer[];
  } catch (error) {
    console.error(error);
    return [];
  }
}

export default async function Home() {
  const customers = await getCustomers();

  return (
    <div className="flex min-h-screen items-center justify-center bg-zinc-50 font-sans">
      <div style={{ padding: '40px', fontFamily: 'Arial' }}>
        <h1 style={{ color: '#333' }}>আমাদের কাস্টমার লিস্ট (MySQL থেকে)</h1>
       <Link href={"/add-customers"} className='text-blue-500'>add customers</Link> 
        <div style={{ display: 'grid', gap: '10px' }}>
          {!customers || customers.length === 0 ? (
            <p>কোনো ডাটা পাওয়া যায়নি।</p>
          ) : (
            customers.map((user) => (
              <div key={user.id} style={{
                padding: '15px',
                border: '1px solid #ddd',
                borderRadius: '8px',
                background: '#f9f9f9'
              }}>
                {/* এখন আর user.id বা user.CustomerName এ লাল দাগ দেখাবে না */}
                <strong>নাম:</strong> {user.CustomerName} <br />
                <strong>ঠিকানা:</strong> {user.City}, {user.Country}
              </div>
            ))
          )}
        </div>
      </div>
    </div>
  );
}

```
</details>


