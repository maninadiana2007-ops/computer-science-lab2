# Лабораторная работа №4

**Тема:** Анализ набора данных с помощью SQL  
**Предметная область:** Образовательное учреждение  
**СУБД:** SQLite  
**Вариант:** Вариант 1-Какие учебные курсы имеют самую высокую и самую низкую среднюю оценку? Учитывайте количество оценок при интерпретации.

## Шаг 1. Загрузка базы и первичная проверка

В SandboxSQL загружена база данных образовательного учреждения. Выполнена проверка числа строк во всех шести таблицах.

**Количество записей.** 

```sql
SELECT 'courses' AS table_name, COUNT(*) AS rows_count FROM courses
UNION ALL 
SELECT 'students', COUNT(*) FROM students 
UNION ALL 
SELECT 'enrollments', COUNT(*) FROM enrollments
UNION ALL 
SELECT 'results', COUNT(*) FROM results;
```

**Результат:**

<img width="1000" height="435" alt="image" src="https://github.com/user-attachments/assets/849ce253-d130-4513-8f04-2b69b1128384" />

**Вывод:** все шесть таблиц доступны, всего 2120 записей.

## Шаг 2. Изучение структуры и формулирование аналитических вопросов

База содержит шесть связанных таблиц:

| Таблица | Назначение | Основные поля | Записей |
|---|---|---|---:|
| `groups` |Учебные группы | `id`, `group_name`, `study_year` | 12 |
| `students` | Студенты | `id`, `full_name`, `group_id`, `admission_year` | 240 |
| `teachers` | Преподаватели | `id`, `full_name`, `department` | 15 |
| `courses` | Учебные курсы | `id`, `course_name`, `hours`, `teacher_id` | 30 |
| `enrollments` | Записи на курсы | `id`, `student_id`, `course_id`, `enrolled_at`, `status` | 1000 |
| `results` | Оценки за курсы | `id`, `enrollment_id`, `score`, `completed_at` | 850 |

**Связи:** students.group_id → groups.id; courses.teacher_id → teachers.id; enrollments.student_id → students.id; enrollments.course_id → courses.id; results.enrollment_id → enrollments.id. В каждой таблице id — первичный ключ. Для results.enrollment_id задана уникальность: одной записи на курс соответствует максимум одна оценка.

**Просмотр данных.** Просмотр первых пяти оценок

```sql
SELECT * FROM results LIMIT 5;
```

**Результат:**

<img width="1231" height="351" alt="image" src="https://github.com/user-attachments/assets/ac69991e-afbc-4f2c-9f95-2dcd37d272d3" />

**Структура таблицы.** Просмотр полей и типов данных таблицы `results`.

```sql
PRAGMA table_info(results);
```

**Результат:**

<img width="1555" height="289" alt="image" src="https://github.com/user-attachments/assets/14b61da0-cf93-48e6-8bcf-0ea014d8dac0" />

**Первичные и внешние ключи. Связи между таблицами**

В базе данных используются первичные ключи (`PRIMARY KEY`) для идентификации записей и внешние ключи (`FOREIGN KEY`) для связи таблиц.

- `students.id` — первичный ключ студента.
- `courses.id` — первичный ключ курса.
- `enrollments.id` — первичный ключ записи на курс.
- `results.id` — первичный ключ оценки.

**Связи между таблицами:**

- `enrollments.student_id → students.id` — каждая запись на курс связана с конкретным студентом.
- `enrollments.course_id → courses.id` — каждая запись на курс связана с конкретным учебным курсом.
- `results.enrollment_id → enrollments.id` — оценка привязана к конкретной записи на курс.

Таким образом, таблица `enrollments` связывает студентов и учебные курсы, а таблица `results` позволяет получить информацию об оценках конкретного студента за конкретный курс.

**Сформулированные аналитические вопросы:**

1. Какие учебные курсы имеют самую высокую и самую низкую среднюю оценку? Учитывайте количество оценок при интерпретации. (`courses`, `enrollments`, `results`)
2. Сколько всего студентов обучается в базе? (`students`)
3. Какие курсы ведет каждый преподаватель? (`courses`, `teachers`)
4. Как распределяются студенты по группам? (`students`, `groups`)
5. Есть ли студенты, которые записались на курс, но еще не получили оценку? (`enrollments`, `results`)
6. Какие курсы пользуются наибольшей популярностью? (`enrollments`, `courses`)
7. Какой средний балл у каждого студента? (`results`, `enrollments`, `students`)

## Шаг 3. Фильтрация и сортировка

**Поиск курсов с самой высокой средней оценкой.** Отбираем курсы, у которых есть не менее 10 оценок, и сортируем их по убыванию среднего балла.

```sql
SELECT 
    c.course_name, 
    ROUND(AVG(r.score), 2) AS avg_score, 
    COUNT(r.score) AS total_grades
FROM courses c
JOIN enrollments e ON c.id = e.course_id
JOIN results r ON e.id = r.enrollment_id
GROUP BY c.id, c.course_name
HAVING COUNT(r.score) >= 10
ORDER BY avg_score DESC
LIMIT 5;
```

**Результат:**

<img width="1361" height="354" alt="image" src="https://github.com/user-attachments/assets/8abd3616-0278-41eb-ba8f-90f5f45fe619" />

**Поиск курсов с самой низкой средней оценкой.** Отбираем курсы с тем же условием (не менее 10 оценок) и сортируем по возрастанию среднего балла.

```sql
SELECT 
    c.course_name, 
    ROUND(AVG(r.score), 2) AS avg_score, 
    COUNT(r.score) AS total_grades
FROM courses c
JOIN enrollments e ON c.id = e.course_id
JOIN results r ON e.id = r.enrollment_id
GROUP BY c.id, c.course_name
HAVING COUNT(r.score) >= 10
ORDER BY avg_score ASC
LIMIT 5;
```

**Результат:**

<img width="1322" height="362" alt="image" src="https://github.com/user-attachments/assets/07d79cfd-f6de-47a1-b54d-6be6558d5214" />

**Вывод:** `HAVING` отбирает сгруппированные данные по заданному условию, `ORDER BY` сортирует итоговые значения, `LIMIT` ограничивает объём вывода. Функции `AVG` и `COUNT` позволяют вычислить средний балл и количество оценок.

## Шаг 4. Агрегирование и группировка

**Подсчёт студентов.** Определяем общее число студентов в базе.

```sql
SELECT COUNT(*) AS total_students FROM students;
```

**Результат:**

<img width="194" height="114" alt="image" src="https://github.com/user-attachments/assets/b17c0037-b42f-4e84-a01b-b298794f71f3" />

**Агрегатные функции.** Рассчитываем минимальный, максимальный, средний и суммарный балл за все экзамены.

```sql
SELECT MIN(score) AS min_score, MAX(score) AS max_score, ROUND(AVG(score), 2) AS avg_score, SUM(score) AS sum_scores FROM results;
```

**Результат:**

<img width="1352" height="120" alt="image" src="https://github.com/user-attachments/assets/6b4fa907-665c-40b9-8f9c-a52ad89f8352" />

**GROUP BY и HAVING.** Группируем записи на курсы по статусу и оставляем только те статусы, которые встречаются более 100 раз.

```sql
SELECT status, COUNT(*) AS enrollments_count FROM enrollments GROUP BY status HAVING COUNT(*) > 100 ORDER BY enrollments_count DESC;
```

**Результат:**

<img width="826" height="119" alt="image" src="https://github.com/user-attachments/assets/60e15175-bc63-49fb-892b-9be8bc3ea0f5" />

**Вывод:** в базе 240 студентов. `HAVING` отбирает группы после агрегирования; условие применяется к `COUNT(*)`.

## Шаг 5. Объединение связанных таблиц

**Студенты и группы.** Используем `INNER JOIN`, чтобы показать первых 10 студентов вместе с названиями их групп.

```sql
SELECT s.id AS student_id, s.full_name, g.group_name 
FROM students s 
INNER JOIN groups g ON s.group_id = g.id 
ORDER BY s.id 
LIMIT 10;
```

**Результат:**

<img width="1296" height="666" alt="image" src="https://github.com/user-attachments/assets/256ae339-1bf8-4a05-add1-f6db9b3251b4" />

**Курсы и преподаватели.** Используем `INNER JOIN`, чтобы показать первые 10 курсов и то, какие преподаватели их ведут.

```sql
SELECT c.id, c.course_name, t.full_name AS teacher_name, t.department 
FROM courses c 
INNER JOIN teachers t ON c.teacher_id = t.id 
ORDER BY c.id 
LIMIT 10;
```

**Результат:**

<img width="1430" height="679" alt="image" src="https://github.com/user-attachments/assets/8ef9423b-4f42-482f-a0ce-9d525b7568f7" />

**Все группы и студенты.** Используем `LEFT JOIN`, чтобы сохранить в результате все группы, включая те, в которых пока нет студентов.

```sql
SELECT g.id, g.group_name, COUNT(s.id) AS students_count
FROM groups g
LEFT JOIN students s ON s.group_id = g.id
GROUP BY g.id, g.group_name
ORDER BY g.id;
```

**Результат:**

<img width="1131" height="769" alt="image" src="https://github.com/user-attachments/assets/beb0ccab-b0ac-4b50-9460-ff537c4a4cd3" />

**Вывод:** `INNER JOIN` выводит совпавшие записи, а `LEFT JOIN` сохраняет строки из левой таблицы даже при отсутствии соответствий.

## Шаг 6. Проверка корректности данных

**Проверка диапазона оценок.** Проверяем, нет ли оценок с отрицательным значением, значением больше 100 или отсутствующим значением.

```sql
SELECT COUNT(*) AS invalid_scores FROM results WHERE score < 0 OR score > 100 OR score IS NULL;
```

**Результат:**

<img width="168" height="109" alt="image" src="https://github.com/user-attachments/assets/643ee274-c463-475d-9254-34fda7c86253" />

**Проверка связей оценок.** Проверяем, нет ли оценок, ссылающихся на несуществующую запись на курс (битые внешние ключи).

```sql
SELECT COUNT(*) AS orphan_results FROM results r 
LEFT JOIN enrollments e ON r.enrollment_id = e.id 
WHERE e.id IS NULL;
```
**Результат:**

<img width="180" height="130" alt="image" src="https://github.com/user-attachments/assets/abc657e3-0dfa-45c5-9137-282d5b9318e3" />

**Вывод:** оба запроса вернули 0. В рамках проведённых проверок нарушений не обнаружено. Это не означает, что проверены абсолютно все возможные ошибки в данных.

## Шаг 7. Индивидуальное аналитическое исследование

**Вариант:** образовательная база.

**Исследовательский вопрос:** как успеваемость студентов (средний балл) зависит от преподавателей и их кафедр?

**Используемые таблицы:** `teachers`, `courses`, `enrollments`, `results`.

**Метод исследования:** с помощью `INNER JOIN`, `GROUP BY`, `COUNT` и `AVG` определяем количество выставленных оценок и средний балл отдельно по кафедрам и по конкретным преподавателям.

Для ответа на исследовательский вопрос выполняются два основных SQL-запроса.

**Аналитика по кафедрам.** Группируем оценки по кафедрам преподавателей и считаем средний балл.

```sql
SELECT t.department, 
       COUNT(r.score) AS total_grades, 
       ROUND(AVG(r.score), 2) AS avg_score 
FROM teachers t
INNER JOIN courses c ON t.id = c.teacher_id
INNER JOIN enrollments e ON c.id = e.course_id
INNER JOIN results r ON e.id = r.enrollment_id
GROUP BY t.department
ORDER BY avg_score DESC;
```

**Результат:**

<img width="1274" height="251" alt="image" src="https://github.com/user-attachments/assets/10fd0710-413f-44a1-be26-9625925a0c6f" />


**Аналитика по преподавателям.** Сравниваем преподавателей по среднему баллу, который получают их студенты (топ-10).

```sql
SELECT t.full_name AS teacher_name, 
       t.department,
       COUNT(r.score) AS total_grades, 
       ROUND(AVG(r.score), 2) AS avg_score 
FROM teachers t
INNER JOIN courses c ON t.id = c.teacher_id
INNER JOIN enrollments e ON c.id = e.course_id
INNER JOIN results r ON e.id = r.enrollment_id
GROUP BY t.id, t.full_name, t.department
HAVING COUNT(r.score) >= 5
ORDER BY avg_score DESC
LIMIT 10;
```

**Результат:**

<img width="1412" height="667" alt="image" src="https://github.com/user-attachments/assets/a84b1994-d323-437c-8d5d-831ee7e6b911" />

**Дополнительная проверка — курсы без студентов:**

**Неиспользуемые курсы.** Используем `LEFT JOIN` для поиска курсов, на которые не записался ни один студент.

```sql
SELECT c.id, c.course_name 
FROM courses c 
LEFT JOIN enrollments e ON c.id = e.course_id 
WHERE e.id IS NULL 
ORDER BY c.id;
```

**Результат:**

<img width="558" height="175" alt="image" src="https://github.com/user-attachments/assets/aeb80ce3-56cf-4683-beb2-7e771a4ce3ef" />

**Аналитические выводы:**

1. Наибольший средний балл по кафедрам показала кафедра «Математика» — 77.77 (на основе 299 оценок). Однако разрыв с другими кафедрами минимален: «Информатика» — 77.59, «Аналитика» — 77.35. Это говорит о том, что уровень успеваемости студентов стабилен и практически не зависит от профиля кафедры.
2. Среди преподавателей наивысший средний балл у «Преподаватель 14» (кафедра Аналитики) — 83.96 (на основе 28 оценок). В топ-10 также вошли преподаватели с достаточно большим количеством выставленных оценок (от 28 до 70), что делает эти результаты статистически достоверными.
3. В ходе дополнительной проверки обнаружено 2 неиспользуемых курса: «Проектный практикум» (ID 29) и «Интеграция систем» (ID 30). На них не записался ни один студент. Это можно использовать как повод для анализа спроса на данные дисциплины или для обновления учебного плана.

## Итог
В работе изучена структура SQLite-базы из шести связанных таблиц, выполнены запросы с `WHERE`, `ORDER BY`, `LIMIT`, агрегатными функциями, `GROUP BY`, `HAVING`, `INNER JOIN`, `LEFT JOIN`; проведены проверки целостности и отдельное аналитическое исследование.






