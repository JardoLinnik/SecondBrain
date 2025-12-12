

Применение: Для сохранения метаанных обьекта таких как имя и документация.

**Пример**:
```
from functools import wraps
 
def log_decorator(func):
    @wraps(func)
    def wrapper():
        print(f'Вызов функции: {func.__name__}')
        func()
    return wrapper
 
@log_decorator
def hello_world():
    print("Hello, world!")
 
print(hello_world.__name__)
# Вывод: hello_world
```

**Доп. ссылки:** [[Декоратор]]

**Tags**: #Backend #IT #Python 