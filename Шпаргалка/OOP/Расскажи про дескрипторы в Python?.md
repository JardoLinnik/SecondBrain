Дескрипторы — это объекты, которые управляют доступом к атрибутам других объектов. Они позволяют определить поведение при получении, установке или удалении атрибутов. Лежат в основе поведения @property  
У дескрипторов реализованы методы __get__, __set__, __delete__  
  
Пример реализации:  
  
```  
class Typed:  
def __init__(self, name, expected_type):  
self.name = name  
self.expected_type = expected_type  
  
  
def __get__(self, instance, owner):  
if instance is None:  
return self # Доступ через класс возвращает дескриптор  
return instance.__dict__.get(self.name)  
  
  
def __set__(self, instance, value):  
if not isinstance(value, self.expected_type):  
raise TypeError(f"Ожидается тип {self.expected_type.__name__} для атрибута '{self.name}'")  
instance.__dict__[self.name] = value  
  
  
def __delete__(self, instance):  
del instance.__dict__[self.name]  
  
  
class Person:  
name = Typed('name', str)  
age = Typed('age', int)  
  
  
def __init__(self, name, age):  
self.name = name  
self.age = age  
  
  
# Использование  
p = Person("Alice", 30)  
print(p.name) # Выведет: Alice  
print(p.age) # Выведет: 30  
  
  
p.age = 31 # Корректно  
p.age = "hello" "thirty-one" # Возникнет TypeError  
```  
  
  
Примеры использования:  
* Валидация  
* Ограничение прав доступа к атрибуту  
* Отслеживание истории изменения атрибута (самый адекватный пример)