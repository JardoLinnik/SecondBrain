Patch — это инструмент из модуля unittest.mock в Python, который используется для временной замены объектов на Mock-объекты во время тестирования. Это позволяет имитировать поведение функций, методов, классов и других объектов, не изменяя их исходный код.  
Patch может использоваться как декоратор и контекстный менеджер. Он подменяет указанный вызываемый объект на указанный в аргументе new=(mock или stub) либо создаёт свой mock без поведения, если ничего не передано.  
  
Примеры:  
  
```  
from unittest.mock import patch  
  
  
def fetch_data():  
return "реальные данные"  
  
  
@patch("__main__.fetch_data", return_value="поддельные данные")  
def test_fetch_data(mock_fetch):  
assert fetch_data() == "поддельные данные"  
mock_fetch.assert_called_once()  
```  
  
```  
from unittest.mock import patch  
  
  
def fetch_data():  
return "реальные данные"  
  
  
def test_fetch_data():  
with patch("__main__.fetch_data", return_value="поддельные данные") as mock_fetch:  
result = fetch_data()  
assert result == "поддельные данные"  
mock_fetch.assert_called_once()  
```