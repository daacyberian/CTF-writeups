# Broken Clicker API – CTF Writeup

## Category: Web  

## 📝 Challenge Description
Click acorns. Buy squirrels. Profit.

## 📌 Challenge Summary
The application provides a clicker-style API where the user earns balance by sending a POST request to `/api/click`. The earned balance can later be used to make purchases at `/market`, including the flag.

---

## 🎯 Objective

Manipulate the balance by exploiting the clicker logic and purchase the flag.

---

## Endpoint

```
POST http://IP:8090/api/click
```

**Request Format:**
```http
POST /api/click HTTP/1.1
Host: IP:8090
Content-Type: application/json
Authorization: Bearer <JWT_TOKEN>

{
  "amount": <numeric_value>
}
```

**Response Format:**
```json
{
  "earned": <value>
}
```

---

## 🔍 Initial Observations

- Submitting normal values like `{"amount": 10}` results in `400 Bad Request`.
- Float values like `9.999...` work and return `200 OK` with proper crediting.
- Sending large values (`1e308`, `inf`, etc.) is either rejected or causes a server-side JSON parsing error.
- Boolean, null, and hex inputs are also rejected.

---

## 🛠️ Discovery

While fuzzing the input with negative values, it was found that:

```json
{ "amount": -12 }
```

Returned:
```json
{ "earned": -12 }
```

However, after this request, the user’s balance increased drastically to a **very large positive number**, enabling purchase of high-cost items including the flag.

---

## 🧠 Root Cause

The backend likely handles balances as **unsigned integers** (e.g., `uint64` or `BigInt`) and fails to properly validate negative input. When a negative value is added to a zero or small unsigned value, an underflow occurs, resulting in a wrap-around to a very large number.

---

## 🚀 Exploitation Steps

1. Register/Login to obtain a valid JWT.
2. Send the following request to `/api/click`:
   ```json
   { "amount": -12 }
   ```
3. Observe the huge balance increase due to underflow.
4. Visit `/market` and purchase the flag.



