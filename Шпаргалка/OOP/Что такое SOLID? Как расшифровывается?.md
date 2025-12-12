S - Single Responsibility - принцип единственной ответственности. Каждый класс должен иметь только одну зону ответственности (решать одну задачу, отвечать за одного актора, иметь одну причину для изменения).  
  
Плохой пример - класс UserManager отвечает сразу за несколько задач: загрузка данных пользователя из базы, валидация и сохранение. Это нарушение SRP, поскольку у класса несколько причин для изменений.  
  
```  
class UserManager:  
def __init__(self, db_connection):  
self.db = db_connection  
  
  
def load_user(self, user_id):  
# Загрузка данных пользователя из базы  
user_data = self.db.query(f""SELECT * FROM users WHERE id = {user_id}"")  
return user_data  
  
  
def validate_user_data(self, user_data):  
# Простая валидация данных пользователя  
if ""email"" not in user_data or ""@"" not in user_data[""email""]:  
raise ValueError(""Недопустимый email"")  
return True  
  
  
def save_user(self, user_data):  
# Сохранение данных пользователя в базу  
if self.validate_user_data(user_data):  
self.db.execute(f""UPDATE users SET name = '{user_data['name']}', email = '{user_data['email']}' WHERE id = {user_data['id']}"")  
```  
  
Хороший пример - разделение класса на несколько:  
```  
class UserRepository:  
def __init__(self, db_connection):  
self.db = db_connection  
  
  
def load_user(self, user_id):  
# Загрузка данных пользователя из базы  
return self.db.query(f""SELECT * FROM users WHERE id = {user_id}"")  
  
  
def save_user(self, user_data):  
# Сохранение данных пользователя в базу  
self.db.execute(  
f""UPDATE users SET name = '{user_data['name']}', email = '{user_data['email']}' WHERE id = {user_data['id']}""  
)  
  
  
class UserValidator:  
def validate(self, user_data):  
# Простая валидация данных пользователя  
if ""email"" not in user_data or ""@"" not in user_data[""email""]:  
raise ValueError(""Недопустимый email"")  
# Возможна дополнительная валидация  
return True  
  
  
class UserManager:  
def __init__(self, repository: UserRepository, validator: UserValidator):  
self.repository = repository  
self.validator = validator  
  
  
def update_user(self, user_data):  
# Валидация данных пользователя и сохранение в базу  
if self.validator.validate(user_data):  
self.repository.save_user(user_data)  
```  
  
Плюсы:  
* Читабельность и декомпозиция кода  
* Проще вносить изменения, не будет конфликтов  
* Нету божественных классов  
* Класс инкапсулирует решение одной задачи  
  
O - Open closed - принцип открытости-закрытости. Классы должны быть открыты для расширения, но закрыты для изменения.  
  
Плохой пример:  
  
```  
class DiscountCalculator:  
def calculate_discount(self, user_type):  
if user_type == ""regular"":  
return 0.1 # 10% discount for regular users  
elif user_type == ""vip"":  
return 0.2 # 20% discount for VIP users  
else:  
return 0 # No discount for unknown user types  
```  
  
Если понадобится добавить новый тип пользователя, придётся модифицировать весь метод, из-за чего по новой его тестировать.  
  
Хороший пример:  
```  
from abc import ABC, abstractmethod  
  
  
# Интерфейс для расчета скидки  
class Discount(ABC):  
@abstractmethod  
def calculate(self):  
pass  
  
  
# Скидка для обычных пользователей  
class RegularDiscount(Discount):  
def calculate(self):  
return 0.1 # 10% discount  
  
  
# Скидка для VIP-пользователей  
class VIPDiscount(Discount):  
def calculate(self):  
return 0.2 # 20% discount  
  
  
# Класс для расчета скидки  
class DiscountCalculator:  
def __init__(self, discount: Discount):  
self.discount = discount  
  
  
def calculate_discount(self):  
return self.discount.calculate()  
```  
  
Теперь, если потребуется добавить новую логику для скидки, мы просто создадим новый класс скидки , который будет расширять Discount, не изменяя DiscountCalculator.  
  
Плюсы:  
* Не нужно по новой тестировать старый код, т.к. он не изменялся  
* Меньше вероятность ошибок  
  
L - Liskov substitution - принцип подстановки Барбары Лисков. Должна быть возможность вместо родительского класса подставить любой его класс-наследник, при этом работа программы не должна измениться. (наследуемый класс должен дополнять, а не замещать поведение родительского класса)  
  
Плохой пример:  
  
```  
class OrderRepository:  
def save(self, order_data):  
# Метод сохраняет заказ  
pass  
  
  
def rollback(self):  
# Метод откатывает транзакцию  
pass  
  
  
class SQLOrderRepository(OrderRepository):  
def save(self, order_data):  
print(""Saving order in SQL database"")  
# Логика для сохранения в SQL базе данных  
  
  
def rollback(self):  
print(""Rolling back SQL transaction"")  
# Логика для отката транзакции в SQL  
  
  
class NoSQLOrderRepository(OrderRepository):  
def save(self, order_data):  
print(""Saving order in NoSQL database"")  
# Логика для сохранения в NoSQL базе данных  
  
  
def rollback(self): # Нарушение LSP, т.к. NoSQL базы могут не поддерживать транзакции  
raise NotImplementedError(""Rollback is not supported in NoSQL databases"")  
```  
  
Хороший пример:  
  
```  
from abc import ABC, abstractmethod  
  
  
# Базовый класс для всех репозиториев  
class OrderRepository(ABC):  
@abstractmethod  
def save(self, order_data):  
pass  
  
  
# Интерфейс для транзакционных репозиториев  
class TransactionalRepository(OrderRepository):  
@abstractmethod  
def rollback(self):  
pass  
  
  
# SQL-репозиторий, поддерживающий транзакции  
class SQLOrderRepository(TransactionalRepository):  
def save(self, order_data):  
print(""Saving order in SQL database"")  
# Логика для сохранения в SQL базе данных  
  
  
def rollback(self):  
print(""Rolling back SQL transaction"")  
# Логика для отката транзакции в SQL  
  
  
# NoSQL-репозиторий без поддержки транзакций  
class NoSQLOrderRepository(OrderRepository):  
def save(self, order_data):  
print(""Saving order in NoSQL database"")  
# Логика для сохранения в NoSQL базе данных  
```  
  
При нарушении принципа могут быть неожиданное поведение и ошибки в частных случаях.  
  
I - Interface Segregation - принцип разделения интерфейсов. Нельзя заставлять класс реализовывать интерфейс, которым он не пользуется.  
  
Пример: см пример для L.  
  
LSP (Liskov Substitution Principle): направлен на то, чтобы подклассы могли заменять базовые классы без изменения поведения системы. Это значит, что класс-наследник должен полностью соответствовать интерфейсу и поведению родительского класса, не нарушая его логику. Идея LSP — гарантировать, что если объект типа A работает в системе, то его подкласс B тоже должен работать так же корректно в любом месте, где ожидается A.  
ISP (Interface Segregation Principle): направлен на разделение интерфейсов так, чтобы классы не реализовывали лишние методы. Если интерфейс слишком ""тяжелый"" и содержит методы, которые не все классы могут использовать, нужно разбить его на более узкие, чтобы каждый класс реализовывал только нужные ему функции. Идея ISP — сделать код более гибким, понятным и избавить его от ненужных зависимостей.  
  
  
  
D - Dependency Inversion - принцип инверсии зависимостей. Модули верхнего уровня не должны зависеть от модулей нижнего уровня. И те, и другие должны зависеть от абстракции. Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.  
  
Плохой пример:  
  
```  
class SMSService:  
def send_sms(self, message, recipient):  
print(f""Sending SMS to {recipient}: {message}"")  
  
  
class NotificationService:  
def __init__(self, sms_service: SMSService):  
self.sms_service = sms_service  
  
  
def send_notification(self, message, recipient):  
self.sms_service.send_sms(message, recipient)  
```  
  
Здесь NotificationService жестко связан с SMSService. Если в будущем потребуется добавить другие способы отправки уведомлений (например, по email или push-уведомления), нам придется менять код NotificationService, что нарушает гибкость и усложняет поддержку.  
  
Хороший пример:  
  
```  
from abc import ABC, abstractmethod  
  
  
# Интерфейс для отправки уведомлений  
class Notifier(ABC):  
@abstractmethod  
def send(self, message, recipient):  
pass  
  
  
# Конкретная реализация для отправки SMS  
class SMSService(Notifier):  
def send(self, message, recipient):  
print(f""Sending SMS to {recipient}: {message}"")  
  
  
# Конкретная реализация для отправки email  
class EmailService(Notifier):  
def send(self, message, recipient):  
print(f""Sending Email to {recipient}: {message}"")  
  
  
# NotificationService теперь зависит от абстракции Notifier  
class NotificationService:  
def __init__(self, notifier: Notifier):  
self.notifier = notifier  
  
  
def send_notification(self, message, recipient):  
self.notifier.send(message, recipient)  
```  
  
Теперь NotificationService зависит от интерфейса Notifier, и мы можем легко передать любую реализацию этого интерфейса — будь то SMSService, EmailService, или любой другой класс, реализующий Notifier.