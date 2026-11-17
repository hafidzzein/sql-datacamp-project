# Student Mental Health Analysis (SQL / PostgreSQL)

An SQL data analysis project from DataCamp. It looks at whether studying abroad affects students' mental health, using survey data from a Japanese international university.

## Background

In 2018 a Japanese international university surveyed its students and published a study the following year. The study found that:

- international students have a higher risk of mental health difficulties than the general population, and
- **social connectedness** (belonging to a social group) and **acculturative stress** (stress from adjusting to a new culture) predict depression.

This project uses PostgreSQL on the `students` table to check whether the same conclusions hold for international students, and whether **length of stay** plays a role.

## Dataset

The `students` table has 286 rows and 50 columns. These are the columns used in the analysis:

| Field Name      | Description                                         |
| --------------- | --------------------------------------------------- |
| `inter_dom`     | Type of student (international or domestic)         |
| `japanese_cate` | Japanese language proficiency                       |
| `english_cate`  | English language proficiency                        |
| `academic`      | Current academic level (undergraduate or graduate)  |
| `age`           | Current age of student                              |
| `stay`          | Current length of stay in years                     |
| `todep`         | Total depression score (PHQ-9 test)                 |
| `tosc`          | Total social connectedness score (SCS test)         |
| `toas`          | Total acculturative stress score (ASISS test)       |

**Data cleaning note:** the 286 rows are 201 international students, 67 domestic students and 18 rows with an empty student type. Those 18 rows store `inter_dom` as an empty string rather than `NULL` and have no test scores, so they are excluded.

## Analysis and Findings

### 1. Length of stay vs mental health (international students)

```sql
SELECT stay,
       COUNT(*)              AS count_int,
       ROUND(AVG(todep), 2)  AS average_phq,
       ROUND(AVG(tosc), 2)   AS average_scs,
       ROUND(AVG(toas), 2)   AS average_as
FROM students
WHERE inter_dom = 'Inter'
GROUP BY stay
ORDER BY stay DESC;
```

| stay | count_int | average_phq | average_scs | average_as |
| ---: | --------: | ----------: | ----------: | ---------: |
| 10   | 1         | 13.00       | 32.00       | 50.00      |
| 8    | 1         | 10.00       | 44.00       | 65.00      |
| 7    | 1         | 4.00        | 48.00       | 45.00      |
| 6    | 3         | 6.00        | 38.00       | 58.67      |
| 5    | 1         | 0.00        | 34.00       | 91.00      |
| 4    | 14        | 8.57        | 33.93       | 87.71      |
| 3    | 46        | 9.09        | 37.13       | 78.00      |
| 2    | 39        | 8.28        | 37.08       | 77.67      |
| 1    | 95        | 7.48        | 38.11       | 72.80      |

- For students staying 1–4 years (194 of the 201), acculturative stress rises steadily from **72.80 to 87.71**.
- Depression scores rise from **7.48 to 9.09** by year 3.
- Social connectedness is lowest in year 4 (**33.93**).
- Groups staying 5 or more years have only 1–3 students each, so their averages are not reliable.

### 2. International vs domestic students

```sql
SELECT inter_dom,
       COUNT(*)              AS jumlah,
       ROUND(AVG(todep), 2)  AS average_phq,
       ROUND(AVG(tosc), 2)   AS average_scs,
       ROUND(AVG(toas), 2)   AS average_as
FROM students
WHERE inter_dom IN ('Inter', 'Dom')
GROUP BY inter_dom;
```

| inter_dom | jumlah | average_phq | average_scs | average_as |
| --------- | -----: | ----------: | ----------: | ---------: |
| Dom       | 67     | 8.61        | 37.64       | 62.84      |
| Inter     | 201    | 8.04        | 37.42       | 75.56      |

International students show much higher acculturative stress (**75.56 vs 62.84**), while depression and social connectedness are similar in both groups. Higher cultural stress does not directly mean a higher average depression score.

### 3. Japanese proficiency (international students)

```sql
SELECT japanese_cate,
       COUNT(*)              AS jumlah,
       ROUND(AVG(todep), 2)  AS average_phq,
       ROUND(AVG(toas), 2)   AS average_as
FROM students
WHERE inter_dom = 'Inter'
GROUP BY japanese_cate
ORDER BY average_as DESC;
```

| japanese_cate | jumlah | average_phq | average_as |
| ------------- | -----: | ----------: | ---------: |
| Low           | 91     | 7.91        | 76.65      |
| Average       | 85     | 8.38        | 75.45      |
| High          | 25     | 7.40        | 72.00      |

Acculturative stress goes down as Japanese proficiency goes up (Low 76.65 → Average 75.45 → High 72.00). Depression scores show no clear pattern across proficiency levels.

### 4. Correlation analysis (international students)

```sql
SELECT ROUND(CORR(todep, tosc)::numeric, 2) AS depresi_vs_sosial,
       ROUND(CORR(todep, toas)::numeric, 2) AS depresi_vs_stres
FROM students
WHERE inter_dom = 'Inter';
```

| Relationship                               | Correlation | Strength          |
| ------------------------------------------ | ----------: | ----------------- |
| Depression vs social connectedness         | **-0.54**   | Moderate negative |
| Depression vs acculturative stress         | **0.41**    | Moderate positive |

These results match the original study: social connectedness and acculturative stress are stronger indicators of depression than length of stay.

## Conclusion

1. **Length of stay alone does not clearly explain depression** among international students.
2. **Social connectedness has the strongest relationship with depression** (r = -0.54), followed by **acculturative stress** (r = 0.41).
3. **Japanese proficiency helps reduce cultural stress**, but does not directly lower depression.

**Recommendation:** universities could prioritize programs that build social connection, such as buddy programs and student communities, alongside language support.

> **Note:** these findings show correlation, not causation, and some groups have small sample sizes.

## Tools

- PostgreSQL (aggregations, `GROUP BY`, `CORR`)
- DataCamp Workspace (DataLab)
