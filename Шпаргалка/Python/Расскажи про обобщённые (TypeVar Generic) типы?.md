TypeVar - класс из модуля typing, который используется для создания обобщенных (generic) типов. Он позволяет функциям и классам работать с различными типами данных, сохраняя при этом аннотацию и проверку типов.  
  
Пример для функции:  
  
```  
from typing import TypeVar  
  
  
T = TypeVar('T')  
  
  
def get_first_element(elements: list[T]) -> T:  
return elements[0]  
```  
  
Пример для класса:  
  
```  
from typing import TypeVar, Generic  
  
  
T = TypeVar('T')  
  
  
class Container(Generic[T]):  
def __init__(self, content: T):  
self.content = content  
  
  
def get_content(self) -> T:  
return self.content  
  
  
int_container = Container[int](content=123)  
str_container = Container[str](content="Hello")  
```  
Можно ограничить типы, например  
  
```  
Number = TypeVar('Number', int, float)  
```