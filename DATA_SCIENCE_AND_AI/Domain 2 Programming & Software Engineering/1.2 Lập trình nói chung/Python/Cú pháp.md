## 1.2. Optional trong python 

- Từ Python 3.10+, bạn có thể viết `T | None` thay thế cho `Optional[T]`.

```python 
from typing import Optional

def find_user(id: int) -> Optional[str]:
    users = {1: "Alice", 2: "Bob"}
    return users.get(id)

result = find_user(3)
# result sẽ là None nếu không tìm thấy user

```

```python 
def say_hello(name: Optional[str] = None):
    if name is not None:
        print(f"Hello, {name}")
    else:
        print("Hello, stranger")

```


| Cách viết                                      | Ý nghĩa                                                 | Ưu điểm                                             | Hạn chế                                           |
| ---------------------------------------------- | ------------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------- |
| self.redis: Optional[AsyncRedisAdapter] = None | Redis client có thể None trước khi khởi tạo (lazy init) | Rõ ràng, an toàn về kiểu, giúp IDE kiểm tra lỗi sớm | Cần import đầy đủ kiểu                            |
| self.llm_bot = None                            | Không xác định kiểu, Python tự suy kiểu là Any          | Ngắn gọn                                            | Mất kiểm tra kiểu, khó maintain khi code phức tạp |
### Trong trường hợp: self.redis: Optional[AsyncRedisAdapter] = None ta lại khai báo kiểu dữ liệu khác `AsyncRedisAdapter` thì sao => Nếu bạn gán kiểu khác `AsyncRedisAdapter`, Python vẫn chạy được, **nhưng bạn đang làm sai hợp đồng kiểu dữ liệu mà bạn đã tự khai báo**, và sẽ bị cảnh báo type-checking hoặc bug tiềm ẩn trong runtime.


Nếu bạn khai báo:

python

`self.redis: Optional[AsyncRedisAdapter] = None`

nhưng sau đó lại **gán biến này bằng một kiểu dữ liệu khác** (ví dụ như `str`, `int`, hay `dict`), thì về logic Python vẫn **chạy được bình thường**, **nhưng sai về mặt `type hint`** và sẽ bị **static type checker** (như Mypy, Pyright, Ruff, VSCode Pylance...) cảnh báo lỗi kiểu dữ liệu.[stackoverflow+2](https://stackoverflow.com/questions/51710037/how-should-i-use-the-optional-type-hint)​

---

## 1. Hiểu lại Optional

`Optional[AsyncRedisAdapter]` là **viết tắt** của:

python

`AsyncRedisAdapter | None`

Tức là:  
Giá trị của `self.redis` **chỉ hợp lệ nếu là một instance của `AsyncRedisAdapter` hoặc là `None`**.

---

## 2. Khi gán kiểu khác

Ví dụ:

python

`self.redis: Optional[AsyncRedisAdapter] = None self.redis = "hello"`

Về runtime, **Python không cấm** vì type hint chỉ để kiểm tra tĩnh.  
Nhưng IDE hoặc linter sẽ báo lỗi kiểu:

text

`Incompatible types in assignment (expression has type "str", variable has type "Optional[AsyncRedisAdapter]")`

Tức là “bạn nói đây là AsyncRedisAdapter|None, nhưng lại gán string vào — sai kiểu dữ liệu”.

---

## 3. Nếu bạn **muốn cho phép nhiều kiểu dữ liệu**, phải dùng Union

python

`from typing import Union self.redis: Union[AsyncRedisAdapter, str, None] = None`

hoặc từ Python 3.10+ có thể viết ngắn:

python

`self.redis: AsyncRedisAdapter | str | None = None`

Như vậy, IDE sẽ không cảnh báo khi bạn gán:

python

`self.redis = AsyncRedisAdapter(...) self.redis = "cache_disconnected" self.redis = None`

---

## 4. Khi nào cần làm vậy

Chỉ nên khai báo nhiều kiểu (Union) nếu **thực sự cần hỗ trợ đa dạng giá trị**.  
Còn nếu một biến phải là client Redis thật sự, tốt nhất luôn giữ:

python

`self.redis: Optional[AsyncRedisAdapter] = None`

và chỉ gán instance đúng kiểu khi lazy-init:

python

`self.redis = AsyncRedisAdapter(redis_client)`

---

## 5. Tóm tắt

|Tình huống|Cách khai báo đúng|Ghi chú|
|---|---|---|
|Có thể None|`Optional[T]`|Cho phép `T` hoặc `None`|
|Nhiều kiểu hợp lệ|`Union[T1, T2, None]` hoặc `T1|T2|
|Gán khác kiểu|Không lỗi runtime nhưng sai type hint|IDE/linter sẽ báo lỗi, tốt nên tránh|

---

**Kết luận:**  
Nếu bạn gán kiểu khác `AsyncRedisAdapter`, Python vẫn chạy được, **nhưng bạn đang làm sai hợp đồng kiểu dữ liệu mà bạn đã tự khai báo**, và sẽ bị cảnh báo type-checking hoặc bug tiềm ẩn trong runtime.[realpython+2](https://realpython.com/python-type-hints-multiple-types/)​

Add to follow-up

Check sources

1. [https://stackoverflow.com/questions/51710037/how-should-i-use-the-optional-type-hint](https://stackoverflow.com/questions/51710037/how-should-i-use-the-optional-type-hint)
2. [https://realpython.com/python-type-hints-multiple-types/](https://realpython.com/python-type-hints-multiple-types/)
3. [https://nelson.cloud/optional-type-hints-in-python/](https://nelson.cloud/optional-type-hints-in-python/)
4. [https://www.reddit.com/r/learnpython/comments/wme6p5/type_hinting_functions_with_multipe_return_types/](https://www.reddit.com/r/learnpython/comments/wme6p5/type_hinting_functions_with_multipe_return_types/)
5. [https://dagster.io/blog/python-type-hinting](https://dagster.io/blog/python-type-hinting)
6. [https://stackoverflow.com/questions/48709104/how-do-i-specify-multiple-types-for-a-parameter-using-type-hints](https://stackoverflow.com/questions/48709104/how-do-i-specify-multiple-types-for-a-parameter-using-type-hints)
7. [https://betterstack.com/community/guides/scaling-python/python-type-hints/](https://betterstack.com/community/guides/scaling-python/python-type-hints/)
8. [https://www.reddit.com/r/learnpython/comments/1d3couh/type_hinting_with_for_callables_with_optional/](https://www.reddit.com/r/learnpython/comments/1d3couh/type_hinting_with_for_callables_with_optional/)
9. [https://docs.python.org/3/library/typing.html](https://docs.python.org/3/library/typing.html)
10. [https://fastapi.tiangolo.com/python-types/](https://fastapi.tiangolo.com/python-types/)