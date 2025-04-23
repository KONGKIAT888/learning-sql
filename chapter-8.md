# บทที่ 8: Subquery และ WITH Clause
## การใช้ `Subquery` ใน `WHERE`, `SELECT`, `FROM`
ใน SQL **Subquery** (หรือ Query ย่อย) คือคำสั่ง `SELECT` ที่ฝังอยู่ในคำสั่ง SQL อื่น เพื่อใช้ผลลัพธ์จาก Subquery ในการประมวลผลต่อ Subquery สามารถใช้ในส่วนต่าง ๆ ของคำสั่ง SQL เช่น **WHERE**, **SELECT**, และ **FROM** เพื่อกรองข้อมูล, คำนวณค่า, หรือสร้างตารางชั่วคราว Subquery มีประโยชน์เมื่อต้องการแยกตรรกะการค้นหาที่ซับซ้อนหรือต้องการผลลัพธ์จากตารางอื่นในการประมวลผล ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### ประเภทของ Subquery
1. **Single-Row Subquery**: คืนค่าเพียงแถวเดียวและคอลัมน์เดียว (ใช้กับตัวดำเนินการ เช่น `=`, `>`, `<`)
2. **Multi-Row Subquery**: คืนค่าหลายแถว (ใช้กับตัวดำเนินการ เช่น `IN`, `ANY`, `ALL`)
3. **Correlated Subquery**: Subquery ที่อ้างอิงค่าจากตารางใน Query หลัก (ประมวลผลแถวต่อแถว)
4. **Nested Subquery**: Subquery ที่ซ้อนกันหลายชั้น

### การใช้งาน Subquery
Subquery สามารถปรากฏในส่วนต่าง ๆ ของคำสั่ง SQL ได้ดังนี้:

### 1. Subquery ใน WHERE
- **คำอธิบาย**: ใช้ Subquery ในเงื่อนไข `WHERE` เพื่อกรองแถวโดยอ้างอิงผลลัพธ์จาก Query ย่อย
- **ลักษณะเด่น**:
    - มักใช้เพื่อเปรียบเทียบกับค่าที่ได้จาก Subquery
    - สามารถเป็น Single-Row หรือ Multi-Row Subquery
- **ไวยากรณ์**:
  ```sql
  SELECT columns
  FROM table
  WHERE column operator (SELECT column FROM table WHERE condition);
  ```

#### ตัวอย่าง:
สมมติมีตาราง `employees`:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    salary NUMBER(10,2),
    department_id NUMBER(5)
);
INSERT INTO employees VALUES (1, 'John', 50000, 10);
INSERT INTO employees VALUES (2, 'Jane', 60000, 20);
INSERT INTO employees VALUES (3, 'Alice', 55000, 10);
INSERT INTO employees VALUES (4, 'Bob', 65000, 20);
INSERT INTO employees VALUES (5, 'Carol', 52000, 10);
COMMIT;
```

##### Single-Row Subquery:
```sql
SELECT first_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```
- เลือกพนักงานที่มีเงินเดือนมากกว่าค่าเฉลี่ยเงินเดือน
- **ผลลัพธ์**:
  ```
  first_name | salary
  ------------+--------
  Jane       | 60000
  Bob        | 65000
  ```
- **อธิบาย**:
    - Subquery `(SELECT AVG(salary) FROM employees)` คืนค่าเฉลี่ยเงินเดือน (56400)
    - `WHERE salary > 56400` กรองพนักงานที่มีเงินเดือนมากกว่า 56400

##### Multi-Row Subquery:
```sql
SELECT first_name, department_id
FROM employees
WHERE department_id IN (SELECT department_id FROM employees GROUP BY department_id HAVING COUNT(*) > 2);
```
- เลือกพนักงานในแผนกที่มีพนักงานมากกว่า 2 คน
- **ผลลัพธ์**:
  ```
  first_name | department_id
  ------------+---------------
  John       | 10
  Alice      | 10
  Carol      | 10
  ```
- **อธิบาย**:
    - Subquery คืน `department_id` ที่มีจำนวนพนักงานมากกว่า 2 (คือ 10)
    - `WHERE department_id IN (10)` กรองพนักงานในแผนก 10

##### Correlated Subquery:
```sql
SELECT first_name, salary, department_id
FROM employees e1
WHERE salary = (SELECT MAX(salary) FROM employees e2 WHERE e2.department_id = e1.department_id);
```
- เลือกพนักงานที่มีเงินเดือนสูงสุดในแต่ละแผนก
- **ผลลัพธ์**:
  ```
  first_name | salary | department_id
  ------------+--------+---------------
  Alice      | 55000  | 10
  Bob        | 65000  | 20
  ```
- **อธิบาย**:
    - Subquery เปรียบเทียบ `department_id` ของแถวใน Query หลัก (`e1`) กับ `department_id` ใน Subquery (`e2`)
    - คืนค่าเงินเดือนสูงสุดของแต่ละแผนก และกรองพนักงานที่มีเงินเดือนตรงกับค่านั้น

### 2. Subquery ใน SELECT
- **คำอธิบาย**: ใช้ Subquery ในส่วน `SELECT` เพื่อคำนวณค่าในแต่ละแถวของผลลัพธ์
- **ลักษณะเด่น**:
    - มักเป็น Single-Row หรือ Correlated Subquery
    - ใช้เพื่อเพิ่มคอลัมน์ที่คำนวณจากตารางอื่น
- **ไวยากรณ์**:
  ```sql
  SELECT column1, (SELECT column FROM table WHERE condition) AS alias
  FROM table;
  ```

#### ตัวอย่าง:
##### Single-Row Subquery:
```sql
SELECT first_name, salary, 
       (SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees;
```
- แสดงเงินเดือนของพนักงานพร้อมค่าเฉลี่ยเงินเดือนทั้งหมด
- **ผลลัพธ์**:
  ```
  first_name | salary | avg_salary
  ------------+--------+------------
  John       | 50000  | 56400
  Jane       | 60000  | 56400
  Alice      | 55000  | 56400
  Bob        | 65000  | 56400
  Carol      | 52000  | 56400
  ```
- **อธิบาย**:
    - Subquery `(SELECT AVG(salary) FROM employees)` คืนค่าเฉลี่ยเงินเดือน (56400)
    - ค่าเฉลี่ยนี้แสดงในทุกแถวของผลลัพธ์

##### Correlated Subquery:
```sql
SELECT first_name, salary, department_id,
       (SELECT COUNT(*) FROM employees e2 WHERE e2.department_id = e1.department_id) AS dept_count
FROM employees e1;
```
- แสดงจำนวนพนักงานในแผนกของพนักงานแต่ละคน
- **ผลลัพธ์**:
  ```
  first_name | salary | department_id | dept_count
  ------------+--------+---------------+------------
  John       | 50000  | 10            | 3
  Jane       | 60000  | 20            | 2
  Alice      | 55000  | 10            | 3
  Bob        | 65000  | 20            | 2
  Carol      | 52000  | 10            | 3
  ```
- **อธิบาย**:
    - Subquery นับจำนวนพนักงานใน `department_id` เดียวกับพนักงานใน Query หลัก

### 3. Subquery ใน FROM (Inline View)
- **คำอธิบาย**: ใช้ Subquery ในส่วน `FROM` เพื่อสร้าง**ตารางชั่วคราว** (เรียกว่า Inline View) ที่ Query หลักจะใช้ต่อ
- **ลักษณะเด่น**:
    - เหมาะสำหรับการคำนวณหรือจัดกลุ่มข้อมูลก่อนนำไปใช้ใน Query หลัก
    - Subquery ต้องมี Alias เพื่ออ้างอิงใน Query หลัก
- **ไวยากรณ์**:
  ```sql
  SELECT columns
  FROM (SELECT columns FROM table WHERE condition) alias
  [WHERE condition];
  ```

#### ตัวอย่าง:
```sql
SELECT dept_summary.department_id, dept_summary.avg_salary
FROM (SELECT department_id, AVG(salary) AS avg_salary
      FROM employees
      GROUP BY department_id) dept_summary
WHERE dept_summary.avg_salary > 55000;
```
- คำนวณค่าเฉลี่ยเงินเดือนในแต่ละแผนก และเลือกเฉพาะแผนกที่มีค่าเฉลี่ยมากกว่า 55,000
- **ผลลัพธ์**:
  ```
  department_id | avg_salary
  --------------+------------
  20            | 62500
  ```
- **อธิบาย**:
    - Subquery `(SELECT department_id, AVG(salary) ...)` สร้างตารางชั่วคราวชื่อ `dept_summary`
    - Query หลักกรองแถวที่มี `avg_salary > 55000`

##### รวมกับ JOIN:
```sql
SELECT e.first_name, e.salary, d.department_name
FROM employees e
JOIN (SELECT department_id, AVG(salary) AS avg_salary
      FROM employees
      GROUP BY department_id
      HAVING AVG(salary) > 55000) dept_avg
ON e.department_id = dept_avg.department_id
JOIN departments d
ON e.department_id = d.dept_id;
```
- เลือกพนักงานในแผนกที่มีค่าเฉลี่ยเงินเดือนมากกว่า 55,000 พร้อมชื่อแผนก
- **ผลลัพธ์**:
  ```
  first_name | salary | department_name
  ------------+--------+----------------
  Jane       | 60000  | IT
  Bob        | 65000  | IT
  ```
- **อธิบาย**:
    - Subquery สร้างตารางชั่วคราวที่มี `department_id` และ `avg_salary` สำหรับแผนกที่มีค่าเฉลี่ยเงินเดือน > 55,000
    - JOIN กับ `employees` และ `departments` เพื่อดึงข้อมูลพนักงานและชื่อแผนก

### ข้อควรระวัง:
- **ประสิทธิภาพ**:
    - Subquery โดยเฉพาะ Correlated Subquery อาจช้าในข้อมูลขนาดใหญ่ เพราะประมวลผลแถวต่อแถว
    - ควรพิจารณาใช้ **JOIN** หรือ **WITH Clause (CTE)** แทน Subquery ในบางกรณีเพื่อเพิ่มประสิทธิภาพ
    - ตัวอย่างการใช้ CTE แทน Inline View:
      ```sql
      WITH dept_summary AS (
          SELECT department_id, AVG(salary) AS avg_salary
          FROM employees
          GROUP BY department_id
      )
      SELECT department_id, avg_salary
      FROM dept_summary
      WHERE avg_salary > 55000;
      ```
- **Single-Row vs Multi-Row**:
    - Single-Row Subquery ต้องคืนค่าแถวเดียว มิฉะนั้นจะเกิดข้อผิดพลาด (เช่น `ORA-01427: single-row subquery returns more than one row`)
    - Multi-Row Subquery ต้องใช้ตัวดำเนินการเช่น `IN`, `ANY`, `ALL`
- **NULL**:
    - ถ้า Subquery คืน `NULL` อาจส่งผลต่อเงื่อนไขใน `WHERE` (เช่น `column = NULL` ไม่ทำงาน ต้องใช้ `IS NULL`)
- **ระบบฐานข้อมูล**:
    - Oracle, MySQL, PostgreSQL, SQL Server รองรับ Subquery คล้ายกัน
    - **MySQL**: ในเวอร์ชันเก่าอาจมีข้อจำกัดด้านประสิทธิภาพสำหรับ Correlated Subquery
    - **PostgreSQL**: รองรับ Subquery และ CTE ได้ดี และมักแนะนำให้ใช้ CTE เพื่อความชัดเจน
    - **SQL Server**: คล้าย Oracle แต่ควรระวังการจัดการ `NULL` ใน Subquery

### การใช้งานใน PL/SQL:
สามารถใช้ Subquery ใน PL/SQL เพื่อดึงข้อมูล เช่น:
```sql
DECLARE
   v_first_name VARCHAR2(50);
   v_salary NUMBER;
   CURSOR high_salary_employees IS
      SELECT first_name, salary
      FROM employees
      WHERE salary > (SELECT AVG(salary) FROM employees)
      ORDER BY salary DESC;
BEGIN
   FOR emp IN high_salary_employees LOOP
      DBMS_OUTPUT.PUT_LINE('Employee: ' || emp.first_name || 
                           ', Salary: ' || emp.salary);
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Employee: Bob, Salary: 65000
  Employee: Jane, Salary: 60000
  ```

**ตัวอย่าง Correlated Subquery**:
```sql
DECLARE
   CURSOR max_salary_per_dept IS
      SELECT first_name, salary, department_id
      FROM employees e1
      WHERE salary = (SELECT MAX(salary) FROM employees e2 
                      WHERE e2.department_id = e1.department_id);
BEGIN
   FOR emp IN max_salary_per_dept LOOP
      DBMS_OUTPUT.PUT_LINE('Employee: ' || emp.first_name || 
                           ', Salary: ' || emp.salary || 
                           ', Department: ' || emp.department_id);
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Employee: Alice, Salary: 55000, Department: 10
  Employee: Bob, Salary: 65000, Department: 20
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการรายงานพนักงานที่มีเงินเดือนสูงกว่าค่าเฉลี่ยของแผนกตนเอง:

#### Subquery ใน WHERE (Correlated):
```sql
SELECT e1.first_name, e1.salary, e1.department_id
FROM employees e1
WHERE e1.salary > (SELECT AVG(salary) FROM employees e2 
                   WHERE e2.department_id = e1.department_id);
```
- **ผลลัพธ์**:
  ```
  first_name | salary | department_id
  ------------+--------+---------------
  Alice      | 55000  | 10
  Bob        | 65000  | 20
  ```
- **อธิบาย**:
    - Subquery คำนวณค่าเฉลี่ยเงินเดือนของแผนกที่พนักงานอยู่ในแต่ละแถว
    - เปรียบเทียบ `salary` ของพนักงานกับค่าเฉลี่ยนั้น

#### Subquery ใน FROM (Inline View):
```sql
SELECT e.first_name, e.salary, e.department_id, dept_avg.avg_salary
FROM employees e
JOIN (SELECT department_id, AVG(salary) AS avg_salary
      FROM employees
      GROUP BY department_id) dept_avg
ON e.department_id = dept_avg.department_id
WHERE e.salary > dept_avg.avg_salary;
```
- **ผลลัพธ์**: เดียวกับด้านบน
- **อธิบาย**:
    - Subquery สร้างตารางชั่วคราวที่มี `department_id` และ `avg_salary`
    - JOIN กับ `employees` และกรองพนักงานที่มีเงินเดือนมากกว่าค่าเฉลี่ยของแผนก

### การเปรียบเทียบ Subquery กับ JOIN:
- **Subquery**:
    - เหมาะสำหรับตรรกะที่ซับซ้อนหรือต้องการแยกขั้นตอนการคำนวณ
    - อาจช้ากว่า JOIN ในบางกรณี โดยเฉพาะ Correlated Subquery
- **JOIN**:
    - มักเร็วกว่าและเหมาะสำหรับการรวมข้อมูลจากหลายตาราง
    - อ่านง่ายกว่าเมื่อต้องการรวมข้อมูลจำนวนมาก
- **ตัวอย่างการแปลง Subquery เป็น JOIN**:
  แทนที่:
  ```sql
  SELECT first_name, salary
  FROM employees
  WHERE salary > (SELECT AVG(salary) FROM employees);
  ```
  เป็น:
  ```sql
  SELECT e.first_name, e.salary
  FROM employees e
  CROSS JOIN (SELECT AVG(salary) AS avg_salary FROM employees) avg_sal
  WHERE e.salary > avg_sal.avg_salary;
  ```
---

## `IN`, `EXISTS`, `NOT EXISTS`
ใน SQL ตัวดำเนินการ **IN**, **EXISTS**, และ **NOT EXISTS** ใช้สำหรับตรวจสอบการมีอยู่ของข้อมูลในเงื่อนไข โดยมักใช้ร่วมกับ **Subquery** เพื่อกรองแถวตามผลลัพธ์จาก Query ย่อย ตัวดำเนินการเหล่านี้มีจุดประสงค์และพฤติกรรมที่แตกต่างกัน โดยเฉพาะในแง่ของประสิทธิภาพและการใช้งาน ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### 1. IN
- **คำอธิบาย**: ตรวจสอบว่าค่าของคอลัมน์อยู่ใน**ชุดค่าที่ระบุ** (อาจเป็นรายการค่าหรือผลลัพธ์จาก Subquery)
- **ลักษณะเด่น**:
    - ใช้กับ **Multi-Row Subquery** ที่คืนค่าหลายแถว
    - เหมาะสำหรับตรวจสอบว่าค่าอยู่ในรายการหรือผลลัพธ์ของ Subquery
    - คล้ายกับการใช้ `OR` หลายเงื่อนไข
- **ไวยากรณ์**:
  ```sql
  SELECT columns
  FROM table
  WHERE column IN (value1, value2, ...) 
  -- หรือ
  WHERE column IN (SELECT column FROM table WHERE condition);
  ```

#### ตัวอย่าง:
สมมติมีสองตาราง:
1. **employees**:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    salary NUMBER(10,2),
    department_id NUMBER(5)
);
INSERT INTO employees VALUES (1, 'John', 50000, 10);
INSERT INTO employees VALUES (2, 'Jane', 60000, 20);
INSERT INTO employees VALUES (3, 'Alice', 55000, 10);
INSERT INTO employees VALUES (4, 'Bob', 65000, 20);
INSERT INTO employees VALUES (5, 'Carol', 52000, 30);
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

```sql
SELECT first_name, department_id
FROM employees
WHERE department_id IN (SELECT department_id FROM departments);
```
- เลือกพนักงานที่มี `department_id` อยู่ในตาราง `departments`
- **ผลลัพธ์**:
  ```
  first_name | department_id
  ------------+---------------
  John       | 10
  Jane       | 20
  Alice      | 10
  Bob        | 20
  ```
- **อธิบาย**:
    - Subquery คืน `department_id` จาก `departments` (10, 20, 40)
    - `WHERE department_id IN (10, 20, 40)` กรองพนักงานที่มี `department_id` เป็น 10 หรือ 20
    - Carol (department_id = 30) ไม่ปรากฏ เพราะ 30 ไม่อยู่ในผลลัพธ์ของ Subquery

**ตัวอย่าง IN กับรายการค่า**:
```sql
SELECT first_name, department_id
FROM employees
WHERE department_id IN (10, 20);
```
- **ผลลัพธ์**: เดียวกับด้านบน

### 2. EXISTS
- **คำอธิบาย**: ตรวจสอบว่า**มีแถวอย่างน้อยหนึ่งแถว**ที่ตรงตามเงื่อนไขใน Subquery
- **ลักษณะเด่น**:
    - ใช้กับ **Correlated Subquery** โดย Subquery จะตรวจสอบแถวใน Query หลัก
    - หยุดประมวลผลทันทีเมื่อพบแถวที่ตรงเงื่อนไข (มีประสิทธิภาพในบางกรณี)
    - ไม่สนใจค่าที่คืนจาก Subquery เพียงตรวจสอบการมีอยู่
- **ไวยากรณ์**:
  ```sql
  SELECT columns
  FROM table1
  WHERE EXISTS (SELECT 1 FROM table2 WHERE condition AND table2.column = table1.column);
  ```

#### ตัวอย่าง:
```sql
SELECT first_name, department_id
FROM employees e
WHERE EXISTS (SELECT 1 FROM departments d WHERE d.department_id = e.department_id);
```
- เลือกพนักงานที่มี `department_id` อยู่ในตาราง `departments`
- **ผลลัพธ์**:
  ```
  first_name | department_id
  ------------+---------------
  John       | 10
  Jane       | 20
  Alice      | 10
  Bob        | 20
  ```
- **อธิบาย**:
    - Subquery ตรวจสอบว่า `department_id` ของพนักงานแต่ละคนมีอยู่ใน `departments`
    - ถ้ามีแถวใน `departments` ที่ตรงกับ `e.department_id` Subquery จะคืนค่า `TRUE`
    - Carol (department_id = 30) ไม่ปรากฏ เพราะไม่มี `department_id = 30` ใน `departments`

### 3. NOT EXISTS
- **คำอธิบาย**: ตรวจสอบว่า**ไม่มีแถว**ที่ตรงตามเงื่อนไขใน Subquery
- **ลักษณะเด่น**:
    - ตรงข้ามกับ `EXISTS`
    - ใช้กับ Correlated Subquery เพื่อหาแถวที่ไม่ตรงกับเงื่อนไขในตารางอื่น
    - หยุดประมวลผลเมื่อพบแถวที่ตรงเงื่อนไข (คืน `FALSE`)
- **ไวยากรณ์**:
  ```sql
  SELECT columns
  FROM table1
  WHERE NOT EXISTS (SELECT 1 FROM table2 WHERE condition AND table2.column = table1.column);
  ```

#### ตัวอย่าง:
```sql
SELECT first_name, department_id
FROM employees e
WHERE NOT EXISTS (SELECT 1 FROM departments d WHERE d.department_id = e.department_id);
```
- เลือกพนักงานที่ `department_id` **ไม่อยู่**ในตาราง `departments`
- **ผลลัพธ์**:
  ```
  first_name | department_id
  ------------+---------------
  Carol      | 30
  ```
- **อธิบาย**:
    - Subquery ตรวจสอบว่า `department_id` ของพนักงานแต่ละคนไม่มีใน `departments`
    - Carol (department_id = 30) ปรากฏเพราะไม่มี `department_id = 30` ใน `departments`

### การเปรียบเทียบ IN, EXISTS, NOT EXISTS
| คุณสมบัติ              | IN                                   | EXISTS                              | NOT EXISTS                          |
|------------------------|--------------------------------------|-------------------------------------|-------------------------------------|
| **ประเภท Subquery**    | Single-Row หรือ Multi-Row            | Correlated Subquery                 | Correlated Subquery                 |
| **การทำงาน**           | เปรียบเทียบค่ากับชุดผลลัพธ์         | ตรวจสอบการมีอยู่ของแถว              | ตรวจสอบการไม่มีแถว                  |
| **ประสิทธิภาพ**        | ช้ากว่าเมื่อ Subquery คืนค่าจำนวนมาก | มักเร็วกว่าในตารางใหญ่ (หยุดเมื่อเจอ) | คล้าย EXISTS แต่ตรวจสอบการไม่มีแถว |
| **การใช้งานทั่วไป**    | รายการค่าหรือ Subquery ง่าย ๆ        | ตรวจสอบความสัมพันธ์ระหว่างตาราง     | หาแถวที่ไม่มีความสัมพันธ์           |
| **ผลลัพธ์เมื่อ NULL**  | อาจทำให้เงื่อนไขเป็น `FALSE`         | ไม่มีผล (สนใจแค่การมีอยู่)          | ไม่มีผล (สนใจแค่การไม่มี)           |

### ตัวอย่างการใช้งานเพิ่มเติม:
สมมติมีตารางเพิ่มเติม:
3. **orders**:
```sql
CREATE TABLE orders (
    order_id NUMBER(10) PRIMARY KEY,
    employee_id NUMBER(10),
    order_amount NUMBER(10,2)
);
INSERT INTO orders VALUES (101, 1, 1000);
INSERT INTO orders VALUES (102, 2, 2000);
INSERT INTO orders VALUES (103, 1, 1500);
COMMIT;
```

#### 1. IN กับ Multi-Row Subquery:
```sql
SELECT first_name, department_id
FROM employees
WHERE employee_id IN (SELECT employee_id FROM orders WHERE order_amount > 1500);
```
- เลือกพนักงานที่มีคำสั่งซื้อมากกว่า 1500
- **ผลลัพธ์**:
  ```
  first_name | department_id
  ------------+---------------
  Jane       | 20
  ```
- **อธิบาย**:
    - Subquery คืน `employee_id` (2) จาก `orders` ที่มี `order_amount > 1500`
    - `WHERE employee_id IN (2)` กรองพนักงานที่มี `employee_id = 2`

#### 2. EXISTS กับ Correlated Subquery:
```sql
SELECT first_name, department_id
FROM employees e
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.employee_id = e.employee_id AND o.order_amount > 1500);
```
- เลือกพนักงานที่มีคำสั่งซื้อมากกว่า 1500
- **ผลลัพธ์**:
  ```
  first_name | department_id
  ------------+---------------
  Jane       | 20
  ```
- **อธิบาย**:
    - Subquery ตรวจสอบว่าแต่ละ `employee_id` มีคำสั่งซื้อที่มี `order_amount > 1500`
    - หยุดเมื่อพบแถวที่ตรงเงื่อนไข

#### 3. NOT EXISTS:
```sql
SELECT first_name, department_id
FROM employees e
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.employee_id = e.employee_id);
```
- เลือกพนักงานที่**ไม่มี**คำสั่งซื้อ
- **ผลลัพธ์**:
  ```
  first_name | department_id
  ------------+---------------
  Alice      | 10
  Bob        | 20
  Carol      | 30
  ```
- **อธิบาย**:
    - Subquery ตรวจสอบว่าไม่มีแถวใน `orders` ที่มี `employee_id` ตรงกับพนักงาน
    - พนักงานที่ไม่มีคำสั่งซื้อจะปรากฏในผลลัพธ์

### ข้อควรระวัง:
- **IN**:
    - ถ้า Subquery คืนค่าจำนวนมาก อาจช้าเพราะต้องเก็บผลลัพธ์ทั้งหมดในหน่วยความจำ
    - ถ้า Subquery คืน `NULL` อาจทำให้เงื่อนไข `IN` ไม่คืนผลลัพธ์ที่คาดหวัง (เช่น `column IN (1, NULL)` อาจมีปัญหา)
    - ตัวอย่าง:
      ```sql
      SELECT first_name FROM employees WHERE department_id IN (10, NULL);
      ```
      จะไม่รวมพนักงานที่มี `department_id = NULL`
- **EXISTS**:
    - มักเร็วกว่า `IN` ในตารางใหญ่ เพราะหยุดเมื่อพบแถวแรกที่ตรงเงื่อนไข
    - เหมาะสำหรับ Correlated Subquery ที่ต้องการตรวจสอบการมีอยู่
- **NOT EXISTS**:
    - อาจช้ากว่า `NOT IN` ในบางกรณีถ้าตารางใน Subquery มีขนาดใหญ่
    - เหมาะสำหรับหาแถวที่ไม่มีความสัมพันธ์
- **ประสิทธิภาพ**:
    - ในบางกรณี การใช้ **JOIN** อาจเร็วกว่า `IN`, `EXISTS`, หรือ `NOT EXISTS`
    - ตัวอย่างการแปลง `EXISTS` เป็น JOIN:
      ```sql
      -- แทน EXISTS
      SELECT first_name, e.department_id
      FROM employees e
      WHERE EXISTS (SELECT 1 FROM departments d WHERE d.department_id = e.department_id);
  
      -- เป็น JOIN
      SELECT e.first_name, e.department_id
      FROM employees e
      INNER JOIN departments d
      ON e.department_id = d.department_id;
      ```
- **NULL**:
    - `IN` มีปัญหากับ `NULL` ใน Subquery (อาจทำให้ไม่มีผลลัพธ์)
    - `EXISTS` และ `NOT EXISTS` ไม่สนใจค่าใน Subquery จึงไม่ได้รับผลกระทบจาก `NULL`
- **ระบบฐานข้อมูล**:
    - Oracle, MySQL, PostgreSQL, SQL Server รองรับ `IN`, `EXISTS`, `NOT EXISTS`
    - **MySQL**: ประสิทธิภาพของ `IN` อาจต่ำในเวอร์ชันเก่า ควรใช้ `EXISTS` หรือ `JOIN`
    - **PostgreSQL**: มีการปรับแต่งประสิทธิภาพที่ดีสำหรับ `EXISTS` และ `NOT EXISTS`
    - **SQL Server**: คล้าย Oracle แต่ควรทดสอบประสิทธิภาพเมื่อใช้กับตารางใหญ่

### การใช้งานใน PL/SQL:
สามารถใช้ `IN`, `EXISTS`, หรือ `NOT EXISTS` ใน PL/SQL เพื่อกรองข้อมูล เช่น:
```sql
DECLARE
   CURSOR no_orders IS
      SELECT first_name, department_id
      FROM employees e
      WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.employee_id = e.employee_id);
BEGIN
   FOR emp IN no_orders LOOP
      DBMS_OUTPUT.PUT_LINE('Employee: ' || emp.first_name || 
                           ', Department: ' || emp.department_id);
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Employee: Alice, Department: 10
  Employee: Bob, Department: 20
  Employee: Carol, Department: 30
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการรายงานพนักงานและแผนกที่มีคำสั่งซื้อเกิน 1000:

#### ใช้ IN:
```sql
SELECT e.first_name, d.department_name
FROM employees e
JOIN departments d
ON e.department_id = d.department_id
WHERE e.employee_id IN (SELECT employee_id FROM orders WHERE order_amount > 1000);
```
- **ผลลัพธ์**:
  ```
  first_name | department_name
  ------------+----------------
  John       | Sales
  Jane       | IT
  ```
- **อธิบาย**:
    - Subquery คืน `employee_id` (1, 2) จาก `orders` ที่มี `order_amount > 1000`
    - กรองพนักงานที่มี `employee_id` อยู่ในรายการนั้น

#### ใช้ EXISTS:
```sql
SELECT e.first_name, d.department_name
FROM employees e
JOIN departments d
ON e.department_id = d.department_id
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.employee_id = e.employee_id AND o.order_amount > 1000);
```
- **ผลลัพธ์**: เดียวกับด้านบน
- **อธิบาย**:
    - Subquery ตรวจสอบว่าแต่ละพนักงานมีคำสั่งซื้อที่มี `order_amount > 1000`

#### ใช้ NOT EXISTS:
```sql
SELECT d.department_name
FROM departments d
WHERE NOT EXISTS (SELECT 1 FROM employees e 
                 JOIN orders o ON e.employee_id = o.employee_id 
                 WHERE e.department_id = d.department_id);
```
- เลือกแผนกที่ไม่มีพนักงานที่มีคำสั่งซื้อ
- **ผลลัพธ์**:
  ```
  department_name
  ----------------
  HR
  ```
- **อธิบาย**:
    - Subquery ตรวจสอบว่าไม่มีพนักงานในแผนกที่มีคำสั่งซื้อ
    - แผนก HR (department_id = 40) ไม่มีพนักงานที่มีคำสั่งซื้อ
---

## การใช้ `WITH` (Common Table Expression)
ใน SQL คำสั่ง **WITH** หรือ **Common Table Expression (CTE)** ใช้สำหรับสร้าง**ตารางชั่วคราว** (Temporary Result Set) ที่สามารถอ้างอิงได้ภายใน Query หลักหรือ Query อื่น ๆ ในขอบเขตเดียวกัน CTE ช่วยให้โค้ด SQL อ่านง่ายขึ้น, บำรุงรักษาง่ายขึ้น, และเหมาะสำหรับการจัดการ Query ที่ซับซ้อน โดยเฉพาะเมื่อต้องการใช้ผลลัพธ์จาก Subquery หลายครั้งหรือแยกตรรกะการคำนวณออกเป็นขั้นตอน ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### คำอธิบายของ WITH (CTE)
- **CTE** คือตารางชั่วคราวที่กำหนดโดยใช้คำสั่ง `WITH` และมีชื่อ (Alias) สำหรับอ้างอิงใน Query
- **ลักษณะเด่น**:
    - อ่านง่ายกว่า Subquery ที่ซ้อนกันหลายชั้น
    - สามารถอ้างอิง CTE เดียวกันได้หลายครั้งใน Query
    - รองรับ **Recursive CTE** สำหรับการประมวลผลข้อมูลแบบวนซ้ำ (เช่น การจัดการโครงสร้างแบบลำดับชั้น)
    - มีขอบเขตจำกัดภายใน Query เดียว (หายไปเมื่อ Query สิ้นสุด)
- **เมื่อใช้ CTE**:
    - แทน Subquery ใน `FROM` หรือ Inline View เพื่อให้โค้ดชัดเจน
    - แยกตรรกะการคำนวณเป็นขั้นตอน
    - ทำงานกับข้อมูลลำดับชั้นหรือกราฟด้วย Recursive CTE
    - ใช้ผลลัพธ์เดียวกันซ้ำในหลายส่วนของ Query

### ไวยากรณ์พื้นฐาน
```sql
WITH cte_name [(column1, column2, ...)] AS (
    SELECT columns
    FROM table
    WHERE condition
    ...
)
SELECT columns
FROM cte_name
[WHERE condition]
[GROUP BY columns]
[HAVING condition]
[ORDER BY columns];
```

- **cte_name**: ชื่อของ CTE (เหมือนตารางชั่วคราว)
- **column1, column2, ...**: ชื่อคอลัมน์ของ CTE (ไม่บังคับ ถ้าไม่ระบุจะใช้ชื่อจาก Subquery)
- **AS (SELECT ...)**: Query ที่สร้างผลลัพธ์สำหรับ CTE
- สามารถกำหนด CTE หลายตัวใน `WITH` เดียวกัน โดยคั่นด้วยเครื่องหมายจุลภาค

### ตัวอย่างการใช้งาน
สมมติมีตาราง:
1. **employees**:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    salary NUMBER(10,2),
    department_id NUMBER(5),
    manager_id NUMBER(10)
);
INSERT INTO employees VALUES (1, 'John', 50000, 10, NULL);
INSERT INTO employees VALUES (2, 'Jane', 60000, 20, 1);
INSERT INTO employees VALUES (3, 'Alice', 55000, 10, 1);
INSERT INTO employees VALUES (4, 'Bob', 65000, 20, 2);
INSERT INTO employees VALUES (5, 'Carol', 52000, 30, 2);
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
INSERT INTO departments VALUES (30, 'HR');
COMMIT;
```

#### 1. CTE พื้นฐาน
```sql
WITH dept_summary AS (
    SELECT department_id, AVG(salary) AS avg_salary, COUNT(*) AS emp_count
    FROM employees
    GROUP BY department_id
)
SELECT d.department_name, ds.avg_salary, ds.emp_count
FROM dept_summary ds
JOIN departments d
ON ds.department_id = d.department_id
WHERE ds.avg_salary > 55000
ORDER BY ds.avg_salary DESC;
```
- **คำอธิบาย**:
    - CTE ชื่อ `dept_summary` คำนวณค่าเฉลี่ยเงินเดือนและจำนวนพนักงานในแต่ละแผนก
    - Query หลัก JOIN กับ `departments` เพื่อดึงชื่อแผนก และกรองแผนกที่มีค่าเฉลี่ยเงินเดือน > 55,000
- **ผลลัพธ์**:
  ```
  department_name | avg_salary | emp_count
  ----------------+------------+-----------
  IT             | 62500      | 2
  ```
- **อธิบาย**:
    - `dept_summary` มีข้อมูลแผนก 10 (avg_salary = 52500, emp_count = 2), 20 (62500, 2), 30 (52000, 1)
    - หลังกรองด้วย `avg_salary > 55000` เหลือเฉพาะแผนก 20 (IT)

#### 2. CTE หลายตัว
```sql
WITH dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
),
high_salary_employees AS (
    SELECT e.first_name, e.salary, e.department_id
    FROM employees e
    JOIN dept_avg da
    ON e.department_id = da.department_id
    WHERE e.salary > da.avg_salary
)
SELECT hse.first_name, hse.salary, d.department_name
FROM high_salary_employees hse
JOIN departments d
ON hse.department_id = d.department_id
ORDER BY hse.salary DESC;
```
- **คำอธิบาย**:
    - CTE `dept_avg` คำนวณค่าเฉลี่ยเงินเดือนในแต่ละแผนก
    - CTE `high_salary_employees` เลือกพนักงานที่มีเงินเดือนมากกว่าค่าเฉลี่ยของแผนก
    - Query หลัก JOIN กับ `departments` เพื่อดึงชื่อแผนก
- **ผลลัพธ์**:
  ```
  first_name | salary | department_name
  ------------+--------+----------------
  Bob        | 65000  | IT
  Alice      | 55000  | Sales
  ```
- **อธิบาย**:
    - แผนก 10: ค่าเฉลี่ย = 52500, Alice (55000) ผ่านเงื่อนไข
    - แผนก 20: ค่าเฉลี่ย = 62500, Bob (65000) ผ่านเงื่อนไข

#### 3. Recursive CTE
- **คำอธิบาย**: ใช้สำหรับประมวลผลข้อมูลที่มีโครงสร้างลำดับชั้น เช่น การหาพนักงานและผู้จัดการทั้งหมดในสายการบังคับบัญชา
- **ตัวอย่าง**:
```sql
WITH emp_hierarchy AS (
    -- Anchor Member: เลือกพนักงานระดับสูงสุด (ไม่มีผู้จัดการ)
    SELECT employee_id, first_name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    -- Recursive Member: เลือกพนักงานที่อยู่ใต้ผู้จัดการ
    SELECT e.employee_id, e.first_name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN emp_hierarchy eh
    ON e.manager_id = eh.employee_id
)
SELECT first_name, level, 
       (SELECT first_name FROM employees WHERE employee_id = emp_hierarchy.manager_id) AS manager_name
FROM emp_hierarchy
ORDER BY level, first_name;
```
- **ผลลัพธ์**:
  ```
  first_name | level | manager_name
  ------------+-------+--------------
  John       | 0     | NULL
  Alice      | 1     | John
  Jane       | 1     | John
  Bob        | 2     | Jane
  Carol      | 2     | Jane
  ```
- **อธิบาย**:
    - **Anchor Member**: เลือก John (manager_id = NULL) เป็นระดับ 0
    - **Recursive Member**: เลือกพนักงานที่อยู่ใต้ผู้จัดการในระดับถัดไป (เช่น Alice, Jane ใต้ John; Bob, Carol ใต้ Jane)
    - คอลัมน์ `level` แสดงระดับในลำดับชั้น
    - Subquery ดึงชื่อผู้จัดการสำหรับแต่ละพนักงาน

#### 4. CTE ร่วมกับ Subquery และ JOIN
```sql
WITH high_salary_depts AS (
    SELECT department_id
    FROM employees
    GROUP BY department_id
    HAVING AVG(salary) > (SELECT AVG(salary) FROM employees)
)
SELECT e.first_name, e.salary, d.department_name
FROM employees e
JOIN departments d
ON e.department_id = d.department_id
WHERE e.department_id IN (SELECT department_id FROM high_salary_depts)
ORDER BY e.salary DESC;
```
- **คำอธิบาย**:
    - CTE `high_salary_depts` หาแผนกที่มีค่าเฉลี่ยเงินเดือนมากกว่าค่าเฉลี่ยรวม
    - Query หลักเลือกพนักงานในแผนกเหล่านั้นและ JOIN กับ `departments`
- **ผลลัพธ์**:
  ```
  first_name | salary | department_name
  ------------+--------+----------------
  Bob        | 65000  | IT
  Jane       | 60000  | IT
  ```
- **อธิบาย**:
    - ค่าเฉลี่ยเงินเดือนรวม = 56400
    - แผนก 20 (IT) มีค่าเฉลี่ย 62500 > 56400 จึงปรากฏในผลลัพธ์

### ข้อควรระวัง
- **ประสิทธิภาพ**:
    - CTE ไม่ได้เก็บผลลัพธ์ในดิสก์ (เหมือนตารางชั่วคราว) แต่เก็บในหน่วยความจำและประมวลผลใหม่ในแต่ละการอ้างอิง
    - ใน Oracle, CTE อาจไม่เหมาะกับการคำนวณซ้ำหลายครั้งใน Query เดียว ถ้าต้องการประสิทธิภาพสูง อาจใช้ **Materialized View** หรือ **Temporary Table**
    - Correlated Subquery ภายใน CTE อาจช้า ควรพิจารณาใช้ JOIN แทน
- **ขอบเขต**:
    - CTE มีขอบเขตจำกัดภายใน Query เดียว ไม่สามารถอ้างอิงนอก Query ได้
    - ถ้าต้องการใช้ผลลัพธ์ในหลาย Query อาจต้องสร้างตารางชั่วคราวหรือ View
- **Recursive CTE**:
    - ต้องระวังการวนซ้ำที่ไม่มีที่สิ้นสุด ควรมีเงื่อนไขที่ทำให้การวนซ้ำหยุด (เช่น จำกัดระดับหรือกรองข้อมูล)
    - ใน Oracle, Recursive CTE ต้องใช้ `UNION ALL` (ไม่รองรับ `UNION`)
- **ระบบฐานข้อมูล**:
    - Oracle, PostgreSQL, SQL Server รองรับ CTE และ Recursive CTE อย่างสมบูรณ์
    - **MySQL**: รองรับ CTE ตั้งแต่เวอร์ชัน 8.0 (ก่อนหน้านี้ต้องใช้ Subquery หรือตารางชั่วคราว)
    - **SQLite**: รองรับ CTE แต่ Recursive CTE อาจมีข้อจำกัดในบางกรณี
- **การตั้งชื่อคอลัมน์**:
    - ถ้าไม่ระบุชื่อคอลัมน์ใน CTE (เช่น `WITH cte_name AS ...`) จะใช้ชื่อคอลัมน์จาก Subquery
    - ควรระบุชื่อคอลัมน์เมื่อ Subquery มีการคำนวณหรือชื่อคอลัมน์ไม่ชัดเจน

### การเปรียบเทียบ CTE กับ Subquery
| คุณสมบัติ              | CTE                                  | Subquery                           |
|------------------------|--------------------------------------|------------------------------------|
| **ความชัดเจน**         | อ่านง่าย แยกตรรกะเป็นขั้นตอน        | อาจซับซ้อนถ้าซ้อนหลายชั้น         |
| **การใช้งานซ้ำ**       | อ้างอิงได้หลายครั้งใน Query เดียว     | ต้องเขียนซ้ำถ้าต้องใช้หลายครั้ง    |
| **Recursive**          | รองรับ Recursive CTE                 | ไม่รองรับการวนซ้ำ                  |
| **ประสิทธิภาพ**        | อาจช้ากว่าในบางกรณี (ขึ้นอยู่กับระบบ) | อาจเร็วกว่าใน Query ง่าย ๆ         |
| **การใช้งานทั่วไป**    | Query ซับซ้อน, ลำดับชั้น           | Query ง่าย, เงื่อนไขสั้น ๆ        |

**ตัวอย่างการแปลง Subquery เป็น CTE**:
- Subquery:
  ```sql
  SELECT first_name, salary
  FROM employees e
  WHERE department_id IN (
      SELECT department_id
      FROM employees
      GROUP BY department_id
      HAVING AVG(salary) > 55000
  );
  ```
- CTE:
  ```sql
  WITH high_salary_depts AS (
      SELECT department_id
      FROM employees
      GROUP BY department_id
      HAVING AVG(salary) > 55000
  )
  SELECT first_name, salary
  FROM employees e
  WHERE department_id IN (SELECT department_id FROM high_salary_depts);
  ```

### การใช้งานใน PL/SQL
สามารถใช้ CTE ใน PL/SQL เพื่อแยกตรรกะและจัดการข้อมูล เช่น:
```sql
DECLARE
   CURSOR emp_report IS
      WITH dept_summary AS (
          SELECT department_id, AVG(salary) AS avg_salary, COUNT(*) AS emp_count
          FROM employees
          GROUP BY department_id
      )
      SELECT d.department_name, ds.avg_salary, ds.emp_count
      FROM dept_summary ds
      JOIN departments d
      ON ds.department_id = d.department_id
      WHERE ds.emp_count > 1
      ORDER BY ds.avg_salary DESC;
BEGIN
   FOR rec IN emp_report LOOP
      DBMS_OUTPUT.PUT_LINE('Department: ' || rec.department_name || 
                           ', Avg Salary: ' || rec.avg_salary || 
                           ', Employees: ' || rec.emp_count);
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Department: IT, Avg Salary: 62500, Employees: 2
  Department: Sales, Avg Salary: 52500, Employees: 2
  ```

### ตัวอย่างสถานการณ์จริง
สมมติต้องการรายงานพนักงานที่มีเงินเดือนสูงกว่าค่าเฉลี่ยของแผนก และแสดงลำดับชั้นของผู้จัดการ:

```sql
WITH dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
),
high_salary_employees AS (
    SELECT e.first_name, e.salary, e.department_id, e.manager_id
    FROM employees e
    JOIN dept_avg da
    ON e.department_id = da.department_id
    WHERE e.salary > da.avg_salary
),
emp_hierarchy AS (
    SELECT employee_id, first_name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.employee_id, e.first_name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN emp_hierarchy eh
    ON e.manager_id = eh.employee_id
)
SELECT hse.first_name, hse.salary, d.department_name, eh.level, 
       (SELECT first_name FROM employees WHERE employee_id = hse.manager_id) AS manager_name
FROM high_salary_employees hse
JOIN departments d
ON hse.department_id = d.department_id
JOIN emp_hierarchy eh
ON hse.first_name = eh.first_name
ORDER BY hse.salary DESC;
```
- **ผลลัพธ์**:
  ```
  first_name | salary | department_name | level | manager_name
  ------------+--------+----------------+-------+--------------
  Bob        | 65000  | IT             | 2     | Jane
  Alice      | 55000  | Sales          | 1     | John
  ```
- **อธิบาย**:
    - `dept_avg`: คำนวณค่าเฉลี่ยเงินเดือนในแต่ละแผนก
    - `high_salary_employees`: เลือกพนักงานที่มีเงินเดือนมากกว่าค่าเฉลี่ยของแผนก
    - `emp_hierarchy`: สร้างลำดับชั้นของพนักงาน
    - Query หลักรวมข้อมูลและแสดงระดับในลำดับชั้นพร้อมชื่อผู้จัดการ
---
