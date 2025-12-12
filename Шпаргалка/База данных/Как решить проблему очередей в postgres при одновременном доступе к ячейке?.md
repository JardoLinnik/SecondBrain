> В PostgreSQL конфликтуют **не «ячейки», а строки**. При одновременной записи действует **row-level locking**: первая транзакция берёт эксклюзивный блок на строку, остальные конкуренты становятся в очередь.
> 
> **Читать** можно параллельно (MVCC: читатели не блокируют писателей и наоборот), а вот **пишущие** конкуренты блокируются.
> 
> Пользователи **не будут ждать вечно**:
> 
> 1. PostgreSQL обнаруживает **взаимные блокировки (deadlock)** и роняет одну из транзакций ошибкой `deadlock detected`;
>     
> 2. мы можем настроить **таймауты** (`lock_timeout`, `statement_timeout`, `idle_in_transaction_session_timeout`).
>     
> 
> Кроме «ждать» есть ещё варианты: **NOWAIT**, **SKIP LOCKED**, **оптимистическая блокировка** через версию, **советнические (advisory) блокировки**. Для очередей — типовой паттерн `SELECT … FOR UPDATE SKIP LOCKED`.

---

## Что именно происходит при конкуренции

- **UPDATE/DELETE/SELECT … FOR UPDATE**: первая транзакция T1 ставит X-lock на строку. Вторая T2 встаёт в очередь на этот же X-lock.
    
- **READ COMMITTED**: читатели видят предыдущую подтверждённую версию строки и **не ждут**.
    
- **REPEATABLE READ/ SERIALIZABLE**: читатели видят снимок на начало транзакции.
    
- **Deadlock**: если T1 ждёт T2, а T2 ждёт T1 (или цикл длиннее), Postgres через короткий таймер deadlock-детектора завершит одну из них ошибкой — ожидания «вечно» не будет.
    
- **Таймауты**: если мы выставили `lock_timeout = '2s'`, ожидание чужого блокировки прерывается ошибкой по таймауту.
    

---

## Что могут делать другие транзакции, кроме как «ждать»

### 1) Взять без ожидания — `NOWAIT`

Сразу упасть, если строка занята — удобно для API с быстрой деградацией.

`-- Пытаемся заблокировать строку без ожидания SELECT * FROM accounts WHERE id = $1 FOR UPDATE NOWAIT; -- если занято -> ERROR: could not obtain lock on row`

### 2) Пропустить занятое — `SKIP LOCKED`

Идеальный инструмент для **очередей / шардированных задач**.

`-- Забираем "свободную" задачу из очереди WITH cte AS (   SELECT id   FROM job_queue   WHERE status = 'pending'   ORDER BY id   FOR UPDATE SKIP LOCKED   LIMIT 1 ) UPDATE job_queue j SET status = 'processing', taken_at = now() FROM cte WHERE j.id = cte.id RETURNING j.*;`

> Так N воркеров не толкаются за одну запись: каждый «подберёт» свою свободную.

### 3) Оптимистическая конкуренция (версионирование)

Без явных блокировок: проверяем версию/хэш в `WHERE`.

`UPDATE accounts SET balance = balance + 100, version = version + 1 WHERE id = $1 AND version = $2; -- rowcount == 1 -> успех; 0 -> конфликт, повторяем`

### 4) Advisory locks (советнические)

Грубая, но удобная логическая блокировка по ключу (например, по client_id).

`SELECT pg_try_advisory_lock(hashtext('client:' || $1)); -- true -> работаем; false -> кто-то уже держит -- ... SELECT pg_advisory_unlock(hashtext('client:' || $1));`

Используем, когда «единица синхронизации» — не строка, а бизнес-ключ.

---

## Будут ли ждать вечно?

Коротко — **нет**:

1. **Deadlock detector** принудительно завершит одну транзакцию с ошибкой, если обнаружит цикл ожиданий.
    
2. Системные **таймауты** ограничивают ожидание:
    
    - `lock_timeout` — лимит ожидания блокировки;
        
    - `statement_timeout` — общий лимит на выполнение запроса;
        
    - `idle_in_transaction_session_timeout` — сессии не лежат «полумёртвыми» с открытой транзакцией.
        

В производстве **обязательно** выставляем разумные значения (например, `lock_timeout = 2s`, `statement_timeout = 30s`) и логируем ошибки.

---

## Паттерны для очередей (как делали в Т-Банке)

1. **Очередь в таблице + SKIP LOCKED**
    
    - Таблица `job_queue(status, shard, payload, taken_at)`
        
    - Воркеры выбирают `LIMIT 1 … FOR UPDATE SKIP LOCKED` по своему `shard`.
        
    - SLA надёжности: «at-least-once» + дедубликация по `job_id`.
        
2. **Шардирование ключом**
    
    - Разбиваем очередь по `shard` (хеш клиента/продукта), чтобы избежать head-of-line blocking.
        
3. **Переотдача задач (requeue)**
    
    - Периодический джоб помечает «зависшие» задачи `processing` с `taken_at < now() - interval 'N minutes'` обратно в `pending`.
        
4. **Таймауты на воркерах**
    
    - На уровне драйвера/ORM выставляем `lock_timeout`, `statement_timeout`; ретраи с джиттером.
        

---

## Пример: конкурирующие списания с одного счёта

**SQL (с защитой от гонок):**

`BEGIN;  -- 1) Блокируем счёт SELECT id, balance FROM accounts WHERE id = $1 FOR UPDATE;  -- 2) Проверяем и списываем UPDATE accounts SET balance = balance - $amount WHERE id = $1 AND balance >= $amount;  -- 3) Контроль результата -- rowcount == 1 -> commit; иначе rollback (недостаточно средств) COMMIT;`

**Альтернатива (оптимистично):**

`UPDATE accounts SET balance = balance - $amount, version = version + 1 WHERE id = $1 AND balance >= $amount AND version = $version;`

---

## Короткий чек-лист (что ответить, если спросят «а что вы делаете в проде?»)

- **Избегаем долгих транзакций** (меньше шансов держать lock).
    
- Везде где можно — **`SKIP LOCKED`** для конкурирующих воркеров.
    
- **Таймауты** (`lock_timeout`, `statement_timeout`) и **ретраи** с backoff+jitter.
    
- **Оптимистическая блокировка** через `version` там, где это уместно.
    
- Для «логических ключей» — **advisory locks**.
    
- Мониторим `pg_locks`, логируем wait events, на инцидентах — `pg_stat_activity`, `pg_blocking_pids()`.