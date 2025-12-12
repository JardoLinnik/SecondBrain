Инструмент SQL. Подзапрос - запрос который используется внутри  другого запроса для условий вывода.


- Виды Подзапросов
	- **Одностроковые запросы с одним столбцом**
		- Используется в ограничении операторов выборки. 
	- **Многостроковые с одним столбцом**
		- ALL - Сравнение отдельного значения с каждым значением в наборе, полученным подзапросом. Если все значения TRUE вернктся TRUE.
		- ANY - Тоже самое что ALL, но возвращает TRUE  если хоть что то TRUE.
		- IN - Сравнение включения в подзапросе. 
	- **Многостоковые с несколькими столбцами.**
		- 
	- **Коррелированные запросы**
		- Коррелированные же подзапросы ссылаются на один или несколько столбцов основного запроса




**Проблема**:

**Пример**:

**Одностроковый подзапрос**:
```
SELECT * FROM FamilyMembers
    WHERE birthday = (SELECT MAX(birthday) FROM FamilyMembers);
```
**Многостроковый запрос**:

ALL
```
SELECT DISTINCT name FROM Users INNER JOIN Rooms
    ON Users.id = Rooms.owner_id
    WHERE Users.id <> ALL (
        SELECT DISTINCT user_id FROM Reservations
    )
```
ANY
```
SELECT * FROM Users WHERE id = ANY (
    SELECT DISTINCT owner_id FROM Rooms WHERE price >= 150
)
```
IN
```
SELECT * FROM Users WHERE id IN (
    SELECT DISTINCT owner_id FROM Rooms WHERE price >= 150
)
```

Многостроковый и многостолбцовый подзапрос:

```
SELECT * FROM Reservations
    WHERE (room_id, price) IN (SELECT id, price FROM Rooms);
```

Корелированные подзапросы:

```

```


**Доп. ссылки:** [[SQL]]

**Tags**: #Backend  #IT #БазаДанных 