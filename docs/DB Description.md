### academic_periods
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| name | String | название четверти |
| start_date | Date | начало четверти |
| finish_date | Date | конец четверти |
| academic_year_id | Integer | id года в таблице academic_years |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |

### academic_years
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| name | String | имя года в формате nnnn/nnnn |
| start_date | Date | начало года |
| finish_date | Date | конец года |
| period_type | Integer | семестр/триместр/четверти |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |
| status | Integer | не используется |

### attendances
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| value | String | Present — 1, Absent — 2, Tardy — 3, Ill — 4, пятое состояние — это отсутствие записей по данному числу |
| date | Date | дата дня |
| academic_period_id | Integer | id из таблицы academic_period |
| student_year_id | Integer | id из таблицы student_year |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |

### goal_check_day_subjects
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| home_work | Integer | нету — 0, есть — 1, выполнено — 2 |
| reapeated_goals | Integer | число дней которые ученик повторяет цель для предмета |
| checkout | Integer | нету — 0, есть — 1, выполнено — 2 |
| selftest | Integer | нету — 0, есть — 1, выполнено — 2 |
| test | Integer | нету — 0, есть — 1, выполнено — 2 |
| vocabulary | Integer | нету — 0, есть — 1, выполнено — 2 |
| rules | Integer | нету — 0, есть — 1, выполнено — 2 |
| comment | String | строка, в которую помещается коммент |
| student_subject_id | Integer | id из таблицы student_subject |
| goal_check_day_id | Integer | id из таблицы goal_check_day |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |
| status | Integer | статус предмета для зелёной точки 0 - нет зеленой точки, 1 - есть зеленая точка|

### goal_check_days
| Имя | Тип | Описание |
|-------|-------|-------|
| detention_comment | String | текст коммента для detention |
| comment | String | текст коммента |
| id | Integer | id строки в данной таблице |
| demerit | Integer | общее кол‑во за день |
| demerit_confirmed | Integer | ???? |
| attendance | Integer | думал перенести сюда, но потом оставил отдельной таблицей |
| detention_completed | Integer |  |
| student_year_id | Integer | id из таблицы student_year |
| completed_pages | Integer | количество законченных страниц |
| set_pages | Integer | количество страниц, которые мы ставим на день |
| projected_pages | Integer | количество страниц, которые бы надо сделать |
| status | Integer | зелёная точка |
| set_pages_user_id | Integer | id пользователя из таблицы user |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |
| date | Date | Дата, к которой относится запись |
| detention_date | Date |  |

### learning_centers
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| name | String |  |
| user_id | Integer |  |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |
| type | Integer | Default, Kindergarten, ABC, Junior, Primary, Senior |
| based_type | Integer | Campus-based, Homeschooling, Distant |
| demerit_type | Integer |  |
| sort_order | Integer |  |

### student_subjects
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| subject_type_id | Integer | id из таблицы subject_type |
| subject_id | Integer | id из таблицы subject |
| student_year_id | Integer | id из таблицы student_year |
| total | Fractional number | среднее за предмет |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |
| status | Integer | 0 — активен, 1 — закончен |

### student_years
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| level | String | Уровень, который должен быть у студента по возрасту |
| status | String | ?????? |
| academic_year_id | Integer |  id из таблицы academic_years |
| learning_center_id | Integer | id из таблицы learning_centers |
| student_id | Integer | id из таблицы students |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |
| current_level | Integer | уровень, который в данный момент у студента без математики |
| current_math_level | Integer | уровень студента по математике |
| grade_level | Integer | уровень согласно русской программе |
| sort_order | Integer | число для сортировки для GoalCheck |

### students
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| user_id | Integer | id пользователя, который ее занес  |
| family_id | Integer | id семь |
| country_id | Integer | id страны |
| first_name | String | Имя  |
| middle_name | String | Отчество |
| last_name | String | Фамилия  |
| birthdate | Date | Дата рождения  |
| gender | Integer |  |
| email | String |  |
| status | Integer | статус студента (выключается после withdrawal) |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |

### subject_paces
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| subject_id | Integer |  |
| number | Integer |  |
| pages | Integer |  |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |

### subject_types
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| name | String |  |
| color | String |  |
| sort_order | Integer |  |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |
| short_name | String |  |
| pace_sort_order | Integer |  |

### Subjects
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| subject_type_id | Integer |  |
| name | String |  |
| gpa_weight | Fractional number |  |
| start_pace | Integer |  |
| finish_pace | Integer |  |
| level | String |  |
| is_pace | Integer |  |
| is_class | Integer |  |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |
| diploma_level | String | ????? |
| level_number | Integer |  |
| count_for_level | Integer | числовое значение level |

### honor_rolls
| Имя | Тип | Описание |
|-------|-------|-------|
| name |  |  |
| id |  |  |
| student_year_id |  |  |
| academic_period_id |  |  |
| date |  |  |
| status |  |  |
| created_at |  |  |
| updated_at |  |  |
| on_off |  |  |
| total_pages |  |  |
| done_pages |  |  |

### student_paces
| Имя | Тип | Описание |
|-------|-------|-------|
| id | Integer | id строки в данной таблице |
| student_subject_id | Integer | id из таблицы student_subject |
| number | Integer | номер этапа/задания |
| result | Integer | результат выполнения (например, оценка или статус) |
| date | Date | дата выполнения |
| status | Integer | статус выполнения (0 — не выполнено, 1 — выполнено и т. д.) |
| created_at | Date and time | дата создания записи |
| updated_at | Date and time | дата последнего обновления |
| is_pace | Integer | флаг, указывающий, относится ли запись к темпу обучения (0 — нет, 1 — да) |


## ER diagram
STUDENT_YEARS (student_year_id)
│
├── ACADEMIC_YEARS (academic_year_id)
│   │
│   ├── ACADEMIC_PERIODS (academic_period_id)
│   │   │
│   │   └── ATTENDANCES (academic_period_id, student_year_id)
│   │
│   └── (обратная связь) STUDENT_YEARS
│
├── LEARNING_CENTERS (learning_center_id)
│   │
│   └── (обратная связь) STUDENT_YEARS
│
├── STUDENTS (student_id)
│   │
│   └── (обратная связь) STUDENT_YEARS
│
├── ATTENDANCES (student_year_id)
│
├── GOAL_CHECK_DAYS (student_year_id)
│   │
│   └── GOAL_CHECK_DAY_SUBJECTS (goal_check_day_id)
│       │
│       └── STUDENT_SUBJECTS (student_subject_id)
│           │
│           ├── SUBJECTS (subject_id)
│           │   │
│           │   ├── SUBJECT_PACES (subject_id)
│           │   │
│           │   └── SUBJECT_TYPES (subject_type_id)
│           │       │
│           │       └── (обратная связь) STUDENT_SUBJECTS
│           │
│           └── STUDENT_PACES (student_subject_id)
│
├── HONOR_ROLLS (student_year_id)
│   │
│   ├── ACADEMIC_PERIODS (academic_period_id)  [через связь с ACADEMIC_YEARS]
│   │
│   └── (обратная связь) STUDENT_YEARS
│
└── STUDENT_SUBJECTS (student_year_id)
    │
    ├── SUBJECTS (subject_id)
    │   │
    │   ├── SUBJECT_PACES (subject_id)
    │   │
    │   └── SUBJECT_TYPES (subject_type_id)
    │
    ├── GOAL_CHECK_DAY_SUBJECTS (student_subject_id)
    │
    └── STUDENT_PACES (student_subject_id)
