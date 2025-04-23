# บทที่ 7: การใช้ JOIN เชื่อมโยงหลายตาราง

## สารบัญ

- [INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL JOIN](#inner-join-left-join-right-join-full-join)
- [การใช้ ON และ USING](#การใช้-on-และ-using)
- [การ JOIN มากกว่า 2 ตาราง](#การ-join-มากกว่า-2-ตาราง)

## `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL JOIN`
ใน SQL การ **JOIN** ใช้สำหรับ**รวมข้อมูลจากตารางตั้งแต่สองตารางขึ้นไป**โดยอ้างอิงความสัมพันธ์ระหว่างคอลัมน์ในตารางเหล่านั้น การ JOIN มีหลายประเภท ได้แก่ **INNER JOIN**, **LEFT JOIN**, **RIGHT JOIN**, และ **FULL JOIN** ซึ่งแต่ละประเภทมีพฤติกรรมแตกต่างกันในการเลือกแถวจากตารางที่เกี่ยวข้อง ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### ประเภทของ JOIN
1. **INNER JOIN**: เลือกเฉพาะแถวที่ตรงกันในทั้งสองตาราง (แถวที่เงื่อนไข JOIN เป็นจริง)
2. **LEFT JOIN** (หรือ LEFT OUTER JOIN): เลือกทุกแถวจากตารางด้านซ้าย และแถวที่ตรงกันจากตารางด้านขวา (ถ้าไม่ตรงจะคืน `NULL` สำหรับคอลัมน์ของตารางด้านขวา)
3. **RIGHT JOIN** (หรือ RIGHT OUTER JOIN): เลือกทุกแถวจากตารางด้านขวา และแถวที่ตรงกันจากตารางด้านซ้าย (ถ้าไม่ตรงจะคืน `NULL` สำหรับคอลัมน์ของตารางด้านซ้าย)
4. **FULL JOIN** (หรือ FULL OUTER JOIN): เลือกทุกแถวจากทั้งสองตาราง โดยคืน `NULL` ในคอลัมน์ของตารางที่ไม่มีแถวตรงกัน

### ไวยากรณ์พื้นฐาน:
```sql
SELECT columns
FROM table1
[INNER | LEFT | RIGHT | FULL] JOIN table2
ON condition
[WHERE condition]
[GROUP BY columns]
[HAVING condition]
[ORDER BY columns];
```

- **table1, table2**: ตารางที่ต้องการรวม
- **ON condition**: เงื่อนไขที่กำหนดความสัมพันธ์ระหว่างตาราง (เช่น `table1.column = table2.column`)
- **columns**: คอลัมน์ที่ต้องการเลือกจากทั้งสองตาราง

### ตัวอย่างการใช้งาน:
สมมติมีสองตาราง:
1. **employees**:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    department_id NUMBER(5)
);
INSERT INTO employees VALUES (1, 'John', 10);
INSERT INTO employees VALUES (2, 'Jane', 20);
INSERT INTO employees VALUES (3, 'Alice', 30);
INSERT INTO employees VALUES (4, 'Bob', NULL);
COMMIT;
```

2. **departments**:
```sql
CREATE TABLE departments (
    department_id NUMBER(5) PRIMARY KEY,
    department_name VARCHAR2(50)
);
INSERT INTO departments VALUES (10, 'Sales');
INSERT INTO departments VALUES (20, 'IT');
INSERT INTO departments VALUES (40, 'HR');
COMMIT;
```

#### 1. INNER JOIN
- **คำอธิบาย**: เลือกเฉพาะแถวที่มีค่า `department_id` ตรงกันในทั้งสองตาราง
- **ตัวอย่าง**:
  ```sql
  SELECT e.first_name, e.department_id, d.department_name
  FROM employees e
  INNER JOIN departments d
  ON e.department_id = d.department_id;
  ```
- **ผลลัพธ์**:
  ```
  first_name | department_id | department_name
  ------------+---------------+----------------
  John       | 10            | Sales
  Jane       | 20            | IT
  ```
- **อธิบาย**:
  - Alice (department_id = 30) และ Bob (department_id = NULL) ไม่ปรากฏ เพราะไม่มี `department_id` ที่ตรงกันในตาราง `departments`
  - แผนก HR (department_id = 40) ไม่ปรากฏ เพราะไม่มีพนักงานที่ตรงกับแผนกนี้

#### 2. LEFT JOIN
- **คำอธิบาย**: เลือกทุกแถวจากตารางด้านซ้าย (`employees`) และแถวที่ตรงกันจากตารางด้านขวา (`departments`) ถ้าไม่ตรง คอลัมน์จากตารางด้านขวาจะเป็น `NULL`
- **ตัวอย่าง**:
  ```sql
  SELECT e.first_name, e.department_id, d.department_name
  FROM employees e
  LEFT JOIN departments d
  ON e.department_id = d.department_id;
  ```
- **ผลลัพธ์**:
  ```
  first_name | department_id | department_name
  ------------+---------------+----------------
  John       | 10            | Sales
  Jane       | 20            | IT
  Alice      | 30            | NULL
  Bob        | NULL          | NULL
  ```
- **อธิบาย**:
  - ทุกพนักงานจาก `employees` ปรากฏในผลลัพธ์
  - Alice และ Bob ไม่มี `department_id` ที่ตรงกับ `departments` ดังนั้น `department_name` เป็น `NULL`

#### 3. RIGHT JOIN
- **คำอธิบาย**: เลือกทุกแถวจากตารางด้านขวา (`departments`) และแถวที่ตรงกันจากตารางด้านซ้าย (`employees`) ถ้าไม่ตรง คอลัมน์จากตารางด้านซ้ายจะเป็น `NULL`
- **ตัวอย่าง**:
  ```sql
  SELECT e.first_name, e.department_id, d.department_name
  FROM employees e
  RIGHT JOIN departments d
  ON e.department_id = d.department_id;
  ```
- **ผลลัพธ์**:
  ```
  first_name | department_id | department_name
  ------------+---------------+----------------
  John       | 10            | Sales
  Jane       | 20            | IT
  NULL       | NULL          | HR
  ```
- **อธิบาย**:
  - ทุกแผนกจาก `departments` ปรากฏในผลลัพธ์
  - แผนก HR (department_id = 40) ไม่มีพนักงานที่ตรงกัน ดังนั้น `first_name` และ `department_id` จาก `employees` เป็น `NULL`
  - Alice และ Bob ไม่ปรากฏ เพราะ `department_id` ของพวกเขา (30 และ NULL) ไม่ตรงกับแผนกใดใน `departments`

#### 4. FULL JOIN
- **คำอธิบาย**: เลือกทุกแถวจากทั้งสองตาราง โดยคืน `NULL` ในคอลัมน์ของตารางที่ไม่มีแถวตรงกัน
- **ตัวอย่าง**:
  ```sql
  SELECT e.first_name, e.department_id, d.department_name
  FROM employees e
  FULL JOIN departments d
  ON e.department_id = d.department_id;
  ```
- **ผลลัพธ์**:
  ```
  first_name | department_id | department_name
  ------------+---------------+----------------
  John       | 10            | Sales
  Jane       | 20            | IT
  Alice      | 30            | NULL
  Bob        | NULL          | NULL
  NULL       | NULL          | HR
  ```
- **อธิบาย**:
  - รวมทุกแถวจาก `employees` และ `departments`
  - Alice (department_id = 30) และ Bob (department_id = NULL) ไม่มีแผนกที่ตรงกัน ดังนั้น `department_name` เป็น `NULL`
  - แผนก HR (department_id = 40) ไม่มีพนักงานที่ตรงกัน ดังนั้น `first_name` และ `department_id` เป็น `NULL`

### การใช้งานร่วมกับเงื่อนไขเพิ่มเติม:
#### รวมกับ WHERE:
```sql
SELECT e.first_name, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.department_id
WHERE d.department_name = 'Sales';
```
- แสดงพนักงานในแผนก Sales เท่านั้น
- **ผลลัพธ์**:
  ```
  first_name | department_name
  ------------+----------------
  John       | Sales
  ```

#### รวมกับ GROUP BY และ HAVING:
```sql
SELECT d.department_name, COUNT(e.employee_id) AS employee_count
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.department_id
GROUP BY d.department_name
HAVING COUNT(e.employee_id) > 1;
```
- นับจำนวนพนักงานในแต่ละแผนก และแสดงเฉพาะแผนกที่มีพนักงานมากกว่า 1 คน
- **ผลลัพธ์**:
  ```
  department_name | employee_count
  ----------------+---------------
  Sales          | 2
  ```

### ข้อควรระวัง:
- **เงื่อนไข ON**:
  - ต้องระบุเงื่อนไขใน `ON` ให้ชัดเจนเพื่อเชื่อมโยงตาราง เช่น `e.department_id = d.department_id`
  - ถ้าเงื่อนไขไม่ชัดเจน อาจได้ผลลัพธ์แบบ Cartesian Product (ทุกแถวคูณกัน)
- **NULL**:
  - ค่า `NULL` ในคอลัมน์ที่ใช้ใน `ON` จะไม่ตรงกับค่าใด ๆ (รวมถึง `NULL` อื่น) ทำให้แถวนั้นอาจไม่ปรากฏใน `INNER JOIN`
- **ประสิทธิภาพ**:
  - การ JOIN ตารางขนาดใหญ่อาจช้า ควรใช้ดัชนี (Indexes) บนคอลัมน์ใน `ON` หรือ `WHERE`
  - `INNER JOIN` มักเร็วกว่า `OUTER JOIN` (LEFT, RIGHT, FULL) เพราะเลือกแถวน้อยกว่า
- **ชื่อคอลัมน์ซ้ำ**:
  - ถ้าตารางมีคอลัมน์ชื่อเดียวกัน (เช่น `department_id`) ควรใช้ Alias (เช่น `e.department_id`) เพื่อหลีกเลี่ยงความกำกวม
- **ระบบฐานข้อมูล**:
  - Oracle, MySQL, PostgreSQL, SQL Server รองรับ `INNER`, `LEFT`, `RIGHT`, `FULL JOIN` คล้ายกัน
  - **MySQL**: ไม่รองรับ `FULL JOIN` ในเวอร์ชันเก่า
  - **Oracle**: รองรับการเขียน JOIN แบบเก่า (เช่น `FROM table1, table2 WHERE table1.col = table2.col`) แต่แนะนำให้ใช้ไวยากรณ์ ANSI JOIN (`INNER JOIN`, `ON`)

### การใช้งานใน PL/SQL:
สามารถใช้ JOIN ใน PL/SQL เพื่อดึงข้อมูลจากหลายตาราง เช่น:
```sql
DECLARE
   CURSOR emp_dept IS
      SELECT e.first_name, d.department_name
      FROM employees e
      LEFT JOIN departments d
      ON e.department_id = d.department_id
      WHERE d.department_name IS NOT NULL
      ORDER BY e.first_name;
BEGIN
   FOR rec IN emp_dept LOOP
      DBMS_OUTPUT.PUT_LINE('Employee: ' || rec.first_name || 
                           ', Department: ' || rec.department_name);
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Employee: Jane, Department: IT
  Employee: John, Department: Sales
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการรายงานพนักงานและชื่อแผนก รวมถึงแผนกที่ไม่มีพนักงาน:
```sql
SELECT d.department_name,
       COUNT(e.employee_id) AS employee_count,
       SUM(e.salary) AS total_salary
FROM departments d
LEFT JOIN employees e
ON d.department_id = e.department_id
GROUP BY d.department_name
ORDER BY employee_count DESC;
```
- นับจำนวนพนักงานและผลรวมเงินเดือนในแต่ละแผนก รวมถึงแผนกที่ไม่มีพนักงาน
- **ผลลัพธ์**:
  ```
  department_name | employee_count | total_salary
  ----------------+---------------+--------------
  Sales          | 2             | 105000
  IT             | 1             | 60000
  HR             | 0             | NULL
  ```
- **อธิบาย**:
  - ใช้ `LEFT JOIN` เพื่อให้ทุกแผนกจาก `departments` ปรากฏ
  - แผนก HR ไม่มีพนักงาน ดังนั้น `employee_count = 0` และ `total_salary = NULL`

### การเปรียบเทียบประเภท JOIN:
| ประเภท JOIN     | แถวจากตารางซ้าย | แถวจากตารางขวา | แถวที่ไม่ตรงกัน |
|-----------------|------------------|-----------------|-----------------|
| **INNER JOIN**  | เฉพาะที่ตรงกัน  | เฉพาะที่ตรงกัน  | ไม่รวม          |
| **LEFT JOIN**   | ทุกแถว          | เฉพาะที่ตรงกัน  | `NULL` สำหรับขวา |
| **RIGHT JOIN**  | เฉพาะที่ตรงกัน  | ทุกแถว          | `NULL` สำหรับซ้าย |
| **FULL JOIN**   | ทุกแถว          | ทุกแถว          | `NULL` ในที่ไม่ตรง |
---

## การใช้ `ON` และ `USING`
ใน SQL คำสั่ง **ON** และ **USING** ใช้สำหรับกำหนด**เงื่อนไขการเชื่อมโยง** (join condition) เมื่อทำการ **JOIN** ตารางตั้งแต่สองตารางขึ้นไป เพื่อระบุว่าคอลัมน์ใดในตารางหนึ่งจะเชื่อมโยงกับคอลัมน์ใดในอีกตารางหนึ่ง ทั้งสองวิธีนี้ใช้ในบริบทของการรวมตาราง เช่น **INNER JOIN**, **LEFT JOIN**, **RIGHT JOIN**, และ **FULL JOIN** แต่มีความแตกต่างในด้านไวยากรณ์และการใช้งาน ต่อไปนี้คือคำอธิบายและตัวอย่างในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### 1. ON
- **คำอธิบาย**: ใช้สำหรับระบุเงื่อนไขการ JOIN อย่างชัดเจน โดยกำหนดความสัมพันธ์ระหว่างคอลัมน์ของตารางในรูปแบบ `table1.column = table2.column` หรือเงื่อนไขที่ซับซ้อนกว่านั้น
- **ลักษณะเด่น**:
    - มีความยืดหยุ่นสูง รองรับเงื่อนไขที่ซับซ้อน เช่น การเปรียบเทียบที่ไม่ใช่แค่ `=` หรือการใช้ฟังก์ชัน
    - ต้องระบุชื่อตารางหรือ Alias เพื่อหลีกเลี่ยงความกำกวมเมื่อคอลัมน์ชื่อซ้ำ
- **ไวยากรณ์**:
  ```sql
  SELECT columns
  FROM table1
  JOIN table2
  ON table1.column = table2.column [AND other_conditions];
  ```

### 2. USING
- **คำอธิบาย**: ใช้สำหรับระบุเงื่อนไขการ JOIN โดยอ้างอิงคอลัมน์ที่มี**ชื่อเหมือนกัน**ในทั้งสองตาราง โดยไม่ต้องระบุชื่อตารางหรือ Alias
- **ลักษณะเด่น**:
    - ใช้งานง่ายเมื่อคอลัมน์ที่ใช้ JOIN มีชื่อเดียวกันในทั้งสองตาราง
    - ไม่สามารถใช้กับเงื่อนไขที่ซับซ้อน (จำกัดเฉพาะการเปรียบเทียบ `=`)
    - คอลัมน์ที่ระบุใน `USING` จะปรากฏเพียงครั้งเดียวในผลลัพธ์ (ไม่ต้องระบุชื่อตาราง)
- **ไวยากรณ์**:
  ```sql
  SELECT columns
  FROM table1
  JOIN table2
  USING (column_name);
  ```

### ความแตกต่างระหว่าง ON และ USING
| คุณสมบัติ              | ON                                  | USING                              |
|------------------------|-------------------------------------|------------------------------------|
| **เงื่อนไข**          | รองรับเงื่อนไขซับซ้อน (>, <, LIKE, ฟังก์ชัน) | จำกัดเฉพาะ `=` และคอลัมน์ชื่อเดียวกัน |
| **ชื่อคอลัมน์**       | ต้องระบุชื่อตารางหรือ Alias (เช่น `table1.col`) | ไม่ต้องระบุชื่อตาราง แค่ชื่อคอลัมน์ |
| **ผลลัพธ์**           | คอลัมน์จากทั้งสองตารางปรากฏแยกกัน | คอลัมน์ที่ใช้ JOIN ปรากฏเพียงครั้งเดียว |
| **ความยืดหยุ่น**      | สูง ใช้ได้ทุกสถานการณ์             | จำกัด ใช้เมื่อคอลัมน์ชื่อเดียวกัน |
| **การใช้งานทั่วไป**    | ใช้เมื่อคอลัมน์ชื่อต่างกันหรือเงื่อนไขซับซ้อน | ใช้เมื่อคอลัมน์ชื่อเดียวกันและเงื่อนไขง่าย |

### ตัวอย่างการใช้งาน:
สมมติมีสองตาราง:
1. **employees**:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    dept_id NUMBER(5)
);
INSERT INTO employees VALUES (1, 'John', 10);
INSERT INTO employees VALUES (2, 'Jane', 20);
INSERT INTO employees VALUES (3, 'Alice', 30);
INSERT INTO employees VALUES (4, 'Bob', NULL);
COMMIT;
```

2. **departments**:
```sql
CREATE TABLE departments (
    dept_id NUMBER(5) PRIMARY KEY,
    department_name VARCHAR2(50)
);
INSERT INTO departments VALUES (10, 'Sales');
INSERT INTO departments VALUES (20, 'IT');
INSERT INTO departments VALUES (40, 'HR');
COMMIT;
```

#### 1. การใช้ ON
```sql
SELECT e.first_name, e.dept_id, d.department_name
FROM employees e
INNER JOIN departments d
ON e.dept_id = d.dept_id;
```
- **ผลลัพธ์**:
  ```
  first_name | dept_id | department_name
  ------------+---------+----------------
  John       | 10      | Sales
  Jane       | 20      | IT
  ```
- **อธิบาย**:
    - ใช้ `ON` เพื่อเชื่อมโยง `dept_id` จาก `employees` กับ `dept_id` จาก `departments`
    - ต้องระบุ Alias (`e.`, `d.`) เพื่อแยกคอลัมน์ `dept_id` จากทั้งสองตาราง
    - Alice (dept_id = 30) และ Bob (dept_id = NULL) ไม่ปรากฏ เพราะไม่มี `dept_id` ที่ตรงกัน

**ตัวอย่าง ON กับเงื่อนไขซับซ้อน**:
```sql
SELECT e.first_name, e.dept_id, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.dept_id = d.dept_id AND d.department_name LIKE 'S%';
```
- **ผลลัพธ์**:
  ```
  first_name | dept_id | department_name
  ------------+---------+----------------
  John       | 10      | Sales
  Jane       | 20      | NULL
  Alice      | 30      | NULL
  Bob        | NULL    | NULL
  ```
- **อธิบาย**:
    - เพิ่มเงื่อนไข `d.department_name LIKE 'S%'` ใน `ON` เพื่อเลือกเฉพาะแผนกที่ชื่อขึ้นต้นด้วย 'S'
    - ใช้ `LEFT JOIN` เพื่อให้ทุกพนักงานปรากฏ แม้ไม่มีแผนกที่ตรงกัน

#### 2. การใช้ USING
```sql
SELECT first_name, dept_id, department_name
FROM employees
INNER JOIN departments
USING (dept_id);
```
- **ผลลัพธ์**:
  ```
  first_name | dept_id | department_name
  ------------+---------+----------------
  John       | 10      | Sales
  Jane       | 20      | IT
  ```
- **อธิบาย**:
    - ใช้ `USING (dept_id)` เพราะทั้งสองตารางมีคอลัมน์ชื่อ `dept_id`
    - ไม่ต้องระบุชื่อตารางใน `USING` หรือใน `SELECT` สำหรับ `dept_id`
    - `dept_id` ปรากฏเพียงครั้งเดียวในผลลัพธ์ (ต่างจาก `ON` ที่อาจมี `e.dept_id` และ `d.dept_id`)

**ตัวอย่าง USING กับ LEFT JOIN**:
```sql
SELECT first_name, dept_id, department_name
FROM employees
LEFT JOIN departments
USING (dept_id);
```
- **ผลลัพธ์**:
  ```
  first_name | dept_id | department_name
  ------------+---------+----------------
  John       | 10      | Sales
  Jane       | 20      | IT
  Alice      | 30      | NULL
  Bob        | NULL    | NULL
  ```
- **อธิบาย**:
    - ใช้ `LEFT JOIN` เพื่อให้ทุกพนักงานปรากฏ
    - แผนกที่ไม่ตรง (เช่น dept_id = 30, NULL) จะมี `department_name` เป็น `NULL`

### ข้อควรระวัง:
- **ON**:
    - ต้องระบุชื่อตารางหรือ Alias เมื่อคอลัมน์ชื่อซ้ำในตารางที่ JOIN
    - เหมาะกับเงื่อนไขที่ซับซ้อน เช่น การ JOIN โดยใช้ฟังก์ชันหรือตัวดำเนินการอื่น (`>`, `<`, `LIKE`)
    - ผลลัพธ์อาจรวมคอลัมน์ที่ซ้ำ (เช่น `e.dept_id`, `d.dept_id`) ต้องระบุใน `SELECT` อย่างชัดเจน
- **USING**:
    - ใช้ได้เฉพาะเมื่อคอลัมน์ที่ใช้ JOIN มี**ชื่อเดียวกัน**ในทั้งสองตาราง
    - ไม่รองรับเงื่อนไขที่ซับซ้อน (เช่น `e.dept_id > d.dept_id` หรือ `UPPER(d.department_name) = 'SALES'`)
    - คอลัมน์ใน `USING` จะปรากฏเพียงครั้งเดียวในผลลัพธ์ ซึ่งอาจทำให้ต้องระวังเมื่ออ้างอิงในส่วนอื่น (เช่น `WHERE`, `ORDER BY`)
- **NULL**:
    - ค่า `NULL` ในคอลัมน์ที่ใช้ใน `ON` หรือ `USING` จะไม่ตรงกับค่าใด (รวมถึง `NULL` อื่น) ทำให้แถวนั้นอาจไม่ปรากฏใน `INNER JOIN`
- **ประสิทธิภาพ**:
    - ทั้ง `ON` และ `USING` มีประสิทธิภาพใกล้เคียงกันใน Oracle แต่ `ON` อาจเร็วกว่าในเงื่อนไขที่ซับซ้อนถ้ามีการใช้ดัชนี (Indexes) อย่างเหมาะสม
    - ควรสร้างดัชนีบนคอลัมน์ที่ใช้ใน `ON` หรือ `USING` เพื่อเพิ่มประสิทธิภาพ
- **ระบบฐานข้อมูล**:
    - **Oracle, PostgreSQL, SQL Server**: รองรับทั้ง `ON` และ `USING`
    - **MySQL**: รองรับทั้งสอง แต่ `USING` อาจใช้บ่อยในโค้ดที่ต้องการความกระชับ
    - **SQLite**: รองรับทั้งสอง แต่ `USING` อาจมีข้อจำกัดในบางกรณี
    - บางระบบอาจมีพฤติกรรมต่างกันเมื่อใช้ `USING` กับคอลัมน์ที่มีชื่อซ้ำในผลลัพธ์

### การใช้งานใน PL/SQL:
สามารถใช้ `ON` หรือ `USING` ใน PL/SQL เพื่อดึงข้อมูลจากหลายตาราง เช่น:
```sql
DECLARE
   CURSOR emp_dept IS
      SELECT first_name, dept_id, department_name
      FROM employees
      INNER JOIN departments
      USING (dept_id)
      ORDER BY first_name;
BEGIN
   FOR rec IN emp_dept LOOP
      DBMS_OUTPUT.PUT_LINE('Employee: ' || rec.first_name || 
                           ', Department: ' || rec.department_name);
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Employee: Jane, Department: IT
  Employee: John, Department: Sales
  ```

**ตัวอย่างใช้ ON**:
```sql
DECLARE
   CURSOR emp_dept IS
      SELECT e.first_name, d.department_name
      FROM employees e
      LEFT JOIN departments d
      ON e.dept_id = d.dept_id
      WHERE e.dept_id IS NOT NULL;
BEGIN
   FOR rec IN emp_dept LOOP
      DBMS_OUTPUT.PUT_LINE('Employee: ' || rec.first_name || 
                           ', Department: ' || NVL(rec.department_name, 'No Department'));
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Employee: John, Department: Sales
  Employee: Jane, Department: IT
  Employee: Alice, Department: No Department
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการรายงานพนักงานและชื่อแผนก รวมถึงพนักงานที่ไม่มีแผนกและแผนกที่ไม่มีพนักงาน:

#### ใช้ ON กับ FULL JOIN:
```sql
SELECT e.first_name, e.dept_id, d.department_name
FROM employees e
FULL JOIN departments d
ON e.dept_id = d.dept_id
ORDER BY e.first_name, d.department_name;
```
- **ผลลัพธ์**:
  ```
  first_name | dept_id | department_name
  ------------+---------+----------------
  Alice      | 30      | NULL
  Bob        | NULL    | NULL
  Jane       | 20      | IT
  John       | 10      | Sales
  NULL       | NULL    | HR
  ```
- **อธิบาย**:
    - ใช้ `ON` เพื่อเชื่อมโยง `dept_id`
    - `FULL JOIN` รวมทุกแถวจากทั้งสองตาราง
    - Alice (dept_id = 30) และ Bob (dept_id = NULL) ไม่มีแผนกที่ตรงกัน
    - แผนก HR ไม่มีพนักงานที่ตรงกัน

#### ใช้ USING กับ LEFT JOIN:
```sql
SELECT first_name, dept_id, department_name
FROM employees
LEFT JOIN departments
USING (dept_id)
WHERE department_name IS NOT NULL
ORDER BY first_name;
```
- **ผลลัพธ์**:
  ```
  first_name | dept_id | department_name
  ------------+---------+----------------
  Jane       | 20      | IT
  John       | 10      | Sales
  ```
- **อธิบาย**:
    - ใช้ `USING (dept_id)` เพราะคอลัมน์ชื่อเดียวกัน
    - `LEFT JOIN` รวมทุกพนักงาน แต่ `WHERE department_name IS NOT NULL` กรองเฉพาะพนักงานที่มีแผนก

### การรวมกับฟังก์ชันอื่น:
#### ใช้ ON กับ GROUP BY:
```sql
SELECT d.department_name, COUNT(e.employee_id) AS employee_count
FROM employees e
LEFT JOIN departments d
ON e.dept_id = d.dept_id
GROUP BY d.department_name
HAVING COUNT(e.employee_id) > 0
ORDER BY employee_count DESC;
```
- นับจำนวนพนักงานในแต่ละแผนกที่มีพนักงานอย่างน้อย 1 คน
- **ผลลัพธ์**:
  ```
  department_name | employee_count
  ----------------+---------------
  Sales          | 2
  IT             | 1
  ```

#### ใช้ USING กับ CASE:
```sql
SELECT dept_id,
       COUNT(*) AS employee_count,
       SUM(CASE WHEN first_name LIKE 'J%' THEN 1 ELSE 0 END) AS j_name_count
FROM employees
INNER JOIN departments
USING (dept_id)
GROUP BY dept_id;
```
- นับจำนวนพนักงานและจำนวนพนักงานที่ชื่อขึ้นต้นด้วย 'J' ในแต่ละแผนก
- **ผลลัพธ์**:
  ```
  dept_id | employee_count | j_name_count
  --------+---------------+-------------
  10      | 1             | 1
  20      | 1             | 1
  ```
---

## การ `JOIN` มากกว่า 2 ตาราง
ใน SQL การ **JOIN** สามารถใช้เพื่อ**รวมข้อมูลจากตารางมากกว่าสองตาราง**ได้โดยการเชื่อมโยงตารางแต่ละคู่ด้วยเงื่อนไขการ JOIN (ใช้ `ON` หรือ `USING`) และสามารถใช้ประเภท JOIN ต่าง ๆ เช่น **INNER JOIN**, **LEFT JOIN**, **RIGHT JOIN**, หรือ **FULL JOIN** ร่วมกันได้ การ JOIN หลายตารางมีประโยชน์เมื่อข้อมูลกระจายอยู่ในหลายตาราง เช่น การรวมข้อมูลพนักงาน, แผนก, และที่ตั้งของสำนักงาน ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### ไวยากรณ์พื้นฐาน:
```sql
SELECT columns
FROM table1
[INNER | LEFT | RIGHT | FULL] JOIN table2
    ON condition1
[INNER | LEFT | RIGHT | FULL] JOIN table3
    ON condition2
[INNER | LEFT | RIGHT | FULL] JOIN table4
    ON condition3
...
[WHERE condition]
[GROUP BY columns]
[HAVING condition]
[ORDER BY columns];
```

- **table1, table2, table3, ...**: ตารางที่ต้องการรวม
- **ON condition**: เงื่อนไขที่กำหนดความสัมพันธ์ระหว่างตาราง (เช่น `table1.column = table2.column`)
- **columns**: คอลัมน์ที่ต้องการเลือกจากตารางทั้งหมด
- **ประเภท JOIN**: สามารถผสมประเภท JOIN ได้ (เช่น ใช้ `INNER JOIN` สำหรับตารางหนึ่งและ `LEFT JOIN` สำหรับอีกตาราง)

### ลำดับการ JOIN:
- SQL ประมวลผล JOIN จาก**ซ้ายไปขวา**ตามลำดับที่ระบุ
- สามารถใช้เครื่องหมายวงเล็บเพื่อควบคุมลำดับการ JOIN ในบางระบบ (เช่น PostgreSQL) แต่ใน Oracle มักไม่จำเป็น
- เงื่อนไขใน `ON` ควรเชื่อมโยงตารางอย่างชัดเจนเพื่อหลีกเลี่ยงผลลัพธ์แบบ Cartesian Product (ทุกแถวคูณกัน)

### ตัวอย่างการใช้งาน:
สมมติมีสามตาราง:
1. **employees**:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    dept_id NUMBER(5),
    location_id NUMBER(5)
);
INSERT INTO employees VALUES (1, 'John', 10, 100);
INSERT INTO employees VALUES (2, 'Jane', 20, 200);
INSERT INTO employees VALUES (3, 'Alice', 30, 100);
INSERT INTO employees VALUES (4, 'Bob', NULL, NULL);
COMMIT;
```

2. **departments**:
```sql
CREATE TABLE departments (
    dept_id NUMBER(5) PRIMARY KEY,
    department_name VARCHAR2(50)
);
INSERT INTO departments VALUES (10, 'Sales');
INSERT INTO departments VALUES (20, 'IT');
INSERT INTO departments VALUES (40, 'HR');
COMMIT;
```

3. **locations**:
```sql
CREATE TABLE locations (
    location_id NUMBER(5) PRIMARY KEY,
    city VARCHAR2(50)
);
INSERT INTO locations VALUES (100, 'New York');
INSERT INTO locations VALUES (200, 'London');
INSERT INTO locations VALUES (300, 'Tokyo');
COMMIT;
```

#### 1. INNER JOIN สามตาราง
- **คำอธิบาย**: รวมข้อมูลพนักงาน, แผนก, และที่ตั้ง โดยเลือกเฉพาะแถวที่ตรงกันในทั้งสามตาราง
- **ตัวอย่าง**:
  ```sql
  SELECT e.first_name, d.department_name, l.city
  FROM employees e
  INNER JOIN departments d
      ON e.dept_id = d.dept_id
  INNER JOIN locations l
      ON e.location_id = l.location_id;
  ```
- **ผลลัพธ์**:
  ```
  first_name | department_name | city
  ------------+----------------+---------
  John       | Sales          | New York
  Jane       | IT             | London
  ```
- **อธิบาย**:
    - `e.dept_id = d.dept_id`: เชื่อมโยง `employees` กับ `departments`
    - `e.location_id = l.location_id`: เชื่อมโยง `employees` กับ `locations`
    - Alice (dept_id = 30) ไม่ปรากฏ เพราะไม่มี `dept_id = 30` ใน `departments`
    - Bob (dept_id = NULL, location_id = NULL) ไม่ปรากฏ เพราะไม่มีข้อมูลที่ตรงกัน
    - แผนก HR และเมือง Tokyo ไม่ปรากฏ เพราะไม่มีพนักงานที่เชื่อมโยง

#### 2. LEFT JOIN สามตาราง
- **คำอธิบาย**: รวมทุกแถวจาก `employees` และแถวที่ตรงกันจาก `departments` และ `locations` โดยคืน `NULL` สำหรับคอลัมน์ที่ไม่ตรงกัน
- **ตัวอย่าง**:
  ```sql
  SELECT e.first_name, d.department_name, l.city
  FROM employees e
  LEFT JOIN departments d
      ON e.dept_id = d.dept_id
  LEFT JOIN locations l
      ON e.location_id = l.location_id;
  ```
- **ผลลัพธ์**:
  ```
  first_name | department_name | city
  ------------+----------------+---------
  John       | Sales          | New York
  Jane       | IT             | London
  Alice      | NULL           | New York
  Bob        | NULL           | NULL
  ```
- **อธิบาย**:
    - ทุกพนักงานจาก `employees` ปรากฏในผลลัพธ์
    - Alice (dept_id = 30) ไม่มีแผนกที่ตรงกัน ดังนั้น `department_name = NULL`
    - Bob (dept_id = NULL, location_id = NULL) ไม่มีข้อมูลที่ตรงกัน ดังนั้น `department_name` และ `city` เป็น `NULL`

#### 3. ผสม INNER JOIN และ LEFT JOIN
- **คำอธิบาย**: ใช้ `INNER JOIN` สำหรับตารางที่มีความสัมพันธ์แน่นอน และ `LEFT JOIN` สำหรับตารางที่อาจไม่มีข้อมูลตรงกัน
- **ตัวอย่าง**:
  ```sql
  SELECT e.first_name, d.department_name, l.city
  FROM employees e
  INNER JOIN departments d
      ON e.dept_id = d.dept_id
  LEFT JOIN locations l
      ON e.location_id = l.location_id;
  ```
- **ผลลัพธ์**:
  ```
  first_name | department_name | city
  ------------+----------------+---------
  John       | Sales          | New York
  Jane       | IT             | London
  ```
- **อธิบาย**:
    - `INNER JOIN` กับ `departments` กรองเฉพาะพนักงานที่มี `dept_id` ตรงกับ `departments`
    - `LEFT JOIN` กับ `locations` รวมทุกพนักงานที่ผ่านการกรองจาก `INNER JOIN` และคืน `NULL` ถ้าไม่มี `location_id` ที่ตรงกัน
    - Alice และ Bob ไม่ปรากฏ เพราะไม่มี `dept_id` ที่ตรงกับ `departments`

#### 4. FULL JOIN สามตาราง
- **คำอธิบาย**: รวมทุกแถวจากทั้งสามตาราง โดยคืน `NULL` ในคอลัมน์ที่ไม่มีแถวตรงกัน
- **ตัวอย่าง**:
  ```sql
  SELECT e.first_name, d.department_name, l.city
  FROM employees e
  FULL JOIN departments d
      ON e.dept_id = d.dept_id
  FULL JOIN locations l
      ON e.location_id = l.location_id;
  ```
- **ผลลัพธ์** (อาจแตกต่างกันขึ้นอยู่กับระบบฐานข้อมูล):
  ```
  first_name | department_name | city
  ------------+----------------+---------
  John       | Sales          | New York
  Jane       | IT             | London
  Alice      | NULL           | New York
  Bob        | NULL           | NULL
  NULL       | HR             | NULL
  NULL       | NULL           | Tokyo
  ```
- **อธิบาย**:
    - รวมทุกแถวจาก `employees`, `departments`, และ `locations`
    - Alice (dept_id = 30) ไม่มีแผนกที่ตรงกัน
    - แผนก HR และเมือง Tokyo ไม่มีพนักงานที่เชื่อมโยง
    - **หมายเหตุ**: การใช้ `FULL JOIN` หลายตารางอาจซับซ้อนและขึ้นอยู่กับลำดับการ JOIN ควรทดสอบผลลัพธ์ให้แน่ใจ

#### 5. รวมกับ USING:
- **คำอธิบาย**: ใช้ `USING` เมื่อคอลัมน์ที่ใช้ JOIN มีชื่อเดียวกันในตาราง
- **ตัวอย่าง**:
  ```sql
  SELECT first_name, department_name, city
  FROM employees
  INNER JOIN departments
      USING (dept_id)
  INNER JOIN locations
      USING (location_id);
  ```
- **ผลลัพธ์**:
  ```
  first_name | department_name | city
  ------------+----------------+---------
  John       | Sales          | New York
  Jane       | IT             | London
  ```
- **อธิบาย**:
    - ใช้ `USING` เพราะ `dept_id` และ `location_id` เป็นชื่อคอลัมน์เดียวกันในตารางที่เกี่ยวข้อง
    - คอลัมน์ `dept_id` และ `location_id` ปรากฏเพียงครั้งเดียวในผลลัพธ์

#### 6. รวมกับ GROUP BY และ HAVING:
- **คำอธิบาย**: รวมสามตารางและวิเคราะห์ข้อมูลด้วยการจัดกลุ่ม
- **ตัวอย่าง**:
  ```sql
  SELECT d.department_name, l.city, COUNT(e.employee_id) AS employee_count
  FROM employees e
  LEFT JOIN departments d
      ON e.dept_id = d.dept_id
  LEFT JOIN locations l
      ON e.location_id = l.location_id
  GROUP BY d.department_name, l.city
  HAVING COUNT(e.employee_id) > 0
  ORDER BY employee_count DESC;
  ```
- **ผลลัพธ์**:
  ```
  department_name | city      | employee_count
  ----------------+-----------+---------------
  Sales          | New York  | 2
  IT             | London    | 1
  NULL           | New York  | 1
  ```
- **อธิบาย**:
    - ใช้ `LEFT JOIN` เพื่อรวมทุกพนักงาน
    - `GROUP BY` จัดกลุ่มตามชื่อแผนกและเมือง
    - `HAVING` กรองเฉพาะกลุ่มที่มีพนักงานอย่างน้อย 1 คน
    - Alice (dept_id = 30) ทำให้มีกลุ่มที่มี `department_name = NULL`

### ข้อควรระวัง:
- **เงื่อนไข ON/USING**:
    - ต้องระบุเงื่อนไขให้ชัดเจนเพื่อเชื่อมโยงตารางทั้งหมด มิฉะนั้นอาจได้ผลลัพธ์แบบ Cartesian Product
    - เมื่อใช้ `USING` คอลัมน์ต้องมีชื่อเดียวกันในตารางที่เกี่ยวข้อง
- **ลำดับการ JOIN**:
    - ลำดับการ JOIN มีผลต่อผลลัพธ์ โดยเฉพาะเมื่อผสม `INNER` และ `OUTER JOIN`
    - ควรทดสอบและตรวจสอบผลลัพธ์เมื่อใช้ `FULL JOIN` กับหลายตาราง
- **NULL**:
    - ค่า `NULL` ในคอลัมน์ที่ใช้ใน `ON` หรือ `USING` จะไม่ตรงกับค่าใด ทำให้แถวนั้นอาจไม่ปรากฏใน `INNER JOIN`
- **ประสิทธิภาพ**:
    - การ JOIN หลายตารางอาจใช้ทรัพยากรเยอะ โดยเฉพาะในตารางขนาดใหญ่
    - ควรสร้างดัชนี (Indexes) บนคอลัมน์ที่ใช้ใน `ON` หรือ `USING`
    - ใช้ `WHERE` เพื่อกรองแถวก่อน JOIN ถ้าเป็นไปได้
- **ระบบฐานข้อมูล**:
    - Oracle, MySQL, PostgreSQL, SQL Server รองรับการ JOIN หลายตาราง
    - **MySQL**: ไม่รองรับ `FULL JOIN` ในบางเวอร์ชัน ต้องใช้การรวม `LEFT JOIN` และ `RIGHT JOIN` ด้วย `UNION`
    - **PostgreSQL**: รองรับการควบคุมลำดับด้วยวงเล็บ (เช่น `(table1 JOIN table2) JOIN table3`)
    - **Oracle**: รองรับไวยากรณ์เก่า (เช่น `FROM table1, table2, table3 WHERE ...`) แต่แนะนำให้ใช้ ANSI JOIN

### การใช้งานใน PL/SQL:
สามารถใช้ JOIN หลายตารางใน PL/SQL เพื่อดึงข้อมูล เช่น:
```sql
DECLARE
   CURSOR emp_dept_loc IS
      SELECT e.first_name, d.department_name, l.city
      FROM employees e
      LEFT JOIN departments d
          ON e.dept_id = d.dept_id
      LEFT JOIN locations l
          ON e.location_id = l.location_id
      WHERE l.city IS NOT NULL
      ORDER BY e.first_name;
BEGIN
   FOR rec IN emp_dept_loc LOOP
      DBMS_OUTPUT.PUT_LINE('Employee: ' || rec.first_name || 
                           ', Department: ' || NVL(rec.department_name, 'None') || 
                           ', City: ' || rec.city);
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Employee: Alice, Department: None, City: New York
  Employee: Jane, Department: IT, City: London
  Employee: John, Department: Sales, City: New York
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการรายงานพนักงาน, ชื่อแผนก, และเมือง โดยรวมทุกพนักงานและแสดงจำนวนพนักงานในแต่ละเมือง:
```sql
SELECT l.city, d.department_name, COUNT(e.employee_id) AS employee_count
FROM employees e
LEFT JOIN departments d
    ON e.dept_id = d.dept_id
LEFT JOIN locations l
    ON e.location_id = l.location_id
GROUP BY l.city, d.department_name
HAVING COUNT(e.employee_id) > 0
ORDER BY employee_count DESC, l.city, d.department_name;
```
- **ผลลัพธ์**:
  ```
  city      | department_name | employee_count
  -----------+----------------+---------------
  New York  | Sales          | 2
  London    | IT             | 1
  New York  | NULL           | 1
  ```
- **อธิบาย**:
    - ใช้ `LEFT JOIN` เพื่อรวมทุกพนักงาน
    - `GROUP BY` จัดกลุ่มตามเมืองและชื่อแผนก
    - `HAVING` กรองเฉพาะกลุ่มที่มีพนักงาน
    - Bob (location_id = NULL) ไม่ปรากฏเพราะไม่มีเมืองที่ตรงกัน
    - Tokyo และ HR ไม่ปรากฏเพราะไม่มีพนักงานที่เชื่อมโยง

### การรวมกับฟังก์ชันอื่น:
#### ใช้ CASE และ JOIN สามตาราง:
```sql
SELECT l.city,
       COUNT(*) AS total_employees,
       SUM(CASE WHEN d.department_name = 'Sales' THEN 1 ELSE 0 END) AS sales_employees
FROM employees e
LEFT JOIN departments d
    ON e.dept_id = d.dept_id
LEFT JOIN locations l
    ON e.location_id = l.location_id
GROUP BY l.city
HAVING COUNT(*) > 0;
```
- นับจำนวนพนักงานทั้งหมดและจำนวนพนักงานในแผนก Sales ในแต่ละเมือง
- **ผลลัพธ์**:
  ```
  city      | total_employees | sales_employees
  -----------+---------------+----------------
  New York  | 3             | 2
  London    | 1             | 0
  ```
---
