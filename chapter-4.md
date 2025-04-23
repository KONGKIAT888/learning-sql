# บทที่ 4: การอ่านข้อมูล (SELECT)

## สารบัญ

- [โครงสร้างพื้นฐานของ SELECT](#โครงสร้างพื้นฐานของ-select)
- [การใช้ WHERE เพื่อกรองข้อมูล](#การใช้-where-เพื่อกรองข้อมูล)
- [การใช้ ORDER BY เพื่อเรียงลำดับข้อมูล](#การใช้-order-by-เพื่อเรียงลำดับข้อมูล)
- [การใช้ DISTINCT เพื่อไม่ให้ข้อมูลซ้ำ](#การใช้-distinct-เพื่อไม่ให้ข้อมูลซ้ำ)

## โครงสร้างพื้นฐานของ `SELECT`
คำสั่ง **SELECT** ใน SQL ใช้สำหรับ**ดึงข้อมูล**จากตารางในฐานข้อมูล เป็นส่วนหนึ่งของ DML (Data Manipulation Language) และเป็นคำสั่งหลักสำหรับการสอบถาม (Query) ข้อมูล คำสั่งนี้มีความยืดหยุ่นสูง สามารถดึงข้อมูลจากคอลัมน์เฉพาะ, กรองข้อมูลตามเงื่อนไข, รวมตาราง, หรือคำนวณผลลัพธ์ได้

### ไวยากรณ์พื้นฐาน:
```sql
SELECT column1, column2, ...
FROM table_name
[WHERE condition]
[GROUP BY column]
[HAVING condition]
[ORDER BY column [ASC | DESC]];
```

### อธิบายส่วนประกอบ:
1. **SELECT**: ระบุคอลัมน์หรือนิพจน์ที่ต้องการดึงข้อมูล
    - สามารถใช้ `*` เพื่อเลือกทุกคอลัมน์
    - รองรับนิพจน์ เช่น `column1 + column2`, ฟังก์ชัน เช่น `COUNT(*)`, หรือนามแฝง (Alias) เช่น `column1 AS alias_name`
2. **FROM**: ระบุตารางที่ต้องการดึงข้อมูล
    - สามารถรวมหลายตารางด้วยการใช้ `JOIN`
3. **WHERE**: เงื่อนไขสำหรับกรองแถว (ถ้ามี)
    - เช่น `WHERE salary > 50000` หรือ `WHERE hire_date = '2023-01-01'`
4. **GROUP BY**: จัดกลุ่มข้อมูลตามคอลัมน์ที่ระบุ มักใช้กับฟังก์ชันรวม (Aggregate Functions) เช่น `COUNT`, `SUM`, `AVG`
5. **HAVING**: กรองกลุ่มที่สร้างโดย `GROUP BY` (คล้าย `WHERE` แต่ใช้กับกลุ่ม)
6. **ORDER BY**: จัดเรียงผลลัพธ์ตามคอลัมน์ที่ระบุ
    - `ASC` (ค่าเริ่มต้น): เรียงจากน้อยไปมาก
    - `DESC`: เรียงจากมากไปน้อย

### ลำดับการประมวลผล:
SQL ประมวลผลคำสั่ง `SELECT` ตามลำดับต่อไปนี้ (ไม่ใช่ลำดับในไวยากรณ์):
1. **FROM**: เลือกตารางและรวมตาราง (ถ้ามี `JOIN`)
2. **WHERE**: กรองแถวตามเงื่อนไข
3. **GROUP BY**: จัดกลุ่มข้อมูล
4. **HAVING**: กรองกลุ่ม
5. **SELECT**: เลือกคอลัมน์หรือคำนวณนิพจน์
6. **ORDER BY**: จัดเรียงผลลัพธ์

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees` ดังนี้:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE,
    salary NUMBER(10,2),
    hire_date DATE,
    department_id NUMBER(5)
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 'Doe', 'john.doe@example.com', 50000.00, TO_DATE('2023-01-15', 'YYYY-MM-DD'), 10);
INSERT INTO employees VALUES (2, 'Jane', 'Smith', 'jane.smith@example.com', 60000.00, TO_DATE('2023-02-10', 'YYYY-MM-DD'), 20);
INSERT INTO employees VALUES (3, 'Alice', 'Brown', 'alice.brown@example.com', 55000.00, TO_DATE('2023-03-01', 'YYYY-MM-DD'), 10);
COMMIT;
```

1. **เลือกทุกคอลัมน์**:
   ```sql
   SELECT *
   FROM employees;
   ```
    - ดึงข้อมูลทุกคอลัมน์และทุกแถวจากตาราง `employees`
    - **ผลลัพธ์**:
      ```
      employee_id | first_name | last_name | email                    | salary   | hire_date  | department_id
      ------------+------------+-----------+--------------------------+----------+------------+--------------
      1           | John       | Doe       | john.doe@example.com     | 50000.00 | 2023-01-15 | 10
      2           | Jane       | Smith     | jane.smith@example.com   | 60000.00 | 2023-02-10 | 20
      3           | Alice      | Brown     | alice.brown@example.com  | 55000.00 | 2023-03-01 | 10
      ```

2. **เลือกคอลัมน์เฉพาะ**:
   ```sql
   SELECT first_name, last_name, salary
   FROM employees;
   ```
    - ดึงเฉพาะ `first_name`, `last_name`, และ `salary`
    - **ผลลัพธ์**:
      ```
      first_name | last_name | salary
      ------------+-----------+--------
      John       | Doe       | 50000.00
      Jane       | Smith     | 60000.00
      Alice      | Brown     | 55000.00
      ```

3. **ใช้ WHERE เพื่อกรองข้อมูล**:
   ```sql
   SELECT first_name, salary
   FROM employees
   WHERE salary > 55000;
   ```
    - ดึงชื่อและเงินเดือนของพนักงานที่มีเงินเดือนมากกว่า 55,000
    - **ผลลัพธ์**:
      ```
      first_name | salary
      ------------+--------
      Jane       | 60000.00
      ```

4. **ใช้ ORDER BY เพื่อจัดเรียง**:
   ```sql
   SELECT first_name, salary
   FROM employees
   ORDER BY salary DESC;
   ```
    - จัดเรียงผลลัพธ์ตาม `salary` จากมากไปน้อย
    - **ผลลัพธ์**:
      ```
      first_name | salary
      ------------+--------
      Jane       | 60000.00
      Alice      | 55000.00
      John       | 50000.00
      ```

5. **ใช้ GROUP BY และ Aggregate Functions**:
   ```sql
   SELECT department_id, COUNT(*) AS employee_count, AVG(salary) AS avg_salary
   FROM employees
   GROUP BY department_id;
   ```
    - นับจำนวนพนักงานและคำนวณเงินเดือนเฉลี่ยในแต่ละแผนก
    - **ผลลัพธ์**:
      ```
      department_id | employee_count | avg_salary
      --------------+---------------+------------
      10            | 2             | 52500.00
      20            | 1             | 60000.00
      ```

6. **ใช้ HAVING เพื่อกรองกลุ่ม**:
   ```sql
   SELECT department_id, COUNT(*) AS employee_count
   FROM employees
   GROUP BY department_id
   HAVING COUNT(*) > 1;
   ```
    - แสดงเฉพาะแผนกที่มีพนักงานมากกว่า 1 คน
    - **ผลลัพธ์**:
      ```
      department_id | employee_count
      --------------+---------------
      10            | 2
      ```

7. **ใช้ Alias และนิพจน์**:
   ```sql
   SELECT first_name || ' ' || last_name AS full_name, salary * 1.10 AS new_salary
   FROM employees;
   ```
    - รวม `first_name` และ `last_name` เป็นชื่อเต็ม และคำนวณเงินเดือนใหม่ (เพิ่ม 10%)
    - **ผลลัพธ์**:
      ```
      full_name      | new_salary
      ----------------+------------
      John Doe       | 55000.00
      Jane Smith     | 66000.00
      Alice Brown    | 60500.00
      ```

### ข้อควรระวัง:
- **ประสิทธิภาพ**: การใช้ `SELECT *` หรือการสอบถามข้อมูลจำนวนมากอาจทำให้ประสิทธิภาพลดลง ควรระบุคอลัมน์ที่ต้องการและใช้เงื่อนไข `WHERE` เพื่อจำกัดข้อมูล
- **WHERE กับ NULL**: การเปรียบเทียบกับ `NULL` ต้องใช้ `IS NULL` หรือ `IS NOT NULL` ไม่ใช่ `= NULL`
  ```sql
  SELECT * FROM employees WHERE email IS NULL;
  ```
- **GROUP BY**: คอลัมน์ใน `SELECT` ที่ไม่ใช่ Aggregate Function ต้องอยู่ใน `GROUP BY`
- **ORDER BY**: หากไม่ระบุ `ASC` หรือ `DESC` ค่าเริ่มต้นคือ `ASC`
- **ข้อจำกัดของระบบ**: ฟังก์ชันหรือรูปแบบอาจแตกต่างกัน เช่น การรวมสตริงใช้ `||` ใน Oracle แต่ใช้ `CONCAT` หรือ `+` ในระบบอื่น

### การใช้งานใน PL/SQL:
สามารถใช้ `SELECT` ใน PL/SQL เพื่อดึงข้อมูลลงในตัวแปรหรือประมวลผล เช่น:
```sql
DECLARE
   v_name VARCHAR2(100);
   v_salary NUMBER(10,2);
BEGIN
   SELECT first_name || ' ' || last_name, salary
   INTO v_name, v_salary
   FROM employees
   WHERE employee_id = 1;
   
   DBMS_OUTPUT.PUT_LINE('Name: ' || v_name || ', Salary: ' || v_salary);
EXCEPTION
   WHEN NO_DATA_FOUND THEN
      DBMS_OUTPUT.PUT_LINE('Employee not found.');
   WHEN TOO_MANY_ROWS THEN
      DBMS_OUTPUT.PUT_LINE('More than one employee found.');
END;
/
```
- ใช้ `INTO` เพื่อเก็บผลลัพธ์ในตัวแปร
- จัดการข้อผิดพลาด เช่น `NO_DATA_FOUND` (ไม่มีข้อมูล) หรือ `TOO_MANY_ROWS` (ผลลัพธ์มากกว่า 1 แถว)

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการรายชื่อพนักงานในแผนก 10 ที่มีเงินเดือนมากกว่า 50,000 และจัดเรียงตามเงินเดือน:
```sql
SELECT first_name, last_name, salary
FROM employees
WHERE department_id = 10 AND salary > 50000
ORDER BY salary DESC;
```
- **ผลลัพธ์**:
  ```
  first_name | last_name | salary
  ------------+-----------+--------
  Alice      | Brown     | 55000.00
  ```
---

## การใช้ `WHERE` เพื่อกรองข้อมูล
ใน SQL คำสั่ง **WHERE** ใช้สำหรับ**กรองข้อมูล**ในตารางโดยเลือกเฉพาะแถวที่ตรงตามเงื่อนไขที่กำหนด มันเป็นส่วนสำคัญของคำสั่ง `SELECT`, `UPDATE`, `DELETE` และอื่น ๆ เพื่อจำกัดผลลัพธ์หรือการดำเนินการให้เฉพาะเจาะจงตามความต้องการ

### ไวยากรณ์พื้นฐาน:
```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition;

UPDATE table_name
SET column = value
WHERE condition;

DELETE FROM table_name
WHERE condition;
```

### อธิบายส่วนประกอบ:
- **condition**: เงื่อนไขที่ใช้กรองข้อมูล อาจประกอบด้วย:
    - **ตัวดำเนินการเปรียบเทียบ**: `=`, `<>`, `>`, `<`, `>=`, `<=`
    - **ตัวดำเนินการเชิงตรรกะ**: `AND`, `OR`, `NOT`
    - **ตัวดำเนินการพิเศษ**: `LIKE`, `IN`, `BETWEEN`, `IS NULL`, `IS NOT NULL`
    - **Subquery**: การใช้คำสั่ง `SELECT` ซ้อนภายในเงื่อนไข

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees` ดังนี้:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE,
    salary NUMBER(10,2),
    hire_date DATE,
    department_id NUMBER(5)
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 'Doe', 'john.doe@example.com', 50000.00, TO_DATE('2023-01-15', 'YYYY-MM-DD'), 10);
INSERT INTO employees VALUES (2, 'Jane', 'Smith', 'jane.smith@example.com', 60000.00, TO_DATE('2023-02-10', 'YYYY-MM-DD'), 20);
INSERT INTO employees VALUES (3, 'Alice', 'Brown', 'alice.brown@example.com', 55000.00, TO_DATE('2023-03-01', 'YYYY-MM-DD'), 10);
INSERT INTO employees VALUES (4, 'Bob', 'Wilson', 'bob.wilson@example.com', 65000.00, TO_DATE('2023-04-01', 'YYYY-MM-DD'), 20);
COMMIT;
```

#### 1. การใช้ตัวดำเนินการเปรียบเทียบ:
```sql
SELECT first_name, salary
FROM employees
WHERE salary > 55000;
```
- ดึงชื่อและเงินเดือนของพนักงานที่มีเงินเดือนมากกว่า 55,000
- **ผลลัพธ์**:
  ```
  first_name | salary
  ------------+--------
  Jane       | 60000.00
  Bob        | 65000.00
  ```

#### 2. การใช้ AND/OR:
```sql
SELECT first_name, department_id, hire_date
FROM employees
WHERE department_id = 10 AND hire_date < TO_DATE('2023-02-01', 'YYYY-MM-DD');
```
- ดึงพนักงานในแผนก 10 ที่เริ่มงานก่อน 1 ก.พ. 2023
- **ผลลัพธ์**:
  ```
  first_name | department_id | hire_date
  ------------+---------------+------------
  John       | 10            | 2023-01-15
  ```

#### 3. การใช้ LIKE สำหรับการค้นหาด้วยรูปแบบ:
```sql
SELECT first_name, email
FROM employees
WHERE email LIKE 'j%.com';
```
- ดึงพนักงานที่มี `email` ขึ้นต้นด้วย 'j' และลงท้ายด้วย '.com'
- **ผลลัพธ์**:
  ```
  first_name | email
  ------------+------------------------
  John       | john.doe@example.com
  Jane       | jane.smith@example.com
  ```
- **รูปแบบของ LIKE**:
    - `%`: แทนตัวอักษรใด ๆ จำนวนเท่าใดก็ได้
    - `_`: แทนตัวอักษรหนึ่งตัว

#### 4. การใช้ IN:
```sql
SELECT first_name, department_id
FROM employees
WHERE department_id IN (10, 20);
```
- ดึงพนักงานที่อยู่ในแผนก 10 หรือ 20
- **ผลลัพธ์**:
  ```
  first_name | department_id
  ------------+---------------
  John       | 10
  Jane       | 20
  Alice      | 10
  Bob        | 20
  ```

#### 5. การใช้ BETWEEN:
```sql
SELECT first_name, salary
FROM employees
WHERE salary BETWEEN 50000 AND 60000;
```
- ดึงพนักงานที่มีเงินเดือนระหว่าง 50,000 ถึง 60,000 (รวมขอบเขต)
- **ผลลัพธ์**:
  ```
  first_name | salary
  ------------+--------
  John       | 50000.00
  Jane       | 60000.00
  Alice      | 55000.00
  ```

#### 6. การใช้ IS NULL / IS NOT NULL:
```sql
SELECT first_name, email
FROM employees
WHERE email IS NOT NULL;
```
- ดึงพนักงานที่มี `email` ไม่เป็น `NULL`
- **ผลลัพธ์**: (ในกรณีนี้ทุกแถว เพราะ `email` มีค่าและเป็น `UNIQUE`)

#### 7. การใช้ Subquery ใน WHERE:
```sql
SELECT first_name, salary
FROM employees
WHERE department_id = (SELECT department_id FROM departments WHERE department_name = 'Sales');
```
- ดึงพนักงานในแผนก 'Sales' โดยใช้ Subquery เพื่อหา `department_id`
- (สมมติว่ามีตาราง `departments` และ `department_name = 'Sales'` ตรงกับ `department_id = 20`)
- **ผลลัพธ์**:
  ```
  first_name | salary
  ------------+--------
  Jane       | 60000.00
  Bob        | 65000.00
  ```

#### 8. การใช้ WHERE กับ UPDATE:
```sql
UPDATE employees
SET salary = salary + 5000
WHERE department_id = 10;
COMMIT;
```
- เพิ่มเงินเดือน 5,000 ให้พนักงานในแผนก 10

#### 9. การใช้ WHERE กับ DELETE:
```sql
DELETE FROM employees
WHERE hire_date < TO_DATE('2023-02-01', 'YYYY-MM-DD');
COMMIT;
```
- ลบพนักงานที่เริ่มงานก่อน 1 ก.พ. 2023

### ข้อควรระวัง:
- **NULL**: การเปรียบเทียบกับ `NULL` ต้องใช้ `IS NULL` หรือ `IS NOT NULL` ไม่ใช่ `= NULL` หรือ `<> NULL`
  ```sql
  SELECT * FROM employees WHERE email = NULL; -- ผิด จะไม่ให้ผลลัพธ์
  SELECT * FROM employees WHERE email IS NULL; -- ถูกต้อง
  ```
- **ประสิทธิภาพ**: เงื่อนไขที่ซับซ้อนหรือการใช้ `LIKE` กับ `%` ด้านหน้าอาจทำให้การค้นหาช้า ควรใช้ดัชนี (Indexes) ถ้าจำเป็น
- **ความเฉพาะเจาะจง**: เงื่อนไขควรชัดเจนเพื่อป้องกันการเลือกหรือแก้ไขข้อมูลผิด เช่น การลืม `WHERE` ใน `UPDATE` หรือ `DELETE` จะกระทบทุกแถว
- **ตัวพิมพ์ใหญ่-เล็ก**: ในบางระบบ (เช่น Oracle) ข้อมูลในคอลัมน์อาจไวต่อตัวพิมพ์ใหญ่-เล็ก ต้องตรวจสอบก่อนใช้ `LIKE` หรือ `=`
- **วันที่**: ใช้ฟังก์ชัน เช่น `TO_DATE` ใน Oracle เพื่อให้แน่ใจว่ารูปแบบวันที่ถูกต้อง
  ```sql
  WHERE hire_date = TO_DATE('2023-01-15', 'YYYY-MM-DD');
  ```

### การใช้งานใน PL/SQL:
สามารถใช้ `WHERE` ใน PL/SQL เพื่อควบคุมการดึงหรือแก้ไขข้อมูล เช่น:
```sql
DECLARE
   v_name VARCHAR2(100);
   v_salary NUMBER(10,2);
BEGIN
   SELECT first_name || ' ' || last_name, salary
   INTO v_name, v_salary
   FROM employees
   WHERE employee_id = 1;
   
   DBMS_OUTPUT.PUT_LINE('Name: ' || v_name || ', Salary: ' || v_salary);
EXCEPTION
   WHEN NO_DATA_FOUND THEN
      DBMS_OUTPUT.PUT_LINE('Employee not found.');
END;
/
```
- ใช้ `WHERE` เพื่อเลือกพนักงานที่มี `employee_id = 1`
- จัดการข้อผิดพลาดถ้าไม่พบข้อมูล

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการหาพนักงานในแผนก 20 ที่มีเงินเดือนมากกว่า 60,000 และเริ่มงานหลัง 1 มี.ค. 2023:
```sql
SELECT first_name, last_name, salary, hire_date
FROM employees
WHERE department_id = 20 
  AND salary > 60000
  AND hire_date > TO_DATE('2023-03-01', 'YYYY-MM-DD');
```
- **ผลลัพธ์**:
  ```
  first_name | last_name | salary   | hire_date
  ------------+-----------+----------+------------
  Bob        | Wilson    | 65000.00 | 2023-04-01
  ```
---

## การใช้ `ORDER BY` เพื่อเรียงลำดับข้อมูล
ใน SQL คำสั่ง **ORDER BY** ใช้สำหรับ**เรียงลำดับข้อมูล**ที่ได้จากคำสั่ง `SELECT` ตามคอลัมน์ที่ระบุ สามารถเรียงลำดับจากน้อยไปมาก (Ascending) หรือจากมากไปน้อย (Descending) และสามารถเรียงลำดับตามหลายคอลัมน์ได้ คำสั่งนี้ช่วยให้ผลลัพธ์แสดงในลำดับที่ต้องการ อ่านง่าย และเหมาะสมกับการวิเคราะห์ข้อมูล

### ไวยากรณ์พื้นฐาน:
```sql
SELECT column1, column2, ...
FROM table_name
[WHERE condition]
ORDER BY column1 [ASC | DESC], column2 [ASC | DESC], ...;
```

### อธิบายส่วนประกอบ:
- **ORDER BY**: ระบุคอลัมน์หรือนิพจน์ที่ใช้ในการเรียงลำดับ
- **ASC**: เรียงลำดับจากน้อยไปมาก (ค่าเริ่มต้น หากไม่ระบุ)
- **DESC**: เรียงลำดับจากมากไปน้อย
- **column1, column2, ...**: คอลัมน์ที่ใช้เรียงลำดับ สามารถระบุได้หลายคอลัมน์ โดยลำดับความสำคัญขึ้นอยู่กับลำดับการระบุ
- สามารถใช้:
    - ชื่อคอลัมน์ (เช่น `salary`)
    - ลำดับคอลัมน์ใน `SELECT` (เช่น `1` สำหรับคอลัมน์แรก)
    - นิพจน์ (เช่น `salary * 1.10`)
    - นามแฝง (Alias) เช่น `SELECT first_name AS name ... ORDER BY name`

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees` ดังนี้:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    salary NUMBER(10,2),
    hire_date DATE,
    department_id NUMBER(5)
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 'Doe', 50000.00, TO_DATE('2023-01-15', 'YYYY-MM-DD'), 10);
INSERT INTO employees VALUES (2, 'Jane', 'Smith', 60000.00, TO_DATE('2023-02-10', 'YYYY-MM-DD'), 20);
INSERT INTO employees VALUES (3, 'Alice', 'Brown', 55000.00, TO_DATE('2023-03-01', 'YYYY-MM-DD'), 10);
INSERT INTO employees VALUES (4, 'Bob', 'Wilson', 65000.00, TO_DATE('2023-04-01', 'YYYY-MM-DD'), 20);
COMMIT;
```

#### 1. เรียงลำดับตามคอลัมน์เดียว (ASC):
```sql
SELECT first_name, salary
FROM employees
ORDER BY salary;
```
- เรียงลำดับตาม `salary` จากน้อยไปมาก (ค่าเริ่มต้นคือ `ASC`)
- **ผลลัพธ์**:
  ```
  first_name | salary
  ------------+--------
  John       | 50000.00
  Alice      | 55000.00
  Jane       | 60000.00
  Bob        | 65000.00
  ```

#### 2. เรียงลำดับตามคอลัมน์เดียว (DESC):
```sql
SELECT first_name, salary
FROM employees
ORDER BY salary DESC;
```
- เรียงลำดับตาม `salary` จากมากไปน้อย
- **ผลลัพธ์**:
  ```
  first_name | salary
  ------------+--------
  Bob        | 65000.00
  Jane       | 60000.00
  Alice      | 55000.00
  John       | 50000.00
  ```

#### 3. เรียงลำดับตามหลายคอลัมน์:
```sql
SELECT department_id, first_name, salary
FROM employees
ORDER BY department_id ASC, salary DESC;
```
- เรียงลำดับตาม `department_id` จากน้อยไปมากก่อน แล้วในแต่ละแผนกเรียงตาม `salary` จากมากไปน้อย
- **ผลลัพธ์**:
  ```
  department_id | first_name | salary
  --------------+------------+--------
  10            | Alice      | 55000.00
  10            | John       | 50000.00
  20            | Bob        | 65000.00
  20            | Jane       | 60000.00
  ```

#### 4. ใช้ลำดับคอลัมน์ใน SELECT:
```sql
SELECT first_name, salary
FROM employees
ORDER BY 2 DESC;
```
- `2` หมายถึงคอลัมน์ที่สองใน `SELECT` (คือ `salary`) เรียงจากมากไปน้อย
- **ผลลัพธ์**: เดียวกับตัวอย่างที่ 2

#### 5. ใช้ Alias ใน ORDER BY:
```sql
SELECT first_name || ' ' || last_name AS full_name, salary
FROM employees
ORDER BY full_name ASC;
```
- เรียงลำดับตามนามแฝง `full_name` จากน้อยไปมาก
- **ผลลัพธ์**:
  ```
  full_name      | salary
  ----------------+--------
  Alice Brown    | 55000.00
  Bob Wilson     | 65000.00
  Jane Smith     | 60000.00
  John Doe       | 50000.00
  ```

#### 6. ใช้นิพจน์ใน ORDER BY:
```sql
SELECT first_name, salary
FROM employees
ORDER BY salary * 1.10 DESC;
```
- เรียงลำดับตาม `salary` ที่เพิ่มขึ้น 10% จากมากไปน้อย
- **ผลลัพธ์**:
  ```
  first_name | salary
  ------------+--------
  Bob        | 65000.00
  Jane       | 60000.00
  Alice      | 55000.00
  John       | 50000.00
  ```

#### 7. รวมกับ WHERE:
```sql
SELECT first_name, salary, hire_date
FROM employees
WHERE department_id = 20
ORDER BY hire_date ASC;
```
- ดึงพนักงานในแผนก 20 และเรียงลำดับตาม `hire_date` จากเก่าไปใหม่
- **ผลลัพธ์**:
  ```
  first_name | salary   | hire_date
  ------------+----------+------------
  Jane       | 60000.00 | 2023-02-10
  Bob        | 65000.00 | 2023-04-01
  ```

### ข้อควรระวัง:
- **ค่า NULL**: ในระบบส่วนใหญ่ (เช่น Oracle):
    - `NULL` จะปรากฏท้ายเมื่อเรียงแบบ `ASC`
    - `NULL` จะปรากฏต้นเมื่อเรียงแบบ `DESC`
    - สามารถควบคุมได้ด้วย `NULLS FIRST` หรือ `NULLS LAST` (ใน Oracle, PostgreSQL):
      ```sql
      ORDER BY salary DESC NULLS LAST;
      ```
- **ประสิทธิภาพ**: การเรียงลำดับข้อมูลจำนวนมากอาจใช้ทรัพยากรเยอะ ควรใช้ดัชนี (Indexes) บนคอลัมน์ที่ใช้ใน `ORDER BY` เพื่อเพิ่มประสิทธิภาพ
- **ชื่อคอลัมน์ซ้ำ**: ถ้ามีการ `JOIN` ตารางและคอลัมน์ชื่อซ้ำ ต้องระบุชื่อตาราง เช่น `employees.salary`
- **ตัวพิมพ์ใหญ่-เล็ก**: การเรียงลำดับสตริงอาจขึ้นอยู่กับการตั้งค่าระบบ (Case-sensitive หรือไม่)
- **ลำดับคอลัมน์**: การระบุหลายคอลัมน์ใน `ORDER BY` จะเรียงตามลำดับความสำคัญจากซ้ายไปขวา
- **ระบบฐานข้อมูล**: ไวยากรณ์พื้นฐานเหมือนกัน แต่บางระบบอาจมีคุณสมบัติเพิ่มเติม เช่น MySQL รองรับ `ORDER BY FIELD()` สำหรับกำหนดลำดับเอง

### การใช้งานใน PL/SQL:
สามารถใช้ `ORDER BY` ใน PL/SQL เพื่อดึงข้อมูลที่เรียงลำดับแล้ว เช่น:
```sql
DECLARE
   CURSOR emp_cursor IS
      SELECT first_name, salary
      FROM employees
      WHERE department_id = 10
      ORDER BY salary DESC;
BEGIN
   FOR emp IN emp_cursor LOOP
      DBMS_OUTPUT.PUT_LINE('Name: ' || emp.first_name || ', Salary: ' || emp.salary);
   END LOOP;
END;
/
```
- ใช้ Cursor เพื่อดึงข้อมูลพนักงานในแผนก 10 และเรียงตาม `salary` จากมากไปน้อย
- **ผลลัพธ์**:
  ```
  Name: Alice, Salary: 55000
  Name: John, Salary: 50000
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการแสดงรายชื่อพนักงานทั้งหมด โดยเรียงลำดับตามแผนกและเงินเดือน (มากไปน้อย):
```sql
SELECT department_id, first_name, last_name, salary
FROM employees
ORDER BY department_id ASC, salary DESC;
```
- **ผลลัพธ์**:
  ```
  department_id | first_name | last_name | salary
  --------------+------------+-----------+--------
  10            | Alice      | Brown     | 55000.00
  10            | John       | Doe       | 50000.00
  20            | Bob        | Wilson    | 65000.00
  20            | Jane       | Smith     | 60000.00
  ```
---

## การใช้ `DISTINCT` เพื่อไม่ให้ข้อมูลซ้ำ
ใน SQL คำสั่ง **DISTINCT** ใช้สำหรับ**กำจัดข้อมูลที่ซ้ำกัน**ในผลลัพธ์ของคำสั่ง `SELECT` โดยจะเลือกเฉพาะแถวที่มีค่าที่ไม่ซ้ำกันในคอลัมน์หรือชุดคอลัมน์ที่ระบุ ทำให้ผลลัพธ์มีเพียงค่าเฉพาะ (unique values) เท่านั้น คำสั่งนี้มีประโยชน์เมื่อต้องการดูข้อมูลที่ไม่ซ้ำ เช่น รายการแผนกหรือประเภทสินค้า

### ไวยากรณ์พื้นฐาน:
```sql
SELECT DISTINCT column1, column2, ...
FROM table_name
[WHERE condition]
[ORDER BY column];
```

### อธิบายส่วนประกอบ:
- **DISTINCT**: ระบุว่าต้องการเฉพาะค่าที่ไม่ซ้ำกัน
- **column1, column2, ...**: คอลัมน์ที่ต้องการตรวจสอบความซ้ำซ้อน
    - ถ้าระบุหลายคอลัมน์ `DISTINCT` จะพิจารณาความซ้ำซ้อนของชุดค่าทั้งหมด
- **FROM**: ตารางที่ดึงข้อมูล
- **WHERE**: เงื่อนไขสำหรับกรองข้อมูลก่อนตรวจสอบความซ้ำซ้อน
- **ORDER BY**: เรียงลำดับผลลัพธ์ (ถ้ามี)

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees` ดังนี้:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    salary NUMBER(10,2),
    department_id NUMBER(5)
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 'Doe', 50000.00, 10);
INSERT INTO employees VALUES (2, 'Jane', 'Smith', 60000.00, 20);
INSERT INTO employees VALUES (3, 'Alice', 'Brown', 55000.00, 10);
INSERT INTO employees VALUES (4, 'Bob', 'Wilson', 65000.00, 20);
INSERT INTO employees VALUES (5, 'Carol', 'Davis', 50000.00, 10);
COMMIT;
```

#### 1. เลือกค่าไม่ซ้ำในคอลัมน์เดียว:
```sql
SELECT DISTINCT department_id
FROM employees;
```
- ดึงรหัสแผนก (`department_id`) ที่ไม่ซ้ำกัน
- **ผลลัพธ์**:
  ```
  department_id
  --------------
  10
  20
  ```
- แม้ว่า `department_id` จะปรากฏหลายครั้ง (10 สามครั้ง, 20 สองครั้ง) `DISTINCT` ทำให้แสดงเพียงค่าไม่ซ้ำ

#### 2. เลือกค่าไม่ซ้ำในหลายคอลัมน์:
```sql
SELECT DISTINCT department_id, salary
FROM employees;
```
- ดึงชุดค่า `department_id` และ `salary` ที่ไม่ซ้ำกัน
- **ผลลัพธ์**:
  ```
  department_id | salary
  --------------+--------
  10            | 50000.00
  10            | 55000.00
  20            | 60000.00
  20            | 65000.00
  ```
- `DISTINCT` พิจารณาความซ้ำของทั้งสองคอลัมน์รวมกัน ดังนั้น `(10, 50000.00)` ปรากฏเพียงครั้งเดียว

#### 3. รวมกับ WHERE:
```sql
SELECT DISTINCT salary
FROM employees
WHERE department_id = 10;
```
- ดึงเงินเดือนที่ไม่ซ้ำกันของพนักงานในแผนก 10
- **ผลลัพธ์**:
  ```
  salary
  --------
  50000.00
  55000.00
  ```
- กรองพนักงานในแผนก 10 ก่อน แล้วเลือก `salary` ที่ไม่ซ้ำ

#### 4. รวมกับ ORDER BY:
```sql
SELECT DISTINCT department_id
FROM employees
ORDER BY department_id ASC;
```
- ดึงรหัสแผนกที่ไม่ซ้ำและเรียงลำดับจากน้อยไปมาก
- **ผลลัพธ์**:
  ```
  department_id
  --------------
  10
  20
  ```

#### 5. ใช้ DISTINCT กับฟังก์ชัน:
```sql
SELECT DISTINCT ROUND(salary, -4) AS rounded_salary
FROM employees;
```
- ปัดเศษเงินเดือนให้เป็นหลักหมื่นและเลือกค่าไม่ซ้ำ
- **ผลลัพธ์**:
  ```
  rounded_salary
  --------------
  50000
  60000
  70000
  ```
- เงินเดือน 50000, 55000 ปัดเป็น 50000; 60000 ปัดเป็น 60000; 65000 ปัดเป็น 70000

### ข้อควรระวัง:
- **ประสิทธิภาพ**: การใช้ `DISTINCT` อาจทำให้ประสิทธิภาพลดลงในตารางขนาดใหญ่ เนื่องจากต้องตรวจสอบและกำจัดข้อมูลที่ซ้ำกัน ควรพิจารณาใช้ดัชนี (Indexes) หรือวิธีอื่น เช่น `GROUP BY` ถ้าเหมาะสม
- **หลายคอลัมน์**: `DISTINCT` ใช้กับชุดค่าทั้งหมด ไม่ใช่แยกคอลัมน์ เช่น:
  ```sql
  SELECT DISTINCT department_id, salary FROM employees;
  ```
  จะไม่กำจัดความซ้ำของ `department_id` หรือ `salary` แยกกัน แต่พิจารณาคู่ `(department_id, salary)`
- **NULL**: `DISTINCT` ถือว่า `NULL` เป็นค่าเดียว ดังนั้น `NULL` จะปรากฏเพียงครั้งเดียวในผลลัพธ์
  ```sql
  SELECT DISTINCT email FROM employees WHERE email IS NULL;
  ```
  ถ้ามีหลายแถวที่ `email` เป็น `NULL` จะแสดง `NULL` เพียงครั้งเดียว
- **การใช้งานกับ Aggregate Functions**: `DISTINCT` สามารถใช้ในฟังก์ชันรวม เช่น `COUNT(DISTINCT column)` เพื่อนับจำนวนค่าไม่ซ้ำ
  ```sql
  SELECT COUNT(DISTINCT department_id) AS unique_depts
  FROM employees;
  ```
    - **ผลลัพธ์**:
      ```
      unique_depts
      ------------
      2
      ```

### การใช้งานใน PL/SQL:
สามารถใช้ `DISTINCT` ใน PL/SQL เพื่อดึงข้อมูลที่ไม่ซ้ำลงในตัวแปรหรือ Cursor เช่น:
```sql
DECLARE
   CURSOR dept_cursor IS
      SELECT DISTINCT department_id
      FROM employees
      ORDER BY department_id;
BEGIN
   FOR dept IN dept_cursor LOOP
      DBMS_OUTPUT.PUT_LINE('Department ID: ' || dept.department_id);
   END LOOP;
END;
/
```
- ดึงรหัสแผนกที่ไม่ซ้ำและแสดงผล
- **ผลลัพธ์**:
  ```
  Department ID: 10
  Department ID: 20
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการหาเงินเดือนที่ไม่ซ้ำกันของพนักงานในทุกแผนก และเรียงลำดับจากมากไปน้อย:
```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC;
```
- **ผลลัพธ์**:
  ```
  salary
  --------
  65000.00
  60000.00
  55000.00
  50000.00
  ```

หรือต้องการนับจำนวนแผนกที่ไม่ซ้ำ:
```sql
SELECT COUNT(DISTINCT department_id) AS unique_departments
FROM employees;
```
- **ผลลัพธ์**:
  ```
  unique_departments
  ------------------
  2
  ```

### ความแตกต่างกับ GROUP BY:
- **DISTINCT**: ใช้กำจัดแถวที่ซ้ำกันในผลลัพธ์โดยไม่ต้องคำนวณอะไรเพิ่มเติม
- **GROUP BY**: ใช้จัดกลุ่มข้อมูลและมักใช้กับ Aggregate Functions เช่น `COUNT`, `SUM`
    - ตัวอย่างเทียบเท่า:
      ```sql
      SELECT DISTINCT department_id FROM employees;
      ```
      เทียบกับ:
      ```sql
      SELECT department_id FROM employees GROUP BY department_id;
      ```
      ทั้งสองให้ผลลัพธ์เดียวกัน แต่ `DISTINCT` อาจอ่านง่ายกว่าสำหรับกรณีที่ไม่ต้องการการคำนวณ

### ข้อจำกัดในระบบฐานข้อมูล:
- **Oracle, MySQL, PostgreSQL, SQL Server**: รองรับ `DISTINCT` ในลักษณะเดียวกัน
- **MySQL**: รองรับ `DISTINCTROW` (หรือ `ALL`) ในบางกรณีเพื่อกำจัดแถวทั้งแถวที่ซ้ำกัน
- **ประสิทธิภาพ**: ระบบฐานข้อมูลบางตัวอาจจัดการ `DISTINCT` แตกต่างกัน ควรตรวจสอบการใช้ดัชนีหรือการปรับแต่ง Query
---
