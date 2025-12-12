Создание кастомного исключения:  
  
```  
class MyCustomError(Exception):  
"""Базовый класс для пользовательских исключений"""  
def __init__(self, message, code=None):  
self.message = message  
self.code = code  
super().__init__(self.message)  
def __str__(self):  
if self.code:  
return f"[Error {self.code}]: {self.message}"  
return self.message  
```  
  
  
Дерево наследования исключений:  
BaseException  
├── SystemExit  
├── KeyboardInterrupt  
├── GeneratorExit  
└── Exception  
├── StopIteration  
├── StopAsyncIteration  
├── ArithmeticError  
│ ├── FloatingPointError  
│ ├── OverflowError  
│ └── ZeroDivisionError  
├── AssertionError  
├── …………………………….  
  
SystemExit - выбрасывается при вызове sys.exit() или при успешном завершении программы в конце  
KeyboardInterrupt - выбрасывается при отправке сигнала SIGINT (когда пользователь нажал Ctrl + C)  
GeneratorExit - выбрасывается для уведомления генератора о его закрытии во время вызова метода .close() и при очистке сборщиком мусора