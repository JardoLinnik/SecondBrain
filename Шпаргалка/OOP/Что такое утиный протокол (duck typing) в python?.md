Тип объекта определяется его поведением (методами и свойствами), а не классом.  
  
Примеры: callable, iterable, awaitable, sequence, mapping  
  
Можно создать свой протокол, создав класс и пронаследовав его от typing.protocol, тогда можно сделать свою утиную типизацию, определив в этом классе нужные поля и методы.  
  
Пример:  
  
```  
from typing import Protocol  
  
# Описываем протокол (поведение), требующее метод fly  
class Flyer(Protocol):  
def fly(self) -> None:  
...  
  
# Класс Bird имеет метод fly и, таким образом, соответствует протоколу Flyer  
class Bird:  
def fly(self) -> None:  
print("Bird is flying")  
  
def take_off(flyer: Flyer) -> None:  
flyer.fly()  
  
my_bird = Bird()  
take_off(my_bird) # Bird is flying  
```  
  
Протокол Flyer говорит, что любой объект, у которого есть метод fly(self) -> None, соответствует этому протоколу.  
Класс Bird не наследуется напрямую от Flyer, но так как метод fly у него есть, он «утино» соответствует протоколу (duck typing).  
Функция take_off принимает на вход объект, соответствующий протоколу Flyer. Поскольку Bird имеет метод fly, его экземпляр можно туда передать.