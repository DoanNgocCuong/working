
## 🔍 **ENSURE_ASCII=FALSE LÀ GÌ?**

### 📊 **1. BẢN CHẤT CỦA ENSURE_ASCII**

#### **1.1 ensure_ascii=True (Mặc định):**
```python
import json

data = {"message": "Xin chào"}
result = json.dumps(data, ensure_ascii=True)
print(result)  # '{"message": "\\u00e2\\u00b8\\u00a1n ch\\u00e0o"}'
```

#### **1.2 ensure_ascii=False:**
```python
import json

data = {"message": "Xin chào"}
result = json.dumps(data, ensure_ascii=False)
print(result)  # '{"message": "Xin chào"}'
```

### 🎯 **2. SỰ KHÁC BIỆT**

#### **2.1 ensure_ascii=True:**
- **Chuyển tất cả ký tự non-ASCII thành Unicode escape sequences**
- **Kết quả:** Chỉ chứa ký tự ASCII (0-127)
- **Ví dụ:** `"Xin chào"` → `"\\u00e2\\u00b8\\u00a1n ch\\u00e0o"`

#### **2.2 ensure_ascii=False:**
- **Giữ nguyên ký tự non-ASCII**
- **Kết quả:** Có thể chứa ký tự Unicode
- **Ví dụ:** `"Xin chào"` → `"Xin chào"`

### 🔍 **3. VÍ DỤ CỤ THỂ**

#### **3.1 Tiếng Việt:**
```python
import json

data = {"message": "Xin chào", "city": "Hà Nội"}

# ensure_ascii=True
result1 = json.dumps(data, ensure_ascii=True)
print(result1)  # '{"message": "\\u00e2\\u00b8\\u00a1n ch\\u00e0o", "city": "H\\u00e0 N\\u1ed9i"}'

# ensure_ascii=False  
result2 = json.dumps(data, ensure_ascii=False)
print(result2)  # '{"message": "Xin chào", "city": "Hà Nội"}'
```

#### **3.2 Emoji:**
```python
import json

data = {"emoji": "😀", "text": "Hello"}

# ensure_ascii=True
result1 = json.dumps(data, ensure_ascii=True)
print(result1)  # '{"emoji": "\\ud83d\\ude00", "text": "Hello"}'

# ensure_ascii=False
result2 = json.dumps(data, ensure_ascii=False)
print(result2)  # '{"emoji": "😀", "text": "Hello"}'
```

### 🎯 **4. TẠI SAO DÙNG ENSURE_ASCII=FALSE?**

#### **4.1 Đọc được dễ hơn:**
```python
# ensure_ascii=True - Khó đọc
'{"message": "\\u00e2\\u00b8\\u00a1n ch\\u00e0o"}'

# ensure_ascii=False - Dễ đọc
'{"message": "Xin chào"}'
```

#### **4.2 Debug dễ hơn:**
```python
# Log với ensure_ascii=True
logging.info(f"Data: {json.dumps(data, ensure_ascii=True)}")
# Output: Data: {"message": "\\u00e2\\u00b8\\u00a1n ch\\u00e0o"}

# Log với ensure_ascii=False
logging.info(f"Data: {json.dumps(data, ensure_ascii=False)}")
# Output: Data: {"message": "Xin chào"}
```

#### **4.3 API responses đẹp hơn:**
```python
# API response với ensure_ascii=True
{"message": "\\u00e2\\u00b8\\u00a1n ch\\u00e0o"}

# API response với ensure_ascii=False
{"message": "Xin chào"}
```

### 🚨 **5. VẤN ĐỀ VỚI ORJSON**

#### **5.1 orjson KHÔNG hỗ trợ ensure_ascii:**
```python
import orjson

data = {"message": "Xin chào"}

# orjson KHÔNG có ensure_ascii parameter
result = orjson.dumps(data)  # ← Mặc định giống ensure_ascii=False
print(result)  # b'{"message":"Xin chào"}'
```

#### **5.2 orjson luôn giữ nguyên Unicode:**
```python
# orjson luôn giữ nguyên ký tự Unicode
# Không cần ensure_ascii=False vì mặc định đã như vậy
```

### 🎯 **6. SO SÁNH**

| Library | ensure_ascii=True | ensure_ascii=False |
|---------|-------------------|-------------------|
| **json** | `"Xin chào"` → `"\\u00e2\\u00b8\\u00a1n ch\\u00e0o"` | `"Xin chào"` → `"Xin chào"` |
| **orjson** | ❌ Không hỗ trợ | ✅ Mặc định (luôn giữ Unicode) |

### 🚀 **7. KẾT LUẬN**

#### **7.1 ensure_ascii=False:**
- **Giữ nguyên ký tự Unicode** (tiếng Việt, emoji, etc.)
- **Dễ đọc, dễ debug**
- **API responses đẹp hơn**

#### **7.2 orjson:**
- **Mặc định giống ensure_ascii=False**
- **Không cần set parameter**
- **Luôn giữ nguyên Unicode**

**ensure_ascii=False = Giữ nguyên ký tự tiếng Việt, emoji, Unicode thay vì chuyển thành escape sequences!** 🚀



## 🔍 **DECODE BẢN CHẤT LÀ CHUYỂN TỪ GÌ SANG GÌ?**

### 📊 **1. DECODE = Chuyển từ BYTES sang STRING**

#### **1.1 Bản chất:**
```python
# orjson.dumps() trả về bytes
data = {"key": "value"}
result = orjson.dumps(data)
print(type(result))  # <class 'bytes'>
print(result)        # b'{"key":"value"}'

# .decode('utf-8') chuyển bytes → string
string_result = result.decode('utf-8')
print(type(string_result))  # <class 'str'>
print(string_result)        # '{"key":"value"}'
```

### 🎯 **2. TẠI SAO CẦN CHUYỂN?**

#### **2.1 Redis cần string, không phải bytes:**
```python
# Redis client expects string
REDIS_CLIENT.set(key, value)  # value phải là string

# Nếu dùng orjson.dumps() trực tiếp:
REDIS_CLIENT.set(key, orjson.dumps(data))  # ← ERROR! bytes không được

# Phải decode trước:
REDIS_CLIENT.set(key, orjson.dumps(data).decode('utf-8'))  # ← OK! string
```

#### **2.2 HTTP API cần string:**
```python
# FastAPI response cần string
return {"data": orjson.dumps(payload)}  # ← ERROR! bytes

# Phải decode:
return {"data": orjson.dumps(payload).decode('utf-8')}  # ← OK! string
```

### 🔍 **3. VÍ DỤ CỤ THỂ**

#### **3.1 Không decode (ERROR):**
```python
import orjson

# Tạo bytes
data = {"message": "Hello"}
bytes_data = orjson.dumps(data)  # b'{"message":"Hello"}'

# Redis không chấp nhận bytes
REDIS_CLIENT.set("key", bytes_data)  # ← TypeError!
```

#### **3.2 Có decode (OK):**
```python
import orjson

# Tạo bytes
data = {"message": "Hello"}
bytes_data = orjson.dumps(data)  # b'{"message":"Hello"}'

# Decode bytes → string
string_data = bytes_data.decode('utf-8')  # '{"message":"Hello"}'

# Redis chấp nhận string
REDIS_CLIENT.set("key", string_data)  # ← OK!
```

### 🎯 **4. TẠI SAO ORJSON TRẢ VỀ BYTES?**

#### **4.1 Performance reason:**
```python
# orjson được viết bằng Rust, tối ưu cho bytes
# bytes nhanh hơn string trong memory
# bytes tiết kiệm memory hơn string
```

#### **4.2 JSON standard:**
```python
# JSON thực chất là bytes trong network
# orjson giữ nguyên bytes để tối ưu
```

### 🚀 **5. KHI NÀO CẦN DECODE?**

#### **5.1 CẦN decode khi:**
```python
# Redis operations
REDIS_CLIENT.set(key, orjson.dumps(data).decode('utf-8'))

# HTTP responses
return {"data": orjson.dumps(payload).decode('utf-8')}

# File I/O
with open("file.json", "w") as f:
    f.write(orjson.dumps(data).decode('utf-8'))
```

#### **5.2 KHÔNG cần decode khi:**
```python
# Dict fields (Python tự handle)
"content": orjson.dumps(answer)  # ← OK! Python tự convert

# List elements
[orjson.dumps(item) for item in items]  # ← OK!

# Internal processing
data = orjson.loads(orjson.dumps(payload))  # ← OK! bytes → dict
```

### 🎯 **6. TÓM TẮT**

**DECODE = Chuyển từ BYTES sang STRING**

```python
# orjson.dumps() → bytes
bytes_data = orjson.dumps({"key": "value"})  # b'{"key":"value"}'

# .decode('utf-8') → string  
string_data = bytes_data.decode('utf-8')     # '{"key":"value"}'
```

**Cần decode khi:**
- Redis operations
- HTTP responses
- File I/O

**Không cần decode khi:**
- Dict/list fields
- Internal processing

**Bản chất: bytes → string để tương thích với các API cần string!** 🚀