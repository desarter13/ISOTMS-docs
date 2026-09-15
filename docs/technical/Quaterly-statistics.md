# Quarterly Statistics Dashboard — техническое описание

Дашборд: https://datalens.yandex/app1gnnfg94cu?_share_link=public

---

## Структура дашборда
- **Quarterly LC Statistics** — статистика по одному learning center в разрезе кварталов текущего учебного года
- **Yearly Statistics** — сводная статистика по всем learning centers за учебный год
- **Technical** — служебная вкладка

---

## Вкладка «Quarterly LC statistics»

Параметры фильтрации: `{{year_name}}` — учебный год, `{{LC}}` — конкретный learning center, `{{HR_status}}` — статус Honor Roll.

### 1. Total Students — количество студентов

```sql
SELECT
    lc.name AS lc,
    ay.name AS year_name,

    -- считаем уникальных студентов, у которых есть хотя бы одна отметка посещаемости в Quarter 1
    COUNT(DISTINCT sy.student_id) AS total_students

FROM student_years sy

JOIN students st
    ON st.id = sy.student_id

JOIN academic_years ay
    ON ay.id = sy.academic_year_id

JOIN learning_centers lc
    ON lc.id = sy.learning_center_id

-- attendances используется как фильтр "студент реально учился в этом квартале"
JOIN attendances att
    ON att.student_year_id = sy.id

JOIN academic_periods ap
    ON ap.id = att.academic_period_id

WHERE
    ay.name = {{year_name}}
    AND lc.name = {{LC}}
    AND ap.name = 'Quarter 1'      -- жёстко зашит Q1, для других кварталов нужен отдельный запрос / параметр

GROUP BY
    lc.name,
    ay.name
```

**Что получаем:** общее число студентов выбранного LC, посещавших занятия в Quarter 1 указанного года. Это базовая метрика для карточки "Total Students" (на дашборде — блок с числом 13 в примере).

---

### 2. Students for HR — количество студентов на Honor Roll

```sql
SELECT
    lc.name AS lc,
    ay.name AS year_name,

    -- считаем студентов, у которых honor_rolls.on_off = 1 (т.е. участвует в HR-программе)
    COUNT(DISTINCT CASE WHEN hr.on_off = 1 THEN st.id END) AS students_with_on_off_1

FROM student_years sy

JOIN students st
    ON st.id = sy.student_id

JOIN academic_years ay
    ON ay.id = sy.academic_year_id

JOIN learning_centers lc
    ON lc.id = sy.learning_center_id

JOIN academic_periods ap
    ON ap.academic_year_id = ay.id

-- честный JOIN (не LEFT), поэтому в выборку попадают только студенты, у которых есть запись в honor_rolls
JOIN honor_rolls hr
    ON hr.student_year_id = sy.id
    AND hr.academic_period_id = ap.id

WHERE
    ay.name = {{year_name}}
    AND lc.name = {{LC}}
    AND ap.name = 'Quarter 1'

GROUP BY
    lc.name,
    ay.name
```

**Что получаем:** число студентов, которые отмечены как участники Honor Roll (`on_off = 1`) в Quarter 1. Используется как знаменатель/база для расчёта % выполнения HR (см. запрос №6).

---

### 3. Average Score — средний балл

```sql
SELECT
    lc.name AS lc,
    ay.name AS year_name,

    -- средний результат по всем засчитанным PACE-тестам за квартал
    AVG(sp.result) AS avg_test_result

FROM student_paces sp

JOIN student_subjects ss
    ON ss.id = sp.student_subject_id

JOIN student_years sy
    ON sy.id = ss.student_year_id

JOIN academic_years ay
    ON ay.id = sy.academic_year_id

JOIN learning_centers lc
    ON lc.id = sy.learning_center_id

JOIN academic_periods ap
    ON ap.academic_year_id = ay.id
    AND sp.date BETWEEN ap.start_date AND ap.finish_date   -- квартал определяется по дате сдачи PACE, а не по attendance

WHERE
    ay.name = {{year_name}}
    AND lc.name = {{LC}}
    AND ap.name = 'Quarter 1'
    AND sp.is_pace = 1              -- берём только PACE, а не промежуточные тесты/чекпоинты
    AND sp.status IN (1, 3)         -- только записи со статусом "сдано" (успешно или пересдано)
    AND sp.result IS NOT NULL       -- исключаем записи без выставленного балла

GROUP BY
    lc.name,
    ay.name
```

**Что получаем:** средний результат (score) по всем успешно сданным PACE-тестам студентов LC за Quarter 1. Соответствует карточке "Average Score" (94,10 в примере).

---

### 4. Average PACEs Done — среднее количество PACE на студента

```sql
SELECT
    -- числитель: сколько всего PACE сдано в LC за квартал
    (SELECT COUNT(*)
     FROM student_paces sp
     JOIN student_subjects ss ON ss.id = sp.student_subject_id
     JOIN student_years sy ON sy.id = ss.student_year_id
     JOIN academic_years ay ON ay.id = sy.academic_year_id
     JOIN learning_centers lc ON lc.id = sy.learning_center_id
     JOIN academic_periods ap ON ap.academic_year_id = ay.id
        AND sp.date BETWEEN ap.start_date AND ap.finish_date
     WHERE ay.name = {{year_name}}
       AND lc.name = {{LC}}
       AND ap.name = 'Quarter 1'
       AND sp.is_pace = 1
       AND sp.status IN (1, 3)
    )
    /
    -- знаменатель: сколько студентов посещало LC в этот квартал (та же логика, что в запросе №1)
    (SELECT COUNT(DISTINCT sy.student_id)
     FROM student_years sy
     JOIN academic_years ay ON ay.id = sy.academic_year_id
     JOIN learning_centers lc ON lc.id = sy.learning_center_id
     JOIN attendances att ON att.student_year_id = sy.id
     JOIN academic_periods ap ON ap.id = att.academic_period_id
     WHERE ay.name = {{year_name}}
       AND lc.name = {{LC}}
       AND ap.name = 'Quarter 1'
    ) AS avg_paces_per_student
```

**Что получаем:** среднее число PACE, сданных одним студентом за квартал (total_paces / total_students). Карточка "Average PACEs" (14,77 в примере). ⚠️ Обратите внимание: два подзапроса используют разную логику подсчёта студентов (см. запросы 1 и 3) — если наборы студентов расходятся, среднее может искажаться.

---

### 5. Total PACEs Done — всего сдано PACE

```sql
SELECT
    COUNT(*) AS total_paces

FROM student_paces sp

JOIN student_subjects ss
    ON ss.id = sp.student_subject_id

JOIN student_years sy
    ON sy.id = ss.student_year_id

JOIN academic_years ay
    ON ay.id = sy.academic_year_id

JOIN learning_centers lc
    ON lc.id = sy.learning_center_id

JOIN academic_periods ap
    ON ap.academic_year_id = ay.id
    AND sp.date BETWEEN ap.start_date AND ap.finish_date

WHERE
    ay.name = {{year_name}}
    AND lc.name = {{LC}}
    AND ap.name = 'Quarter 1'
    AND sp.is_pace = 1
    AND sp.status IN (1, 3)
```

**Что получаем:** абсолютное количество сданных PACE по всему LC за Quarter 1 (карточка "Total PACEs", 192 в примере). Это тот же числитель, что использован в запросе №4.

---

### 6. HR completion — процент выполнения Honor Roll по кварталам

```sql
SELECT
    ay.name AS year_name,
    lc.name AS learning_center_name,
    ap.name AS academic_period_name,

    -- доля студентов, выполнивших HR (нужный status) среди всех участников HR-программы (on_off = 1)
    COUNT(DISTINCT CASE WHEN hr.on_off = 1 AND hr.status = {{HR_status}} THEN sy.student_id END) * 1.0
    /
    NULLIF(
        COUNT(DISTINCT CASE WHEN hr.on_off = 1 THEN sy.student_id END),
        0
    ) AS honor_roll_percent

FROM student_years sy

JOIN students st
    ON st.id = sy.student_id

JOIN academic_years ay
    ON ay.id = sy.academic_year_id

JOIN learning_centers lc
    ON lc.id = sy.learning_center_id

JOIN academic_periods ap
    ON ap.academic_year_id = ay.id

-- LEFT JOIN (в отличие от запроса №2), чтобы периоды без HR-записей тоже попали в выборку
LEFT JOIN honor_rolls hr
    ON hr.student_year_id = sy.id
   AND hr.academic_period_id = ap.id

WHERE
    lc.name = {{LC}}
    AND ay.name = {{year_name}}
    -- фильтра по ap.name нет — запрос возвращает сразу все 4 квартала

GROUP BY
    ay.name,
    lc.name,
    ap.name,
    ap.start_date

ORDER BY
    ap.start_date;
```

**Что получаем:** таблицу вида "квартал → % студентов, выполнивших Honor Roll со статусом {{HR_status}}" по всем четырём кварталам сразу. Питает верхний график "HR completion in % by quarters" (столбики 23%, 62%, 67%, 23% в примере).

---

## Вкладка «Yearly Statistics»

Параметр фильтрации: `{{year_name}}` — учебный год. LC отбираются по `based_type IN (0, 2)` — предположительно это фильтр "билингвальные + международные" LC (исключает прочие типы центров).

### Amount of students — общее количество студентов

```sql
-- Запрос 1: все студенты выбранного года среди LC нужного типа
SELECT
    COUNT(DISTINCT sy.student_id) AS total_students

FROM student_years sy

JOIN academic_years ay
    ON ay.id = sy.academic_year_id

JOIN learning_centers lc
    ON lc.id = sy.learning_center_id

JOIN students s
    ON s.id = sy.student_id

WHERE
    ay.name = {{year_name}}
    AND (lc.based_type = 0 OR lc.based_type = 2)   -- билингвальные (0) + международные (2) LC
```

**Что получаем:** общее число студентов по всем LC нужного типа за год. Карточка "Total students in school" (342 в примере).

```sql
-- Запрос 2: студенты с признаком rus = 1 (русскоязычные / российское гражданство?)
SELECT
    COUNT(DISTINCT sy.student_id) AS total_students
FROM student_years sy

JOIN academic_years ay
    ON ay.id = sy.academic_year_id

JOIN learning_centers lc
    ON lc.id = sy.learning_center_id

JOIN students s
    ON s.id = sy.student_id

JOIN student_year_settings sys
    ON sys.student_year_id = sy.id

WHERE
    ay.name = {{year_name}}
    AND (lc.based_type = 0 OR lc.based_type = 2)
    AND sys.rus = 1        -- флаг "rus" из student_year_settings — уточнить точное бизнес-значение
```

**Что получаем:** подмножество студентов с флагом `rus = 1`. Судя по дашборду, используется в паре с "не rus" для получения карточек "Bilingual" / "International" (132 и 210 соответственно) — вероятно `rus = 1` соответствует одной из этих категорий, а дополнение (`rus = 0` или NULL) — другой. ⚠️ На скрине дашборда нет отдельного запроса для "International" — уточнить, как считается вторая категория.

---

### PACEs Done — всего сдано PACE за год

```sql
SELECT
    COUNT(*) AS total_paces
FROM student_paces sp

JOIN student_subjects ss
    ON ss.id = sp.student_subject_id

JOIN student_years sy
    ON sy.id = ss.student_year_id

JOIN academic_years ay
    ON ay.id = sy.academic_year_id

JOIN learning_centers lc
    ON lc.id = sy.learning_center_id

WHERE
    ay.name = {{year_name}}
    AND lc.based_type IN (0, 2)
    AND sp.is_pace = 1
    AND sp.status IN (1, 3)          -- только выполненные PACE
    AND sp.date BETWEEN ay.start_date AND ay.finish_date;   -- считаем по датам всего учебного года, а не по кварталам
```

**Что получаем:** суммарное количество сданных PACE за весь учебный год по всем LC нужного типа. Карточка "All students" в блоке "PACEs done" (13 444 в примере).

---

### Average score — средний балл за год

```sql
SELECT
    AVG(sp.result) AS avg_test_result
FROM student_paces sp

JOIN student_subjects ss
    ON ss.id = sp.student_subject_id

JOIN student_years sy
    ON sy.id = ss.student_year_id

JOIN academic_years ay
    ON ay.id = sy.academic_year_id

JOIN learning_centers lc
    ON lc.id = sy.learning_center_id

WHERE
    ay.name = {{year_name}}
    AND lc.based_type IN (0, 2)
    AND sp.is_pace = 1
    AND sp.status IN (1, 3)        -- только выполненные PACE, без провальных попыток
    AND sp.date BETWEEN ay.start_date AND ay.finish_date
    AND sp.result IS NOT NULL;
```

**Что получаем:** средний балл по всем сданным PACE за учебный год. Карточка "Average Score → All students" (95,79 в примере).

---

### Average PACEs Done — среднее число PACE на студента за год (с фильтром по минимуму)

```sql
SELECT
    -- суммарное кол-во PACE у отобранных студентов / кол-во этих студентов
    SUM(pace_count) / COUNT(*) AS avg_paces_per_student
FROM (
    SELECT
        sy.student_id,
        COUNT(sp.id) AS pace_count,           -- сколько PACE сдал конкретный студент за год
        COALESCE(sys.rus, 0) AS is_rus        -- признак "rus", по умолчанию 0, если настройки нет

    FROM student_years sy

    JOIN academic_years ay
        ON ay.id = sy.academic_year_id

    JOIN learning_centers lc
        ON lc.id = sy.learning_center_id

    LEFT JOIN student_year_settings sys
        ON sys.student_year_id = sy.id

    LEFT JOIN student_subjects ss
        ON ss.student_year_id = sy.id

    LEFT JOIN student_paces sp
        ON sp.student_subject_id = ss.id
        AND sp.is_pace = 1
        AND sp.status IN (1, 3)
        AND sp.date BETWEEN ay.start_date AND ay.finish_date

    WHERE
        ay.name = {{year_name}}
        AND (lc.based_type = 0 OR lc.based_type = 2)

    GROUP BY
        sy.student_id,
        COALESCE(sys.rus, 0)
) AS t

WHERE
    -- отсекаем студентов, которые не набрали минимальный порог PACE за год
    -- (порог зависит от того, "rus" студент или нет — разные ожидаемые нагрузки)
    pace_count >= CASE
        WHEN is_rus = 1
            THEN 4 * {{min_paces_rus}}    -- порог для "rus"-студентов, умноженный на 4 (видимо, 4 квартала)
        ELSE 4 * {{min_paces_eng}}        -- порог для остальных студентов
    END
```

**Что получаем:** среднее число PACE на студента за год, но только среди студентов, выполнивших минимальный план (отсекаются те, кто набрал меньше `4 × min_paces_rus/eng` — то есть не дотянул до годовой нормы). Карточка "Average PACEs → All students" (71,93 в примере). ⚠️ Логика отличается от квартального расчёта (запрос №4 вкладки Quarterly) — там фильтра по минимуму нет, здесь есть.


### Age Groups
```sql
SELECT
    -- распределяем студентов по возрастным группам на возраст на 1 октября
    -- (в БД для этого используется ay.start_date, т.к. учебный год стартует 1 октября)
    COUNT(DISTINCT CASE WHEN age_years <= 3 THEN student_id END) AS preschool_3_or_younger,
    COUNT(DISTINCT CASE WHEN age_years BETWEEN 4 AND 5 THEN student_id END) AS kindergarten_4_5,
    COUNT(DISTINCT CASE WHEN age_years BETWEEN 6 AND 12 THEN student_id END) AS age_6_12,
    COUNT(DISTINCT CASE WHEN age_years >= 13 THEN student_id END) AS age_13_plus

FROM (
    SELECT
        sy.student_id,

        -- считаем "исполнившийся" возраст студента на 1 октября (ay.start_date),
        -- а не просто разницу лет — учитываем, наступил ли день рождения к этой дате
        CASE
            -- если день рождения в этом году ещё не наступил к 1 октября — вычитаем 1 год
            WHEN DATE_ADD(st.birthdate, INTERVAL EXTRACT(YEAR FROM ay.start_date) - EXTRACT(YEAR FROM st.birthdate) YEAR) > ay.start_date
            THEN (EXTRACT(YEAR FROM ay.start_date) - EXTRACT(YEAR FROM st.birthdate)) - 1
            -- иначе день рождения уже был — берём разницу лет как есть
            ELSE EXTRACT(YEAR FROM ay.start_date) - EXTRACT(YEAR FROM st.birthdate)
        END AS age_years

    FROM student_years sy

    JOIN students st
        ON st.id = sy.student_id

    JOIN academic_years ay
        ON ay.id = sy.academic_year_id

    JOIN learning_centers lc
        ON lc.id = sy.learning_center_id

    WHERE
        ay.name = {{year_name}}
        AND (lc.based_type = 0 OR lc.based_type = 2)   -- билингвальные (0) + международные (2) LC
) t;
```

---

## Общие технические заметки
- Даты для attendance-based метрик определяются через `attendances` (реальное посещение), а для PACE-based метрик — через `sp.date BETWEEN ap.start_date AND ap.finish_date` (дата сдачи теста). Эти два подхода могут давать разные наборы студентов.
- `sp.status IN (1, 3)` — используется как признак "PACE засчитан" во всех запросах; нужно явно задокументировать, что означают статусы 1 и 3 в справочнике (например: 1 = сдано с первого раза, 3 = пересдано успешно).
- `lc.based_type` — принимает как минимум значения 0 и 2, используемые в фильтрах Yearly Statistics; значение и назначение остальных типов не описано — стоит зафиксировать полный список.
- `student_year_settings.rus` — используется как разделитель "Bilingual vs International", но явного соответствия в коде нет — стоит уточнить и явно закомментировать в дашборде.
- Квартал захардкожен как `'Quarter 1'` в большинстве запросов вкладки Quarterly LC — вероятно, для остальных кварталов существуют аналогичные копии запроса с заменой строки; стоит либо параметризовать `ap.name`, либо явно перечислить все 4 версии в документации.



