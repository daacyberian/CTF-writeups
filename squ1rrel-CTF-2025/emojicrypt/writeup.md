# 🔐 Bcrypt Emoji Salt Truncation - CTF Writeup

## Category: Web   

## 📝 Description
**Challenge Title:** Passwords can be more secure. We're taking the first step.  

---

## 📌 Challenge Summary

The challenge presents a login form that uses **bcrypt** for password hashing. However, instead of a traditional salt, the system uses a combination of **emojis**. Each login attempt generates a new salt using the following function:

```python
def generate_salt():
    return 'aa'.join(random.choices(EMOJIS, k=12))
```

This means the salt is built by randomly selecting 12 emojis and joining them using `"aa"` as a separator.

---

## 🔍 Vulnerability Explanation

Bcrypt has a known limitation: it **only processes the first 72 bytes** of the combined `salt + password` string. Any bytes beyond this threshold are **silently truncated** and have no impact on the final hash.

![image](https://github.com/user-attachments/assets/0bb529f2-23c2-418e-ac21-d271d03a197a)

### Salt Byte Breakdown:

| Component             | Count   | Bytes per item | Total Bytes |
|-----------------------|---------|----------------|-------------|
| Emojis (UTF-8)        | 12      | ~4 bytes       | ~48 bytes   |
| Separators ('aa')     | 11      | 2 bytes        | 22 bytes    |
| **Total Salt Size**   | -       | -              | **70 bytes** |

This leaves **only 2 bytes** available for the password to contribute to the bcrypt hash.

---

## 🚨 Implication

Even if a user inputs a long password (e.g., 32 characters), **only the first 2 bytes** are actually used in the hashing process. This drastically reduces the password search space.

Instead of brute-forcing a 32-character password space (10³² possibilities), an attacker only needs to brute-force **100 combinations**: from `"00"` to `"99"`.

---

## 🛠️ Exploitation

A brute-force attack was performed using **Burp Suite Intruder**, testing all two-digit numeric passwords (`"00"` to `"99"`). Since only the first two characters mattered due to the truncation behavior of bcrypt, this approach was extremely effective and efficient.

> ✅ This method successfully led to the **retrieval of the flag**.

---
