Quaterly statistics dashboard description

you can find it here

https://datalens.yandex/app1gnnfg94cu

dashboard structure

selectors

lwft side
Total students
Students for HR
Average score
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

average PACEs done

right side
Graph hr status by quarters
Graph on level by quarters
