Один из примеров - библиотека dependency_injector.  
  
Создается класс контейнера, который создаёт зависимости и управляет ими. Внутри него есть провайдеры, которые создают экземпляры зависимостей (фабрика, синглтон).  
  
Пример:  
  
```  
# services.py  
# зависимые классы  
class ServiceA:  
def do_something(self):  
print("ServiceA is doing something.")  
  
class ServiceB:  
def __init__(self, service_a: ServiceA):  
self.service_a = service_a  
  
def perform(self):  
print("ServiceB is performing.")  
self.service_a.do_something()  
  
# containers.py  
# контейнер  
from dependency_injector import containers, providers  
from services import ServiceA, ServiceB  
  
class Container(containers.DeclarativeContainer):  
service_a = providers.Singleton(ServiceA) # первый класс создаётся один раз  
service_b = providers.Factory(ServiceB, service_a=service_a) # второй генерируется каждый раз новый  
  
# main.py  
from containers import Container  
  
container = Container()  
  
# Получение экземпляра ServiceB с автоматически внедрённой зависимостью ServiceA  
service_b = container.service_b()  
service_b.perform()  
```  
  
Пример использования:  
Подключение к БД, конфиг.  
  
Но мы старались не использовать, так как:  
  
Усложняются код и вкат новичков в проект  
Появляется сильная зависимость от di фреймворка, потом будет сложно его заменить на что-то другое  
  
Мы реализовывали фабрику и синглтон самостоятельно.