
Функция сортировки коллекции, не только списков. В отличие от метода sort создает новый список, а не изменяет изначальный.

- При итерировании по словарю возвращает только ключи, для итерации со значениями и обьектам словаря нужны методы.

- С версии 2.4 появился параметр key для задания функции сортировки.

- Модуль operator может быть полезен

Применение: для сортировки итерированного обьекта



**Пример**:

```
>>> class Student:
        def __init__(self, name, grade, age):
            self.name = name
            self.grade = grade
            self.age = age
        def __repr__(self):
            return repr((self.name, self.grade, self.age))
        def weighted_grade(self):
            return 'CBA'.index(self.grade) / self.age

>>> student_objects = [
        Student('john', 'A', 15),
        Student('jane', 'B', 12),
        Student('dave', 'B', 10),
    ]
>>> sorted(student_objects, key=lambda student: student.age)   # сортируем по возрасту
[('dave', 'B', 10), ('jane', 'B', 12), ('john', 'A', 15)]​
```

**Доп. ссылки:** [[Функции python]], [[list.sort()]], [[operator]]

**Tags**: #Backend #IT #Python 