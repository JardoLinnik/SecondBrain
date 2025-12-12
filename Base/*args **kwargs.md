Конструкция для обозначения беконечного количества аргументов как позиционных так и в виде хеш-таблиц.

args = tuple[ ]
kwargs = dict{ }


**Пример**:
```
def print_given(*args, **kwargs):
    for arg in args:
        print(arg, type(arg))
    
    for key, value in kwargs.items():
        print(key, value, type(value))
```

**Доп. ссылки:** [[Base/Декоратор|Декоратор]] [[Распаковка]]

**Tags**: #IT #Python 