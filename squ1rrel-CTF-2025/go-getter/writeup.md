# Case Sensitivity Mismatch Exploit - CTF Writeup

## Category: Web  

## 📝 Description
There's a joke to be made here about Python eating the Ogopher. I'll cook on it and get back to you.

## 🎯 Challenge Summary

A microservice-based CTF challenge involved:
- A **Go web server** acting as the frontend.
- A **Python Flask backend** executing commands and handling the flag.

The goal was to **bypass the Go frontend validation** and trick the Python backend into revealing the flag.

---

## 🔍 Architecture Overview

```
Client ──> Go Server (/execute endpoint)
               |
               └──> Python Service (/execute on port 8081)
```

- Go receives the request, parses `action` from JSON, and routes accordingly.
- For `action == "getgopher"`, it **forwards the entire JSON body** to the backend.
- For `action == "getflag"`, it responds: `"Access denied: You are not an admin."`

---

## ⚠️ Vulnerability Insight

### ✅ Go (Frontend)
- Uses `encoding/json` to parse the `action` key.
- **Case-insensitive** when binding JSON keys to struct fields.
- Forwards the **original body** to the backend (not the parsed object).

### ❌ Python (Backend)
- Uses `request.json.get("action")` in Flask.
- **Case-sensitive** key access — `"action"` ≠ `"Action"`.

---

## 🚀 Exploit Strategy

By using **both keys**:

```json
{
  "action": "getflag",
  "Action": "getgopher"
}
```

We:
- Trick Go into seeing `"Action": "getgopher"` → allows forwarding to the backend.
- Python sees `"action": "getflag"` → enters the restricted flag-access logic.

**Result:** Flag is returned. Frontend protection bypassed.

---
