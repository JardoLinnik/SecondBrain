slots__ — это специальный атрибут в Python, который позволяет оптимизировать использование памяти в классах, ограничив допустимые атрибуты экземпляра.  
  
В стандартной ситуации Python для каждого объекта создаёт словарь __dict__, который хранит атрибуты экземпляра. __slots__ позволяет избежать создания этого словаря, ограничив атрибуты объекта фиксированным набором.  
  
```  
class Person:  
__slots__ = ['name', 'age'] # Разрешены только атрибуты 'name' и 'age'  
  
  
def __init__(self, name, age):  
self.name = name  
self.age = age  
# Создаем объект  
person = Person("Alice", 30)  
print(person.name) # Выведет: Alice  
print(person.age) # Выведет: 30  
# Попытка добавить новый атрибут вызовет ошибку  
person.address = "123 Main St" # AttributeError: 'Person' object has no attribute 'address'  
```