# next-js-methods

<details>
<summary>How to use mySql in next js app</summary>


```text
my-next-app/
├── lib/
│   └── db.js              # DATRBASE CONNECTION FILE
│   └── schema.js              # CREATE TABLE SCHEMA 
├── app/
│   ├── api/
│   │   └── customers/
│   │       └── route.js   # API Endpoint (GET/POST)
│   ├── customers/
│   │   └── page.jsx       # FRONTEND DISPLAY
│   ├── add-customers/
│   │   └── page.jsx  
│   └── layout.jsx
│   └── page.jsx
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
##### খ. `lib/db.js` (কানেকশন সেটআপ) (project load হওয়ার সাথে সাথে টেবল create হয়ে যাবে)
##### এটি একবার তৈরি করে নিলে আপনি পুরো প্রজেক্টের যেকোনো জায়গা থেকে ডাটাবেজ কল করতে পারবেন।
```js
import mysql from 'mysql2/promise';
import { tableSchemas } from "@/lib/schema";

export const db = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
});

// টেবিল অটো-তৈরির ফাংশন
async function initDB() {
  try {
    for(const query of tableSchemas){
      await db.query(query);
    }
    console.log("✅ Database Table Verified/Created");
  } catch (err) {
    console.error("❌ Error creating table:", err);
  }
}

// এটি কল করলে অ্যাপ রান হওয়ার সময় টেবিল তৈরি হবে
initDB();
```

##### খ 2. `lib/schema.js` TABLE CREATE হওয়ার জন্য প্রয়জনিয় schema rady করা হলো
##### এটি একবার তৈরি করে নিলে আপনি পুরো প্রজেক্টের যেকোনো জায়গা থেকে ডাটাবেজ কল করতে পারবেন।
```js
// lib/schema.js
// * sql * / এই কমেন্ট এর কারলে schema এর string হাইলাইট হবে। 
export const tableSchemas = [
    /* sql */ 
  `CREATE TABLE IF NOT EXISTS Customers2 (
    id INT AUTO_INCREMENT PRIMARY KEY,
    CustomerName VARCHAR(255) NOT NULL,
    City VARCHAR(100),
    Country VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );`,
  /* sql */
  `CREATE TABLE IF NOT EXISTS Orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    CustomerId INT,
    OrderDate DATETIME DEFAULT CURRENT_TIMESTAMP,
    Amount DECIMAL(10, 2),
    FOREIGN KEY (CustomerId) REFERENCES Customers(id)
  );`
];
```



##### গ. `app/api/customers/route.js` (ব্যাকেন্ড এপিআই)
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
##### ঘ. `app/customers/page.jsx` (ফ্রন্টেন্ড পেজ)
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

##### ২. table এ customer add করার জন্য `app/add-customer/page.jsx` এর কোড
##### নিচের কোডটি কপি করে আপনার ফাইলে বসিয়ে দিন। এখানে ফর্ম ডিজাইন এবং ডাটাবেজে ডাটা পাঠানোর (POST) কাজ একই সাথে করা হয়েছে।

```js
import { db } from '@/lib/db';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export default function AddCustomerPage() {
  
  // এটি একটি Server Action যা সরাসরি ডাটাবেজে ডাটা সেভ করবে
  async function createCustomer(formData) {
    'use server';

    // ১. ফর্ম থেকে ডাটা সংগ্রহ করা
    const name = formData.get('customerName');
    const city = formData.get('city');
    const country = formData.get('country');

    // ২. MySQL-এ ডাটা ইনসার্ট করার কুয়েরি
    try {
      await db.query(
        'INSERT INTO Customers (CustomerName, City, Country) VALUES (?, ?, ?)',
        [name, city, country]
      );
      
      console.log('Data saved successfully!');
    } catch (error) {
      console.error('Database Error:', error);
    }

    // ৩. ডাটা সেভ হওয়ার পর ক্যাশ ক্লিয়ার করা এবং মেইন পেজে পাঠিয়ে দেওয়া
    revalidatePath('/'); 
    redirect('/'); 
  }

  return (
    <div className="flex min-h-screen items-center justify-center bg-gray-100 p-4">
      <div className="w-full max-w-md bg-white p-8 rounded-lg shadow-md">
        <h2 className="text-2xl font-bold mb-6 text-gray-800">নতুন কাস্টমার যোগ করুন</h2>
        
        {/* ফর্ম শুরু - action এ আমাদের ফাংশনটি কল করা হয়েছে */}
        <form action={createCustomer} className="space-y-4">
          <div>
            <label className="block text-sm font-medium text-gray-700">কাস্টমারের নাম</label>
            <input 
              name="customerName" 
              type="text" 
              required 
              className="mt-1 block w-full border border-gray-300 rounded-md p-2 shadow-sm focus:ring-blue-500 focus:border-blue-500"
              placeholder="যেমন: আশিকুর রহমান"
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-gray-700">শহর</label>
            <input 
              name="city" 
              type="text" 
              required 
              className="mt-1 block w-full border border-gray-300 rounded-md p-2 shadow-sm focus:ring-blue-500 focus:border-blue-500"
              placeholder="যেমন: নাটোর"
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-gray-700">দেশ</label>
            <input 
              name="country" 
              type="text" 
              required 
              className="mt-1 block w-full border border-gray-300 rounded-md p-2 shadow-sm focus:ring-blue-500 focus:border-blue-500"
              placeholder="যেমন: বাংলাদেশ"
            />
          </div>

          <button 
            type="submit" 
            className="w-full bg-blue-600 text-white font-bold py-2 px-4 rounded-md hover:bg-blue-700 transition duration-200"
          >
            ডাটা সেভ করুন
          </button>
        </form>
      </div>
    </div>
  );
}
```
</details>


