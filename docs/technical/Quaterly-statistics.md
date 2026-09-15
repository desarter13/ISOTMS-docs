# OUTDATED !!!! WIP !!! Quarterly Statistics Dashboard Description OUTDATED !!!! WIP !!!

Dashboard:
[https://datalens.yandex/app1gnnfg94cu](https://datalens.yandex/app1gnnfg94cu?_share_link=public)

---

# Dashboard Structure
- 'Quaterly LC statistics'
- 'Yearly Statistics'
- 'Technical'


---

# 'Quaterly LC statistics'

## 1. Total Students

```sql
SELECT
    lc.name AS lc,
    ay.name AS year_name,

    COUNT(DISTINCT sy.student_id) AS total_students

FROM student_years sy

JOIN students st
    ON st.id = sy.student_id

JOIN academic_years ay
    ON ay.id = sy.academic_year_id

JOIN learning_centers lc
    ON lc.id = sy.learning_center_id

JOIN attendances att
    ON att.student_year_id = sy.id

JOIN academic_periods ap
    ON ap.id = att.academic_period_id

WHERE
    ay.name = {{year_name}}
    AND lc.name = {{LC}}
    AND ap.name = 'Quarter 1'

GROUP BY
    lc.name,
    ay.name
```

## 2. Students for HR
```sql
SELECT
    lc.name AS lc,
    ay.name AS year_name,

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

## 3. Average Score
```sql
SELECT
    lc.name AS lc,
    ay.name AS year_name,

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
    AND sp.date BETWEEN ap.start_date AND ap.finish_date

WHERE
    ay.name = {{year_name}}
    AND lc.name = {{LC}}
    AND ap.name = 'Quarter 1'
    AND sp.is_pace = 1
    AND sp.status IN (1, 3)
    AND sp.result IS NOT NULL

GROUP BY
    lc.name,
    ay.name
```

## 4. Average PACEs Done
```sql
SELECT
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
## 5. Total PACEs Done
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
## 6. HR completion
```sql
SELECT
    ay.name AS year_name,
    lc.name AS learning_center_name,
    ap.name AS academic_period_name,

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

LEFT JOIN honor_rolls hr
    ON hr.student_year_id = sy.id
   AND hr.academic_period_id = ap.id

WHERE
    lc.name = {{LC}}
    AND ay.name = {{year_name}}

GROUP BY
    ay.name,
    lc.name,
    ap.name,
    ap.start_date

ORDER BY
    ap.start_date;
    ```
---

# Yearly Statistics

## Amount of students
 ```sql


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
    AND (lc.based_type = 0 OR lc.based_type = 2)
 ```

 ```sql
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
    AND sys.rus = 1

 ```
## PACEs DOne
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
    AND sp.date BETWEEN ay.start_date AND ay.finish_date;
 ```
## Average score
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
## Average PACEs Done
 ```sql
SELECT
    SUM(pace_count) / COUNT(*) AS avg_paces_per_student
FROM (
    SELECT
        sy.student_id,
        COUNT(sp.id) AS pace_count,
        COALESCE(sys.rus, 0) AS is_rus

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
    pace_count >= CASE
        WHEN is_rus = 1
            THEN 4 * {{min_paces_rus}}
        ELSE 4 * {{min_paces_eng}}
    END
 ```





