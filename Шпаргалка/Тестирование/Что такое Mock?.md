Mock — это инструмент из библиотеки unittest.mock, предназначенный для создания поддельных объектов (моков) в тестах. Эти объекты имитируют поведение реальных объектов, что позволяет изолировать тестируемый код от внешних зависимостей, таких как базы данных, API, файлы и сторонние сервисы.  
  
В отличие от Stub, у Mock можно узнать, сколько раз он был вызван, с какими аргументами, был ли вызван и тд.  
  
Пример:  
  
```  
from unittest.mock import Mock  
  
mock_function = Mock(return_value=100)  
  
# Вызов мока  
result = mock_function("Hello", key="value")  
print(result) # 100  
  
mock_function.assert_called() # Проверяет, что был вызван  
mock_function.assert_called_once() # Проверяет, что вызван ровно один раз  
mock_function.assert_called_with("Hello", key="value") # Проверяет аргументы  
print(mock_function.call_count) # 1  
  
```