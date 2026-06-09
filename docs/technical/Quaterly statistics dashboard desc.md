# Quarterly Statistics Dashboard Description

Dashboard:
https://datalens.yandex/app1gnnfg94cu

---

# Dashboard Structure

## Selectors

- `year_name`
- `LC`
- `HR_status`

---

# LEFT SIDE WIDGETS

## 1. Total Students

```sql
SELECT
    COUNT(DISTINCT st.id) AS total_students,
    lc.name AS LC,
    ay.name AS year_name,
    ap.name AS academic_period_name
FROM student_years sy
JOIN students st ON st.id = sy.student_id
JOIN academic_years ay ON ay.id = sy.academic_year_id
JOIN learning_centers lc ON lc.id = sy.learning_center_id
JOIN attendances a ON a.student_year_id = sy.id
JOIN academic_periods ap ON ap.id = a.academic_period_id
LEFT JOIN honor_rolls hr ON hr.student_year_id = sy.id
WHERE ay.name = {{year_name}}
  AND lc.name = {{LC}}
  AND ap.name = 'Quarter 1'
GROUP BY ap.name, lc.name, ay.name;
```

## 2. Students for HR
```sql
SELECT
    COUNT(DISTINCT st.id) AS students_for_hr,
    lc.name AS LC,
    ay.name AS year_name,
    ap.name AS academic_period_name
FROM student_years sy
JOIN students st ON st.id = sy.student_id
JOIN academic_years ay ON ay.id = sy.academic_year_id
JOIN learning_centers lc ON lc.id = sy.learning_center_id
JOIN attendances a ON a.student_year_id = sy.id
JOIN academic_periods ap ON ap.id = a.academic_period_id
LEFT JOIN honor_rolls hr ON hr.student_year_id = sy.id
WHERE ay.name = {{year_name}}
  AND lc.name = {{LC}}
  AND ap.name = 'Quarter 1'
  AND hr.on_off = 1
GROUP BY ap.name, lc.name, ay.name;
```

## 3. Average Score
```sql
SELECT
    ap.name AS academic_period_name,
    lc.name AS LC,
    AVG(sp.result) AS average_grade
FROM student_paces sp
JOIN student_subjects ss ON sp.student_subject_id = ss.id
JOIN student_years sy ON ss.student_year_id = sy.id
JOIN learning_centers lc ON sy.learning_center_id = lc.id
JOIN attendances a ON sp.date = a.date
    AND a.student_year_id = sy.id
JOIN academic_periods ap ON a.academic_period_id = ap.id
JOIN academic_years ay ON ap.academic_year_id = ay.id
WHERE sp.is_pace = 1
  AND sp.result IS NOT NULL
  AND ay.name = {{year_name}}
  AND lc.name = {{LC}}
  AND ap.name = 'Quarter 4'
GROUP BY ap.name, lc.name;
```

## 4. Average PACEs Done
```sql
SELECT
    COUNT(DISTINCT st.id) AS students_with_paces,
    lc.name AS LC,
    ay.name AS year_name,
    ap.name AS academic_period_name
FROM student_years sy
JOIN students st ON st.id = sy.student_id
JOIN academic_years ay ON ay.id = sy.academic_year_id
JOIN learning_centers lc ON lc.id = sy.learning_center_id
JOIN attendances a ON a.student_year_id = sy.id
JOIN academic_periods ap ON ap.id = a.academic_period_id
LEFT JOIN honor_rolls hr ON hr.student_year_id = sy.id
WHERE ay.name = {{year_name}}
  AND lc.name = {{LC}}
  AND ap.name = 'Quarter 1'
GROUP BY ap.name, lc.name, ay.name;
```

# RIGHT SIDE WIDGETS
## 5. HR Status by Quarters
```sql
SELECT
    ay.name AS year_name,
    lc.name AS learning_center_name,
    ap.name AS academic_period_name,

    COUNT(CASE WHEN hr.status = {{HR_status}} THEN 1 END)
    / NULLIF(COUNT(DISTINCT sy.student_id), 0) AS honor_roll_percent

FROM student_years sy
JOIN students st ON st.id = sy.student_id
JOIN academic_years ay ON ay.id = sy.academic_years_id
JOIN learning_centers lc ON lc.id = sy.learning_center_id
JOIN academic_periods ap ON ap.academic_years_id = ay.id
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
## 6. On Level by Quarters
-- TODO: define metric logic


