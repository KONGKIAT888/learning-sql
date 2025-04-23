# บทที่ 6: การใช้ฟังก์ชันกลุ่ม (Aggregate Functions)

## สารบัญ

- [SUM, AVG, MIN, MAX, COUNT](#sum-avg-min-max-count)
- [การใช้ GROUP BY เพื่อจัดกลุ่มข้อมูล](#การใช้-group-by-เพื่อจัดกลุ่มข้อมูล)
- [การใช้ HAVING เพื่อกรองกลุ่ม](#การใช้-having-เพื่อกรองกลุ่ม)

## `SUM`, `AVG`, `MIN`, `MAX`, `COUNT`
ใน SQL ฟังก์ชันรวม (**Aggregate Functions**) เช่น **SUM**, **AVG**, **MIN**, **MAX**, และ **COUNT** ใช้สำหรับคำนวณหรือสรุปข้อมูลจากหลายแถวในตาราง ฟังก์ชันเหล่านี้มักใช้ร่วมกับคำสั่ง `SELECT` และ `GROUP BY` เพื่อวิเคราะห์ข้อมูล เช่น การหาผลรวม, ค่าเฉลี่ย, ค่าน้อยสุด, ค่าสูงสุด, หรือจำนวนแถว ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### 1. SUM
- **คำอธิบาย**: คำนวณ**ผลรวม**ของค่าตัวเลขในคอลัมน์
- **ไวยากรณ์**:
  ```sql
  SUM(column_name)
  ```
- **หมายเหตุ**: ละเว้นค่า `NULL` และทำงานกับคอลัมน์ตัวเลขเท่านั้น
- **ตัวอย่าง**:
  ```sql
  SELECT SUM(salary) AS total_salary
  FROM employees;
  ```
  - หาผลรวมเงินเดือนของพนักงานทั้งหมด

### 2. AVG
- **คำอธิบาย**: คำนวณ**ค่าเฉลี่ย**ของค่าตัวเลขในคอลัมน์
- **ไวยากรณ์**:
  ```sql
  AVG(column_name)
  ```
- **หมายเหตุ**: ละเว้นค่า `NULL` และคำนวณจากจำนวนแถวที่มีค่าไม่เป็น `NULL`
- **ตัวอย่าง**:
  ```sql
  SELECT AVG(salary) AS average_salary
  FROM employees;
  ```
  - หาค่าเฉลี่ยเงินเดือนของพนักงาน

### 3. MIN
- **คำอธิบาย**: หาค่า**น้อยที่สุด**ในคอลัมน์ (ใช้ได้กับตัวเลข, สตริง, หรือวันที่)
- **ไวยากรณ์**:
  ```sql
  MIN(column_name)
  ```
- **ตัวอย่าง**:
  ```sql
  SELECT MIN(salary) AS min_salary
  FROM employees;
  ```
  - หาเงินเดือนที่น้อยที่สุด

### 4. MAX
- **คำอธิบาย**: หาค่า**สูงที่สุด**ในคอลัมน์ (ใช้ได้กับตัวเลข, สตริง, หรือวันที่)
- **ไวยากรณ์**:
  ```sql
  MAX(column_name)
  ```
- **ตัวอย่าง**:
  ```sql
  SELECT MAX(salary) AS max_salary
  FROM employees;
  ```
  - หาเงินเดือนที่สูงที่สุด

### 5. COUNT
- **คำอธิบาย**: นับ**จำนวนแถว**หรือจำนวนค่าที่ไม่เป็น `NULL` ในคอลัมน์
- **ไวยากรณ์**:
  ```sql
  COUNT(column_name | * | DISTINCT column_name)
  ```
  - `COUNT(*)`: นับทุกแถว รวมทั้งที่มี `NULL`
  - `COUNT(column_name)`: นับแถวที่มีค่าไม่เป็น `NULL` ในคอลัมน์นั้น
  - `COUNT(DISTINCT column_name)`: นับจำนวนค่าไม่ซ้ำในคอลัมน์
- **ตัวอย่าง**:
  ```sql
  SELECT COUNT(*) AS total_employees
  FROM employees;
  ```
  - นับจำนวนพนักงานทั้งหมด

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees`:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    salary NUMBER(10,2),
    department_id NUMBER(5)
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 50000, 10);
INSERT INTO employees VALUES (2, 'Jane', 60000, 20);
INSERT INTO employees VALUES (3, 'Alice', 55000, 10);
INSERT INTO employees VALUES (4, 'Bob', 65000, 20);
INSERT INTO employees VALUES (5, ' Carol', NULL, 10);
COMMIT;
```

#### 1. ใช้ฟังก์ชันรวมทั้งหมด:
```sql
SELECT 
    SUM(salary) AS total_salary,
    AVG(salary) AS avg_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    COUNT(*) AS total_rows,
    COUNT(salary) AS non_null_salary,
    COUNT(DISTINCT department_id) AS unique_depts
FROM employees;
```
- **ผลลัพธ์**:
  ```
  total_salary | avg_salary | min_salary | max_salary | total_rows | non_null_salary | unique_depts
  -------------+------------+------------+------------+------------+-----------------+--------------
  230000       | 57500      | 50000      | 65000      | 5          | 4               | 2
  ```
- **อธิบาย**:
  - `SUM(salary)`: รวมเงินเดือน = 50000 + 60000 + 55000 + 65000 = 230000
  - `AVG(salary)`: ค่าเฉลี่ย = 230000 / 4 = 57500 (ละเว้น `NULL`)
  - `MIN(salary)`: เงินเดือนน้อยสุด = 50000
  - `MAX(salary)`: เงินเดือนสูงสุด = 65000
  - `COUNT(*)`: นับทุกแถว = 5
  - `COUNT(salary)`: นับแถวที่มี `salary` ไม่เป็น `NULL` = 4
  - `COUNT(DISTINCT department_id)`: นับแผนกไม่ซ้ำ = 2 (10 และ 20)

#### 2. รวมกับ GROUP BY:
```sql
SELECT department_id,
       SUM(salary) AS dept_total,
       AVG(salary) AS dept_avg,
       MIN(salary) AS dept_min,
       MAX(salary) AS dept_max,
       COUNT(*) AS dept_count
FROM employees
GROUP BY department_id;
```
- คำนวณผลรวม, ค่าเฉลี่ย, ค่าน้อยสุด, ค่าสูงสุด, และจำนวนพนักงานในแต่ละแผนก
- **ผลลัพธ์**:
  ```
  department_id | dept_total | dept_avg   | dept_min | dept_max | dept_count
  --------------+------------+------------+----------+----------+------------
  10            | 105000     | 52500      | 50000    | 55000    | 3
  20            | 125000     | 62500      | 60000    | 65000    | 2
  ```
- **อธิบาย**:
  - แผนก 10: รวมเงินเดือน = 50000 + 55000 = 105000, เฉลี่ย = 105000 / 2 = 52500
  - แผนก 20: รวมเงินเดือน = 60000 + 65000 = 125000, เฉลี่ย = 125000 / 2 = 62500
  - `COUNT(*)` รวมแถวที่มี `salary` เป็น `NULL` (เช่น Carol ในแผนก 10)

#### 3. รวมกับ HAVING:
```sql
SELECT department_id,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 60000;
```
- แสดงเฉพาะแผนกที่มีค่าเฉลี่ยเงินเดือนมากกว่า 60,000
- **ผลลัพธ์**:
  ```
  department_id | avg_salary
  --------------+------------
  20            | 62500
  ```

#### 4. รวมกับ WHERE:
```sql
SELECT COUNT(*) AS high_salary_count
FROM employees
WHERE salary > 55000;
```
- นับจำนวนพนักงานที่มีเงินเดือนมากกว่า 55,000
- **ผลลัพธ์**:
  ```
  high_salary_count
  -----------------
  2
  ```

### ข้อควรระวัง:
- **NULL**:
  - `SUM`, `AVG`, `MIN`, `MAX` ละเว้นค่า `NULL` โดยอัตโนมัติ
  - `COUNT(*)` นับทุกแถว รวมทั้งที่มี `NULL`, แต่ `COUNT(column_name)` นับเฉพาะค่าไม่เป็น `NULL`
- **GROUP BY**:
  - ถ้าใช้ Aggregate Functions ร่วมกับคอลัมน์ที่ไม่ใช่ Aggregate ต้องรวมคอลัมน์นั้นใน `GROUP BY`
    ```sql
    SELECT department_id, AVG(salary) FROM employees GROUP BY department_id;
    ```
- **ประเภทข้อมูล**:
  - `SUM` และ `AVG` ใช้กับตัวเลขเท่านั้น
  - `MIN` และ `MAX` ใช้ได้กับตัวเลข, สตริง, หรือวันที่ (เช่น `MIN(hire_date)` หาวันที่เริ่มงานเร็วสุด)
- **ประสิทธิภาพ**:
  - การคำนวณข้อมูลจำนวนมากอาจช้า ควรใช้ดัชนี (Indexes) บนคอลัมน์ที่ใช้ใน `WHERE` หรือ `GROUP BY`
- **ระบบฐานข้อมูล**:
  - Oracle, MySQL, PostgreSQL, SQL Server รองรับฟังก์ชันเหล่านี้เหมือนกัน
  - MySQL: `COUNT(*)` อาจเร็วกว่าในบางกรณี เนื่องจากการ优化
  - PostgreSQL: รองรับ `COUNT(DISTINCT column)` เช่นเดียวกับ Oracle
  - SQL Server: คล้าย Oracle แต่ควรระวังการจัดการ `NULL` ในบางกรณี

### การใช้งานใน PL/SQL:
สามารถใช้ Aggregate Functions ใน PL/SQL เพื่อคำนวณและประมวลผลข้อมูล เช่น:
```sql
DECLARE
   v_total_salary NUMBER;
   v_avg_salary NUMBER;
   v_employee_count NUMBER;
BEGIN
   SELECT SUM(salary), AVG(salary), COUNT(*)
   INTO v_total_salary, v_avg_salary, v_employee_count
   FROM employees
   WHERE department_id = 10;
   
   DBMS_OUTPUT.PUT_LINE('Total Salary: ' || v_total_salary);
   DBMS_OUTPUT.PUT_LINE('Average Salary: ' || v_avg_salary);
   DBMS_OUTPUT.PUT_LINE('Employee Count: ' || v_employee_count);
EXCEPTION
   WHEN NO_DATA_FOUND THEN
      DBMS_OUTPUT.PUT_LINE('No data found.');
END;
/
```
- **ผลลัพธ์**:
  ```
  Total Salary: 105000
  Average Salary: 52500
  Employee Count: 3
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการวิเคราะห์ข้อมูลพนักงานในแต่ละแผนก:
```sql
SELECT department_id,
       COUNT(*) AS employee_count,
       SUM(salary) AS total_salary,
       ROUND(AVG(salary), 2) AS avg_salary,
       MIN(salary) AS min_salary,
       MAX(salary) AS max_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 1
ORDER BY avg_salary DESC;
```
- คำนวณจำนวนพนักงาน, ผลรวมเงินเดือน, ค่าเฉลี่ย (ปัด 2 ทศนิยม), เงินเดือนต่ำสุดและสูงสุดในแต่ละแผนก
- กรองเฉพาะแผนกที่มีพนักงานมากกว่า 1 คน
- เรียงลำดับตามค่าเฉลี่ยเงินเดือนจากมากไปน้อย
- **ผลลัพธ์**:
  ```
  department_id | employee_count | total_salary | avg_salary | min_salary | max_salary
  --------------+---------------+--------------+------------+------------+------------
  20            | 2             | 125000       | 62500.00   | 60000      | 65000
  10            | 3             | 105000       | 52500.00   | 50000      | 55000
  ```

### การรวมกับฟังก์ชันอื่น:
สามารถใช้ Aggregate Functions ร่วมกับ `CASE` หรือ `DISTINCT` เช่น:
```sql
SELECT department_id,
       SUM(CASE WHEN salary > 55000 THEN salary ELSE 0 END) AS high_salary_total
FROM employees
GROUP BY department_id;
```
- คำนวณผลรวมเงินเดือนเฉพาะพนักงานที่มีเงินเดือนมากกว่า 55,000 ในแต่ละแผนก
- **ผลลัพธ์**:
  ```
  department_id | high_salary_total
  --------------+------------------
  10            | 0
  20            | 125000
  ```
---

## การใช้ `GROUP BY` เพื่อจัดกลุ่มข้อมูล
ใน SQL คำสั่ง **GROUP BY** ใช้สำหรับ**จัดกลุ่มข้อมูล**ในตารางตามคอลัมน์ที่ระบุ เพื่อคำนวณหรือสรุปผลด้วย **Aggregate Functions** เช่น `SUM`, `AVG`, `COUNT`, `MIN`, `MAX` คำสั่งนี้มีประโยชน์ในการวิเคราะห์ข้อมูล เช่น การหาผลรวมเงินเดือนในแต่ละแผนก หรือนับจำนวนพนักงานตามสถานะ ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### ไวยากรณ์พื้นฐาน:
```sql
SELECT column1, column2, ..., aggregate_function(column)
FROM table_name
[WHERE condition]
GROUP BY column1, column2, ...
[HAVING condition]
[ORDER BY column];
```

### อธิบายส่วนประกอบ:
- **SELECT**: ระบุคอลัมน์ที่ต้องการแสดงและ Aggregate Functions
  - คอลัมน์ที่ไม่ใช่ Aggregate Function ต้องอยู่ใน `GROUP BY`
- **FROM**: ตารางที่ดึงข้อมูล
- **WHERE**: กรองแถวก่อนจัดกลุ่ม (ถ้ามี)
- **GROUP BY**: ระบุคอลัมน์ที่ใช้จัดกลุ่ม
  - แถวที่มีค่าเหมือนกันในคอลัมน์เหล่านี้จะถูกรวมเป็นกลุ่มเดียว
- **HAVING**: กรองกลุ่มตามเงื่อนไข (คล้าย `WHERE` แต่ใช้กับกลุ่ม)
- **ORDER BY**: จัดเรียงผลลัพธ์ (ถ้ามี)
- **Aggregate Functions**:
  - `SUM`: ผลรวม
  - `AVG`: ค่าเฉลี่ย
  - `COUNT`: จำนวน
  - `MIN`: ค่าน้อยสุด
  - `MAX`: ค่าสูงสุด

### ลำดับการประมวลผล:
1. **FROM**: เลือกตาราง
2. **WHERE**: กรองแถว
3. **GROUP BY**: จัดกลุ่มตามคอลัมน์
4. **HAVING**: กรองกลุ่ม
5. **SELECT**: เลือกคอลัมน์และคำนวณ Aggregate Functions
6. **ORDER BY**: จัดเรียงผลลัพธ์

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees`:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    salary NUMBER(10,2),
    department_id NUMBER(5),
    hire_date DATE
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 50000, 10, TO_DATE('2023-01-15', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (2, 'Jane', 60000, 20, TO_DATE('2023-02-10', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (3, 'Alice', 55000, 10, TO_DATE('2023-03-01', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (4, 'Bob', 65000, 20, TO_DATE('2023-04-01', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (5, 'Carol', NULL, 10, TO_DATE('2023-05-01', 'YYYY-MM-DD'));
COMMIT;
```

#### 1. จัดกลุ่มตามแผนกและนับจำนวนพนักงาน:
```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id;
```
- นับจำนวนพนักงานในแต่ละแผนก
- **ผลลัพธ์**:
  ```
  department_id | employee_count
  --------------+---------------
  10            | 3
  20            | 2
  ```

#### 2. คำนวณผลรวมและค่าเฉลี่ยเงินเดือนตามแผนก:
```sql
SELECT department_id,
       SUM(salary) AS total_salary,
       ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY department_id;
```
- หาผลรวมและค่าเฉลี่ยเงินเดือน (ปัด 2 ทศนิยม) ในแต่ละแผนก
- **ผลลัพธ์**:
  ```
  department_id | total_salary | avg_salary
  --------------+--------------+------------
  10            | 105000       | 52500.00
  20            | 125000       | 62500.00
  ```
- **หมายเหตุ**: `AVG(salary)` ละเว้น `NULL` (เช่น Carol) ดังนั้นในแผนก 10 คำนวณจาก 2 ค่า (50000, 55000)

#### 3. รวมกับ WHERE:
```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employees
WHERE hire_date >= TO_DATE('2023-03-01', 'YYYY-MM-DD')
GROUP BY department_id;
```
- นับพนักงานที่เริ่มงานตั้งแต่ 1 มี.ค. 2023 ในแต่ละแผนก
- **ผลลัพธ์**:
  ```
  department_id | employee_count
  --------------+---------------
  10            | 2
  20            | 1
  ```

#### 4. รวมกับ HAVING:
```sql
SELECT department_id,
       SUM(salary) AS total_salary
FROM employees
GROUP BY department_id
HAVING SUM(salary) > 100000;
```
- แสดงเฉพาะแผนกที่มีผลรวมเงินเดือนมากกว่า 100,000
- **ผลลัพธ์**:
  ```
  department_id | total_salary
  --------------+--------------
  10            | 105000
  20            | 125000
  ```

#### 5. จัดกลุ่มตามหลายคอลัมน์:
```sql
SELECT department_id,
       EXTRACT(YEAR FROM hire_date) AS hire_year,
       COUNT(*) AS employee_count
FROM employees
GROUP BY department_id, EXTRACT(YEAR FROM hire_date)
ORDER BY department_id, hire_year;
```
- นับจำนวนพนักงานตามแผนกและปีที่เริ่มงาน
- **ผลลัพธ์**:
  ```
  department_id | hire_year | employee_count
  --------------+-----------+---------------
  10            | 2023      | 3
  20            | 2023      | 2
  ```

#### 6. รวมกับ CASE:
```sql
SELECT department_id,
       SUM(CASE WHEN salary > 55000 THEN 1 ELSE 0 END) AS high_salary_count
FROM employees
GROUP BY department_id;
```
- นับจำนวนพนักงานที่มีเงินเดือนมากกว่า 55,000 ในแต่ละแผนก
- **ผลลัพธ์**:
  ```
  department_id | high_salary_count
  --------------+------------------
  10            | 0
  20            | 2
  ```

### ข้อควรระวัง:
- **คอลัมน์ใน SELECT**:
  - คอลัมน์ที่ไม่ใช่ Aggregate Function (เช่น `SUM`, `COUNT`) ต้องอยู่ใน `GROUP BY`
    ```sql
    SELECT department_id, first_name, COUNT(*) -- ผิด เพราะ first_name ไม่อยู่ใน GROUP BY
    FROM employees
    GROUP BY department_id;
    ```
    แก้เป็น:
    ```sql
    SELECT department_id, COUNT(*)
    FROM employees
    GROUP BY department_id;
    ```
- **NULL**:
  - ค่า `NULL` ในคอลัมน์ที่ใช้ใน `GROUP BY` จะถูกรวมเป็นกลุ่มเดียว
  - Aggregate Functions (ยกเว้น `COUNT(*)`) ละเว้น `NULL`
- **WHERE vs HAVING**:
  - `WHERE`: กรองแถวก่อนจัดกลุ่ม
  - `HAVING`: กรองกลุ่มหลังจากใช้ `GROUP BY`
    ```sql
    SELECT department_id, AVG(salary)
    FROM employees
    WHERE salary > 50000
    GROUP BY department_id
    HAVING AVG(salary) > 55000;
    ```
- **ประสิทธิภาพ**:
  - การใช้ `GROUP BY` กับข้อมูลขนาดใหญ่อาจช้า ควรใช้ดัชนี (Indexes) บนคอลัมน์ใน `WHERE` หรือ `GROUP BY`
- **ระบบฐานข้อมูล**:
  - Oracle, MySQL, PostgreSQL, SQL Server รองรับ `GROUP BY` คล้ายกัน
  - **MySQL**: ในโหมดไม่เข้มงวด อาจอนุญาตให้เลือกคอลัมน์ที่ไม่อยู่ใน `GROUP BY` (แต่ผลลัพธ์อาจไม่แน่นอน)
  - **PostgreSQL**: เข้มงวด ต้องระบุคอลัมน์ใน `GROUP BY` เสมอ
  - **SQL Server**: คล้าย Oracle ต้องปฏิบัติตามกฎ `GROUP BY` อย่างเคร่งครัด

### การใช้งานใน PL/SQL:
สามารถใช้ `GROUP BY` ใน PL/SQL เพื่อดึงข้อมูลที่จัดกลุ่มแล้ว เช่น:
```sql
DECLARE
   CURSOR dept_summary IS
      SELECT department_id,
             COUNT(*) AS emp_count,
             SUM(salary) AS total_salary
      FROM employees
      GROUP BY department_id
      HAVING COUNT(*) > 1
      ORDER BY department_id;
BEGIN
   FOR dept IN dept_summary LOOP
      DBMS_OUTPUT.PUT_LINE('Department ' || dept.department_id || 
                           ': ' || dept.emp_count || ' employees, ' ||
                           'Total Salary: ' || dept.total_salary);
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Department 10: 3 employees, Total Salary: 105000
  Department 20: 2 employees, Total Salary: 125000
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการวิเคราะห์ข้อมูลพนักงานตามแผนก โดยแสดงเฉพาะแผนกที่มีพนักงานมากกว่า 1 คน:
```sql
SELECT department_id,
       COUNT(*) AS employee_count,
       ROUND(AVG(salary), 2) AS avg_salary,
       MIN(salary) AS min_salary,
       MAX(salary) AS max_salary
FROM employees
WHERE hire_date >= TO_DATE('2023-01-01', 'YYYY-MM-DD')
GROUP BY department_id
HAVING COUNT(*) > 1
ORDER BY avg_salary DESC;
```
- นับจำนวนพนักงาน, ค่าเฉลี่ยเงินเดือน (ปัด 2 ทศนิยม), เงินเดือนต่ำสุดและสูงสุดในแต่ละแผนก
- กรองเฉพาะพนักงานที่เริ่มงานในปี 2023
- แสดงเฉพาะแผนกที่มีพนักงานมากกว่า 1 คน
- เรียงลำดับตามค่าเฉลี่ยเงินเดือนจากมากไปน้อย
- **ผลลัพธ์**:
  ```
  department_id | employee_count | avg_salary | min_salary | max_salary
  --------------+---------------+------------+------------+------------
  20            | 2             | 62500.00   | 60000      | 65000
  10            | 3             | 52500.00   | 50000      | 55000
  ```

### การรวมกับฟังก์ชันอื่น:
สามารถใช้ `GROUP BY` ร่วมกับ `CASE` หรือ `DISTINCT` เช่น:
```sql
SELECT department_id,
       COUNT(DISTINCT CASE WHEN salary > 55000 THEN employee_id END) AS high_salary_employees
FROM employees
GROUP BY department_id;
```
- นับจำนวนพนักงานที่มีเงินเดือนมากกว่า 55,000 (ไม่ซ้ำ) ในแต่ละแผนก
- **ผลลัพธ์**:
  ```
  department_id | high_salary_employees
  --------------+---------------------
  10            | 0
  20            | 2
  ```
---

## การใช้ `HAVING` เพื่อกรองกลุ่ม
ใน SQL คำสั่ง **HAVING** ใช้สำหรับ**กรองกลุ่ม**ที่สร้างโดยคำสั่ง `GROUP BY` ตามเงื่อนไขที่ระบุ คล้ายกับ `WHERE` แต่ `HAVING` ทำงานกับ**ผลลัพธ์ของการจัดกลุ่ม** (เช่น ผลรวม, ค่าเฉลี่ย, จำนวน) แทนการกรองแถวแต่ละแถวก่อนจัดกลุ่ม คำสั่งนี้มักใช้ร่วมกับ **Aggregate Functions** เช่น `SUM`, `AVG`, `COUNT`, `MIN`, `MAX` เพื่อจำกัดผลลัพธ์ให้เฉพาะกลุ่มที่ตรงตามเงื่อนไข

### ไวยากรณ์พื้นฐาน:
```sql
SELECT column1, column2, ..., aggregate_function(column)
FROM table_name
[WHERE condition]
GROUP BY column1, column2, ...
HAVING condition
[ORDER BY column];
```

### อธิบายส่วนประกอบ:
- **SELECT**: ระบุคอลัมน์และ Aggregate Functions ที่ต้องการแสดง
- **FROM**: ตารางที่ดึงข้อมูล
- **WHERE**: กรองแถวก่อนจัดกลุ่ม (ถ้ามี)
- **GROUP BY**: จัดกลุ่มข้อมูลตามคอลัมน์ที่ระบุ
- **HAVING**: ระบุเงื่อนไขเพื่อกรองกลุ่ม (เช่น `HAVING COUNT(*) > 5`)
- **ORDER BY**: จัดเรียงผลลัพธ์ (ถ้ามี)
- **เงื่อนไขใน HAVING**: มักใช้ Aggregate Functions เช่น:
  - `SUM(column) > value`
  - `AVG(column) < value`
  - `COUNT(*) >= value`

### ความแตกต่างระหว่าง WHERE และ HAVING:
- **WHERE**:
  - กรองแถวแต่ละแถว**ก่อน**จัดกลุ่ม
  - ใช้กับคอลัมน์ทั่วไป (เช่น `salary > 50000`)
  - ไม่สามารถใช้ Aggregate Functions
- **HAVING**:
  - กรอง**กลุ่ม**ที่สร้างโดย `GROUP BY`
  - ใช้กับ Aggregate Functions (เช่น `SUM(salary) > 100000`)
  - ทำงาน**หลัง**การจัดกลุ่ม

### ลำดับการประมวลผล:
1. **FROM**: เลือกตาราง
2. **WHERE**: กรองแถว
3. **GROUP BY**: จัดกลุ่ม
4. **HAVING**: กรองกลุ่ม
5. **SELECT**: เลือกคอลัมน์และคำนวณ Aggregate Functions
6. **ORDER BY**: จัดเรียงผลลัพธ์

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees`:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    salary NUMBER(10,2),
    department_id NUMBER(5),
    hire_date DATE
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 50000, 10, TO_DATE('2023-01-15', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (2, 'Jane', 60000, 20, TO_DATE('2023-02-10', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (3, 'Alice', 55000, 10, TO_DATE('2023-03-01', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (4, 'Bob', 65000, 20, TO_DATE('2023-04-01', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (5, 'Carol', 52000, 10, TO_DATE('2023-05-01', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (6, 'David', 58000, 30, TO_DATE('2023-06-01', 'YYYY-MM-DD'));
COMMIT;
```

#### 1. กรองกลุ่มที่มีพนักงานมากกว่า 2 คน:
```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 2;
```
- นับจำนวนพนักงานในแต่ละแผนก และแสดงเฉพาะแผนกที่มีพนักงานมากกว่า 2 คน
- **ผลลัพธ์**:
  ```
  department_id | employee_count
  --------------+---------------
  10            | 3
  ```

#### 2. กรองกลุ่มที่มีผลรวมเงินเดือนมากกว่า 100,000:
```sql
SELECT department_id, SUM(salary) AS total_salary
FROM employees
GROUP BY department_id
HAVING SUM(salary) > 100000;
```
- คำนวณผลรวมเงินเดือนในแต่ละแผนก และแสดงเฉพาะแผนกที่มีผลรวมเงินเดือนมากกว่า 100,000
- **ผลลัพธ์**:
  ```
  department_id | total_salary
  --------------+--------------
  10            | 157000
  20            | 125000
  ```

#### 3. รวมกับ WHERE และ HAVING:
```sql
SELECT department_id, AVG(salary) AS avg_salary
FROM employees
WHERE hire_date >= TO_DATE('2023-02-01', 'YYYY-MM-DD')
GROUP BY department_id
HAVING AVG(salary) > 55000;
```
- กรองพนักงานที่เริ่มงานตั้งแต่ 1 ก.พ. 2023, จัดกลุ่มตามแผนก, และแสดงเฉพาะแผนกที่มีค่าเฉลี่ยเงินเดือนมากกว่า 55,000
- **ผลลัพธ์**:
  ```
  department_id | avg_salary
  --------------+------------
  20            | 62500
  ```

#### 4. ใช้ HAVING กับหลายเงื่อนไข:
```sql
SELECT department_id,
       COUNT(*) AS employee_count,
       SUM(salary) AS total_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) >= 2 AND SUM(salary) > 120000;
```
- แสดงแผนกที่มีพนักงานอย่างน้อย 2 คน และผลรวมเงินเดือนมากกว่า 120,000
- **ผลลัพธ์**:
  ```
  department_id | employee_count | total_salary
  --------------+---------------+--------------
  10            | 3             | 157000
  20            | 2             | 125000
  ```

#### 5. รวมกับ CASE ใน HAVING:
```sql
SELECT department_id,
       COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING SUM(CASE WHEN salary > 55000 THEN 1 ELSE 0 END) > 1;
```
- แสดงแผนกที่มีพนักงานที่มีเงินเดือนมากกว่า 55,000 มากกว่า 1 คน
- **ผลลัพธ์**:
  ```
  department_id | employee_count
  --------------+---------------
  20            | 2
  ```

#### 6. รวมกับ ORDER BY:
```sql
SELECT department_id,
       COUNT(*) AS employee_count,
       ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 1
ORDER BY avg_salary DESC;
```
- นับจำนวนพนักงานและคำนวณค่าเฉลี่ยเงินเดือนในแต่ละแผนก, กรองเฉพาะแผนกที่มีพนักงานมากกว่า 1 คน, และเรียงลำดับตามค่าเฉลี่ยเงินเดือนจากมากไปน้อย
- **ผลลัพธ์**:
  ```
  department_id | employee_count | avg_salary
  --------------+---------------+------------
  20            | 2             | 62500.00
  10            | 3             | 52333.33
  ```

### ข้อควรระวัง:
- **ต้องใช้ GROUP BY**:
  - `HAVING` ใช้ได้เฉพาะเมื่อมี `GROUP BY` เพราะมันกรองกลุ่มที่สร้างขึ้น
  - ถ้าต้องการกรองโดยไม่จัดกลุ่ม ใช้ `WHERE` แทน
- **เงื่อนไขใน HAVING**:
  - มักใช้ Aggregate Functions (เช่น `SUM`, `COUNT`) เพราะทำงานกับกลุ่ม
  - ถ้าใส่เงื่อนไขที่ไม่ใช่ Aggregate Function ใน `HAVING` อาจทำให้ผลลัพธ์ไม่ถูกต้องหรือช้า ควรย้ายไปใช้ `WHERE` แทน
    ```sql
    -- ผิด: ใช้เงื่อนไขทั่วไปใน HAVING
    SELECT department_id, COUNT(*)
    FROM employees
    GROUP BY department_id
    HAVING department_id = 10;

    -- ถูก: ย้ายไป WHERE
    SELECT department_id, COUNT(*)
    FROM employees
    WHERE department_id = 10
    GROUP BY department_id;
    ```
- **NULL**:
  - Aggregate Functions ใน `HAVING` ละเว้น `NULL` (ยกเว้น `COUNT(*)`)
  - ถ้าไม่มีแถวในกลุ่มหลัง `WHERE` กลุ่มนั้นจะไม่ปรากฏในผลลัพธ์
- **ประสิทธิภาพ**:
  - การใช้ `HAVING` กับข้อมูลขนาดใหญ่อาจช้า ควรใช้ `WHERE` เพื่อลดจำนวนแถวก่อนจัดกลุ่ม และใช้ดัชนี (Indexes) บนคอลัมน์ใน `WHERE` หรือ `GROUP BY`
- **ระบบฐานข้อมูล**:
  - Oracle, MySQL, PostgreSQL, SQL Server รองรับ `HAVING` คล้ายกัน
  - **MySQL**: อาจอนุญาตให้ใช้คอลัมน์ที่ไม่อยู่ใน `GROUP BY` ใน `HAVING` ในโหมดไม่เข้มงวด
  - **PostgreSQL** และ **SQL Server**: เข้มงวด ต้องใช้ Aggregate Functions หรือคอลัมน์ใน `GROUP BY` ใน `HAVING`

### การใช้งานใน PL/SQL:
สามารถใช้ `HAVING` ใน PL/SQL เพื่อดึงข้อมูลกลุ่มที่กรองแล้ว เช่น:
```sql
DECLARE
   CURSOR dept_summary IS
      SELECT department_id,
             COUNT(*) AS emp_count,
             SUM(salary) AS total_salary
      FROM employees
      GROUP BY department_id
      HAVING SUM(salary) > 100000
      ORDER BY total_salary DESC;
BEGIN
   FOR dept IN dept_summary LOOP
      DBMS_OUTPUT.PUT_LINE('Department ' || dept.department_id || 
                           ': ' || dept.emp_count || ' employees, ' ||
                           'Total Salary: ' || dept.total_salary);
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Department 10: 3 employees, Total Salary: 157000
  Department 20: 2 employees, Total Salary: 125000
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการวิเคราะห์แผนกที่มีพนักงานอย่างน้อย 2 คน และค่าเฉลี่ยเงินเดือนมากกว่า 55,000:
```sql
SELECT department_id,
       COUNT(*) AS employee_count,
       ROUND(AVG(salary), 2) AS avg_salary,
       SUM(salary) AS total_salary
FROM employees
WHERE hire_date >= TO_DATE('2023-01-01', 'YYYY-MM-DD')
GROUP BY department_id
HAVING COUNT(*) >= 2 AND AVG(salary) > 55000
ORDER BY avg_salary DESC;
```
- กรองพนักงานที่เริ่มงานในปี 2023
- จัดกลุ่มตามแผนก
- แสดงเฉพาะแผนกที่มีพนักงานอย่างน้อย 2 คน และค่าเฉลี่ยเงินเดือนมากกว่า 55,000
- เรียงลำดับตามค่าเฉลี่ยเงินเดือนจากมากไปน้อย
- **ผลลัพธ์**:
  ```
  department_id | employee_count | avg_salary | total_salary
  --------------+---------------+------------+--------------
  20            | 2             | 62500.00   | 125000
  ```
  
### การรวมกับฟังก์ชันอื่น:
สามารถใช้ `HAVING` ร่วมกับ `CASE` หรือ `DISTINCT` เช่น:
```sql
SELECT department_id,
       COUNT(DISTINCT employee_id) AS unique_employees,
       SUM(CASE WHEN salary > 55000 THEN salary ELSE 0 END) AS high_salary_total
FROM employees
GROUP BY department_id
HAVING SUM(CASE WHEN salary > 55000 THEN salary ELSE 0 END) > 100000;
```
- นับจำนวนพนักงานไม่ซ้ำ และคำนวณผลรวมเงินเดือนของพนักงานที่มีเงินเดือนมากกว่า 55,000
- แสดงเฉพาะแผนกที่มีผลรวมเงินเดือนสูง (> 100,000)
- **ผลลัพธ์**:
  ```
  department_id | unique_employees | high_salary_total
  --------------+-----------------+------------------
  20            | 2               | 125000
  ```
---
