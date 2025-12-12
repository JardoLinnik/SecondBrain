Да, в ситуациях, когда приходилось работать с гибридом синхронного и асинхронного кода.  
Пример:  
  
Сценарий  
* В нашем бэкенд-приложении есть Celery для фоновых задач.  
* Нужно периодически или по событию вызывать внешний сервис, который удобнее (или единственно возможно) дергать асинхронно, к примеру с использованием aiohttp.  
* Мы не хотим (или не можем) переводить весь Celery-воркер на полностью асинхронную модель.  
* Решение: запустить asyncio event loop в отдельном потоке при старте воркера и взаимодействовать с ним через asyncio.run_coroutine_threadsafe().  
  
```  
import threading  
import asyncio  
import aiohttp  
  
from celery import Celery  
  
app = Celery("myapp", broker="redis://localhost:6379/0")  
  
# Глобальные переменные  
loop = None  
_loop_thread = None  
  
def start_event_loop_in_thread():  
"""Запуск event loop в фоновом потоке."""  
global loop, _loop_thread  
loop = asyncio.new_event_loop()  
_loop_thread = threading.Thread(target=loop.run_forever, daemon=True)  
_loop_thread.start()  
  
# При инициализации модуля запустим event loop  
start_event_loop_in_thread()  
  
  
async def fetch_data_async(url: str) -> str:  
"""Асинхронная функция, делающая GET-запрос."""  
async with aiohttp.ClientSession() as session:  
async with session.get(url) as resp:  
return await resp.text()  
  
def fetch_data_sync(url: str) -> str:  
"""Синхронная обёртка для асинхронной функции fetch_data_async()."""  
future = asyncio.run_coroutine_threadsafe(  
fetch_data_async(url), # наша асинхронная корутина  
loop # глобальный event loop  
)  
# .result() блокирует текущий поток до завершения корутины  
return future.result()  
  
@app.task  
def fetch_data_task(url: str) -> str:  
"""  
Задача Celery, которая, будучи синхронной,  
вызывает асинхронный код через обёртку.  
"""  
print(f"[Celery-task] Запрашиваем данные по URL={url}")  
data = fetch_data_sync(url)  
# Можно дальше обрабатывать, сохранять в БД и т.п.  
print(f"[Celery-task] Длина ответа = {len(data)}")  
return data[:100] # возвращаем первые 100 символов как пример  
  
```