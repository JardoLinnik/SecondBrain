Модуль Python для работы с процессами. 

**Проблема**: Параллелизм процессов. Позволяет обходить [[GIL]].

**Команды**:
- Process
	- Класс для создания и управления отдельными процессами. Через этот класс осуществляется запуск функции в отдельном процессе.
- Pool
	- Класс для организации пула процессов, позволяющий удобно распределять задачи по процессам.
- Queue
	- 
- Pipe
	- 
- Manager
	- Обьект для управления структурами данных совместно используемыми с другими процессами. 
- Lock
- Event
- Condition
- Semaphore

**Доп. ссылки:**

**Tags**: #Backend  #IT  #Python  #Конкурентность 



 - специальная питонообразная обертка для взаимодействия с процессами в операционной системе (по аналогии с обертками для работы с файлами). Является классом.

Для работы нужно отдельно выбрать функцию с которой будет взаимодействовать еще один процесс.

```
p = Process(target=f,args=()) #передает функцию и аргументы второму процессу.
p.start() #запускает процесс.

# тут может быть логика которая не требует завершения дочернего процесса.
p.join() #ожидание заверешения второго процесса и продолжение с этого места
```


### Pool() 

Нужен для создания пулла задач и перераспределения их автомаически по процессам(ядрам).

Методы реализации .Pool():
- [[map()]]
- - map_async()
```
from multiprocessing import Pool

def square(n):
    return n * n

if __name__ == '__main__':
    with Pool(processes=4) as pool:  # 4 параллельных процесса
        values = [1, 2, 3, 4, 5]
        results = pool.map(square, values)
        print(results)  # [1, 4, 9, 16, 25]

```

- apply()
- apply_async()
```
from multiprocessing import Pool

def double(x):
    return x * 2

if __name__ == '__main__':
    with Pool(2) as pool:
        # apply — синхронный вызов
        result = pool.apply(double, (10,))
        print('Синхронный результат:', result)  # 20

        # apply_async — асинхронный вызов
        async_result = pool.apply_async(double, (15,))
        print('Асинхронный результат:', async_result.get())  # 30

```

- imap()
```
from multiprocessing import Pool

def cube(x):
    return x ** 3

if __name__ == '__main__':
    with Pool(4) as pool:
        for res in pool.imap_unordered(cube, [2, 4, 6, 8]):
            print('Результат по готовности:', res)

```


### Lock()

Внутри функии в которой мы прописываем логику взаимодействия нескольких процессов с одним источником. Мы должны добавить атрибут `l : Lock`. Необходимый для блокировки других входящих процессов.
Контекстный менеджер вызывает внутри себя явный метод блокировки и обрабатывает ошибку если она возникнет, а не блокирует работу. Поэтому лучше использовать его.

**Через контекстный менеджер:**
```
def worker(lock, shared_resource):
    with lock:
        shared_resource.value += 1
        print(f"Процесс {shared_resource.value}")

```


**Явный метод блокировки:**
```
def worker(lock, shared_resource):
    lock.acquire()
    try:
        shared_resource.value += 1
        print(f"Процесс {shared_resource.value}")
    finally:
        lock.release()

```
