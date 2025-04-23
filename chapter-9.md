# บทที่ 9: View, Index และ Sequence
## การสร้าง `VIEW` เพื่อแสดงข้อมูลเฉพาะ
ใน SQL คำสั่ง **VIEW** คือ**ตารางเสมือน** (Virtual Table) ที่สร้างจากผลลัพธ์ของคำสั่ง `SELECT` ซึ่งช่วยให้สามารถแสดงข้อมูลเฉพาะจากตารางหรือหลายตารางได้โดยไม่ต้องเก็บข้อมูลจริงในฐานข้อมูล VIEW มีประโยชน์ในการทำให้ Query ซับซ้อนใช้งานง่ายขึ้น, จำกัดการเข้าถึงข้อมูล, หรือนำเสนอข้อมูลในรูปแบบที่กำหนดเอง ต่อไปนี้คือคำอธิบายและตัวอย่างการสร้างและใช้งาน VIEW ในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### คำอธิบายของ VIEW
- **VIEW** คือ Query ที่บันทึกไว้ในฐานข้อมูลและสามารถเรียกใช้เหมือนตาราง
- **ลักษณะเด่น**:
    - ไม่เก็บข้อมูลจริง แต่ดึงข้อมูลจากตารางต้นทาง (Underlying Tables) เมื่อถูกเรียกใช้
    - อัปเดตอัตโนมัติเมื่อข้อมูลในตารางต้นทางเปลี่ยนแปลง
    - สามารถใช้เพื่อจำกัดการเข้าถึงคอลัมน์หรือแถวที่ต้องการ
    - สามารถรวมข้อมูลจากหลายตารางด้วย `JOIN`, `Subquery`, หรือ `WITH`
    - รองรับการอัปเดตข้อมูลในบางกรณี (ถ้า VIEW เป็นแบบ "Updatable")
- **เมื่อใช้ VIEW**:
    - ลดความซับซ้อนของ Query ที่ใช้บ่อย
    - จำกัดการเข้าถึงข้อมูลเพื่อความปลอดภัย (เช่น แสดงเฉพาะบางคอลัมน์)
    - นำเสนอข้อมูลในรูปแบบที่เหมาะสมสำหรับผู้ใช้
    - รวมข้อมูลจากหลายตารางเพื่อการรายงาน

### ไวยากรณ์พื้นฐาน
```sql
CREATE [OR REPLACE] VIEW view_name [(column1, column2, ...)]
AS
    SELECT columns
    FROM table(s)
    [WHERE condition]
    [GROUP BY columns]
    [HAVING condition]
    [WITH CHECK OPTION]
    [WITH READ ONLY];
```

- **CREATE VIEW**: สร้าง VIEW ใหม่
- **OR REPLACE**: ถ้ามี VIEW ชื่อเดียวกันอยู่แล้ว จะแทนที่ด้วย VIEW ใหม่
- **view_name**: ชื่อของ VIEW
- **(column1, column2, ...)**: ชื่อคอลัมน์ของ VIEW (ไม่บังคับ ถ้าไม่ระบุจะใช้ชื่อจาก `SELECT`)
- **AS SELECT**: Query ที่กำหนดข้อมูลใน VIEW
- **WITH CHECK OPTION**: จำกัดการอัปเดตหรือแทรกข้อมูลให้ตรงตามเงื่อนไขของ VIEW
- **WITH READ ONLY**: ทำให้ VIEW อ่านได้อย่างเดียว (ไม่อนุญาตให้อัปเดต)

### ตัวอย่างการใช้งาน
สมมติมีตาราง:
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
INSERT INTO departments VALUES (30, 'HR');
COMMIT;
```

#### 1. สร้าง VIEW พื้นฐาน
```sql
CREATE VIEW emp_sales AS
    SELECT employee_id, first_name, salary
    FROM employees
    WHERE department_id = 10;
```
- **คำอธิบาย**:
    - สร้าง VIEW ชื่อ `emp_sales` เพื่อแสดงข้อมูลพนักงานในแผนก Sales (department_id = 10)
    - รวมเฉพาะคอลัมน์ `employee_id`, `first_name`, `salary`
- **เรียกใช้ VIEW**:
  ```sql
  SELECT * FROM emp_sales;
  ```
- **ผลลัพธ์**:
  ```
  employee_id | first_name | salary
  ------------+------------+--------
  1          | John       | 50000
  3          | Alice      | 55000
  ```
- **อธิบาย**:
    - VIEW แสดงเฉพาะพนักงานในแผนก 10
    - หากข้อมูลใน `employees` เปลี่ยนแปลง ผลลัพธ์ของ VIEW จะอัปเดตอัตโนมัติ

#### 2. สร้าง VIEW ด้วย JOIN
```sql
CREATE OR REPLACE VIEW emp_dept_info AS
    SELECT e.first_name, e.salary, d.department_name
    FROM employees e
    JOIN departments d
    ON e.department_id = d.department_id
    WHERE e.salary > 55000
    WITH READ ONLY;
```
- **คำอธิบาย**:
    - สร้าง VIEW ชื่อ `emp_dept_info` ที่รวมข้อมูลจาก `employees` และ `departments`
    - แสดงเฉพาะพนักงานที่มีเงินเดือนมากกว่า 55,000
    - `WITH READ ONLY` ทำให้ VIEW นี้ไม่สามารถอัปเดตได้
- **เรียกใช้ VIEW**:
  ```sql
  SELECT * FROM emp_dept_info;
  ```
- **ผลลัพธ์**:
  ```
  first_name | salary | department_name
  ------------+--------+----------------
  Jane       | 60000  | IT
  Bob        | 65000  | IT
  ```
- **อธิบาย**:
    - VIEW รวมข้อมูลชื่อพนักงาน, เงินเดือน, และชื่อแผนก
    - กรองเฉพาะพนักงานที่มีเงินเดือน > 55,000

#### 3. สร้าง VIEW ด้วย WITH CHECK OPTION
```sql
CREATE OR REPLACE VIEW emp_high_salary AS
    SELECT employee_id, first_name, salary, department_id
    FROM employees
    WHERE salary > 55000
    WITH CHECK OPTION;
```
- **คำอธิบาย**:
    - สร้าง VIEW ชื่อ `emp_high_salary` สำหรับพนักงานที่มีเงินเดือนมากกว่า 55,000
    - `WITH CHECK OPTION` ตรวจสอบว่า การแทรกหรืออัปเดตข้อมูลผ่าน VIEW ต้องมี `salary > 55000`
- **เรียกใช้ VIEW**:
  ```sql
  SELECT * FROM emp_high_salary;
  ```
- **ผลลัพธ์**:
  ```
  employee_id | first_name | salary | department_id
  ------------+------------+--------+----------------
  2          | Jane       | 60000  | 20
  4          | Bob        | 65000  | 20
  ```
- **ทดลองอัปเดต**:
  ```sql
  UPDATE emp_high_salary
  SET salary = 50000
  WHERE employee_id = 2;
  ```
    - **ผลลัพธ์**: เกิดข้อผิดพลาด (เช่น `ORA-01402: view WITH CHECK OPTION where-clause violation`) เพราะ `salary = 50000` ฝ่าฝืนเงื่อนไข `salary > 55000`
- **ทดลองแทรก**:
  ```sql
  INSERT INTO emp_high_salary (employee_id, first_name, salary, department_id)
  VALUES (6, 'David', 52000, 20);
  ```
    - **ผลลัพธ์**: เกิดข้อผิดพลาดเช่นกัน เพราะ `salary = 52000` ไม่ผ่านเงื่อนไข

#### 4. สร้าง VIEW ด้วย Subquery หรือ CTE
```sql
CREATE OR REPLACE VIEW dept_summary AS
    WITH dept_avg AS (
        SELECT department_id, AVG(salary) AS avg_salary, COUNT(*) AS emp_count
        FROM employees
        GROUP BY department_id
    )
    SELECT d.department_name, da.avg_salary, da.emp_count
    FROM dept_avg da
    JOIN departments d
    ON da.department_id = d.department_id
    WHERE da.emp_count > 1;
```
- **คำอธิบาย**:
    - สร้าง VIEW ชื่อ `dept_summary` ที่ใช้ CTE เพื่อคำนวณค่าเฉลี่ยเงินเดือนและจำนวนพนักงานในแต่ละแผนก
    - แสดงเฉพาะแผนกที่มีพนักงานมากกว่า 1 คน
- **เรียกใช้ VIEW**:
  ```sql
  SELECT * FROM dept_summary;
  ```
- **ผลลัพธ์**:
  ```
  department_name | avg_salary | emp_count
  ----------------+------------+-----------
  Sales          | 52500      | 2
  IT             | 62500      | 2
  ```
- **อธิบาย**:
    - CTE `dept_avg` คำนวณข้อมูลสรุป
    - VIEW รวมชื่อแผนกและกรองแผนกที่มี `emp_count > 1`

#### 5. สร้าง VIEW สำหรับการจำกัดการเข้าถึง
```sql
CREATE OR REPLACE VIEW emp_basic_info AS
    SELECT employee_id, first_name
    FROM employees
    WITH READ ONLY;
```
- **คำอธิบาย**:
    - สร้าง VIEW ชื่อ `emp_basic_info` เพื่อแสดงเฉพาะ `employee_id` และ `first_name`
    - ใช้สำหรับผู้ใช้ที่ไม่ควรเห็นข้อมูล sensitive เช่น `salary` หรือ `department_id`
    - `WITH READ ONLY` ป้องกันการอัปเดต
- **เรียกใช้ VIEW**:
  ```sql
  SELECT * FROM emp_basic_info;
  ```
- **ผลลัพธ์**:
  ```
  employee_id | first_name
  ------------+------------
  1          | John
  2          | Jane
  3          | Alice
  4          | Bob
  5          | Carol
  ```

### การจัดการ VIEW
- **ลบ VIEW**:
  ```sql
  DROP VIEW view_name;
  ```
    - ตัวอย่าง: `DROP VIEW emp_sales;`
    - การลบ VIEW ไม่กระทบตารางต้นทาง
- **ดูโครงสร้าง VIEW**:
    - ใน Oracle:
      ```sql
      SELECT text FROM user_views WHERE view_name = 'EMP_SALES';
      ```
    - หรือใช้เครื่องมือเช่น SQL Developer เพื่อดูคำสั่ง `CREATE VIEW`
- **ตรวจสอบข้อมูลใน VIEW**:
    - ใช้เหมือนตารางปกติ:
      ```sql
      SELECT * FROM emp_dept_info WHERE salary > 60000;
      ```
- **ให้สิทธิ์การเข้าถึง VIEW**:
    - ตัวอย่าง:
      ```sql
      GRANT SELECT ON emp_basic_info TO user_name;
      ```
    - อนุญาตให้ผู้ใช้เฉพาะเห็นข้อมูลใน VIEW

### ข้อควรระวัง
- **ประสิทธิภาพ**:
    - VIEW ไม่เก็บข้อมูล ดังนั้น Query ที่ซับซ้อนใน VIEW อาจช้าเมื่อเรียกใช้
    - ควรใช้ดัชนี (Indexes) บนคอลัมน์ในตารางต้นทางที่ใช้ใน `WHERE` หรือ `JOIN`
    - ถ้าต้องการเก็บผลลัพธ์เพื่อประสิทธิภาพ อาจใช้ **Materialized View** แทน
- **การอัปเดต VIEW**:
    - VIEW สามารถอัปเดตได้ถ้า:
        - มีความสัมพันธ์แบบ 1:1 กับตารางต้นทาง
        - ไม่มี Aggregate Functions (`SUM`, `AVG`), `GROUP BY`, `DISTINCT`, หรือ Subquery ซับซ้อน
        - ไม่มี `JOIN` ที่ซับซ้อนเกินไป
    - `WITH READ ONLY` ป้องกันการอัปเดต
    - `WITH CHECK OPTION` ตรวจสอบเงื่อนไขเมื่ออัปเดตหรือแทรก
- **การตั้งชื่อคอลัมน์**:
    - ถ้า Query ใน VIEW มีการคำนวณ (เช่น `AVG(salary)`) ควรระบุชื่อคอลัมน์ใน `CREATE VIEW ... (column1, column2, ...)`
    - ตัวอย่าง:
      ```sql
      CREATE VIEW dept_avg (dept_id, avg_salary) AS
          SELECT department_id, AVG(salary)
          FROM employees
          GROUP BY department_id;
      ```
- **ขอบเขต**:
    - VIEW ถูกเก็บในฐานข้อมูลและสามารถใช้ได้ในหลาย Query (ต่างจาก CTE ที่จำกัดใน Query เดียว)
- **ระบบฐานข้อมูล**:
    - Oracle, PostgreSQL, SQL Server, MySQL รองรับ VIEW
    - **MySQL**: รองรับ VIEW แต่การอัปเดต VIEW มีข้อจำกัดมากกว่า Oracle
    - **PostgreSQL**: รองรับ **Updatable VIEW** และ **Materialized View** ได้ดี
    - **SQL Server**: คล้าย Oracle แต่ควรระวังการใช้ `WITH CHECK OPTION` ในบางกรณี
- **ความปลอดภัย**:
    - VIEW ช่วยจำกัดการเข้าถึงข้อมูล แต่ต้องกำหนดสิทธิ์ (GRANT) ให้เหมาะสม
    - ถ้าตารางต้นทางถูกลบหรือเปลี่ยนโครงสร้าง VIEW อาจใช้งานไม่ได้

### การเปรียบเทียบ VIEW กับ CTE
| คุณสมบัติ              | VIEW                                | CTE                                  |
|------------------------|-------------------------------------|--------------------------------------|
| **การเก็บข้อมูล**      | เก็บในฐานข้อมูล (ถาวร)              | ชั่วคราวใน Query เดียว              |
| **การใช้งานซ้ำ**       | ใช้ได้ในหลาย Query และผู้ใช้         | ใช้ได้เฉพาะใน Query ที่กำหนด         |
| **การอัปเดต**          | อัปเดตได้ในบางกรณี                  | ไม่สามารถอัปเดต (อ่านอย่างเดียว)     |
| **Recursive**          | ไม่รองรับ Recursive                 | รองรับ Recursive CTE                |
| **การใช้งานทั่วไป**    | จำกัดการเข้าถึง, รายงานถาวร        | Query ซับซ้อนชั่วคราว, ลำดับชั้น   |

### การใช้งานใน PL/SQL
สามารถใช้ VIEW ใน PL/SQL เพื่อดึงข้อมูลหรือจัดการตรรกะ เช่น:
```sql
DECLARE
   CURSOR high_salary_employees IS
      SELECT first_name, salary, department_name
      FROM emp_dept_info
      WHERE salary > 60000;
BEGIN
   FOR emp IN high_salary_employees LOOP
      DBMS_OUTPUT.PUT_LINE('Employee: ' || emp.first_name || 
                           ', Salary: ' || emp.salary || 
                           ', Department: ' || emp.department_name);
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Employee: Bob, Salary: 65000, Department: IT
  ```

### ตัวอย่างสถานการณ์จริง
สมมติต้องการสร้าง VIEW สำหรับทีม HR เพื่อดูข้อมูลพนักงานที่มีเงินเดือนสูงและอยู่ในแผนกที่มีพนักงานมากกว่า 1 คน:

```sql
CREATE OR REPLACE VIEW hr_high_salary_report AS
    WITH dept_counts AS (
        SELECT department_id, COUNT(*) AS emp_count
        FROM employees
        GROUP BY department_id
        HAVING COUNT(*) > 1
    )
    SELECT e.first_name, e.salary, d.department_name, dc.emp_count
    FROM employees e
    JOIN departments d
    ON e.department_id = d.department_id
    JOIN dept_counts dc
    ON e.department_id = dc.department_id
    WHERE e.salary > (SELECT AVG(salary) FROM employees)
    WITH READ ONLY;
```
- **คำอธิบาย**:
    - CTE `dept_counts` หาแผนกที่มีพนักงานมากกว่า 1 คน
    - VIEW รวมข้อมูลชื่อพนักงาน, เงินเดือน, ชื่อแผนก, และจำนวนพนักงานในแผนก
    - กรองเฉพาะพนักงานที่มีเงินเดือนมากกว่าค่าเฉลี่ยรวม
    - `WITH READ ONLY` ป้องกันการแก้ไข
- **เรียกใช้ VIEW**:
  ```sql
  SELECT * FROM hr_high_salary_report;
  ```
- **ผลลัพธ์**:
  ```
  first_name | salary | department_name | emp_count
  ------------+--------+----------------+-----------
  Jane       | 60000  | IT             | 2
  Bob        | 65000  | IT             | 2
  ```
- **อธิบาย**:
    - ค่าเฉลี่ยเงินเดือนรวม = 56400
    - แผนก IT (department_id = 20) มีพนักงาน 2 คน และมี Jane, Bob ที่เงินเดือน > 56400

### การใช้ VIEW กับการควบคุมการเข้าถึง
สมมติต้องการให้ผู้ใช้บางคนเห็นเฉพาะข้อมูลพื้นฐานของพนักงาน:
1. สร้าง VIEW:
   ```sql
   CREATE VIEW emp_public_info AS
       SELECT employee_id, first_name
       FROM employees
       WITH READ ONLY;
   ```
2. ให้สิทธิ์:
   ```sql
   GRANT SELECT ON emp_public_info TO hr_user;
   ```
3. ผู้ใช้ `hr_user` สามารถเรียกใช้:
   ```sql
   SELECT * FROM emp_public_info;
   ```
    - เห็นเฉพาะ `employee_id` และ `first_name` ไม่เห็น `salary` หรือ `department_id`
---

## `INDEX` เพื่อเพิ่มความเร็วในการค้นหา
ใน SQL **INDEX** คือโครงสร้างข้อมูลที่สร้างขึ้นในฐานข้อมูลเพื่อ**เพิ่มความเร็วในการค้นหาและเรียกข้อมูล**จากตาราง โดยเฉพาะเมื่อใช้คำสั่ง `SELECT`, `WHERE`, `JOIN`, `ORDER BY`, หรือ `GROUP BY` INDEX ช่วยลดเวลาการประมวลผลโดยจัดเก็บข้อมูลในลักษณะที่เหมาะสม (เช่น B-Tree หรือ Bitmap) เพื่อให้ฐานข้อมูลสามารถค้นหาแถวได้เร็วขึ้น ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งาน INDEX ในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### คำอธิบายของ INDEX
- **INDEX** คือโครงสร้างที่เก็บข้อมูลของคอลัมน์ที่เลือกไว้ในรูปแบบที่ช่วยให้ค้นหาได้เร็ว (เช่น คล้ายสมุดโทรศัพท์ที่เรียงลำดับตามชื่อ)
- **ลักษณะเด่น**:
    - เพิ่มความเร็วในการค้นหา, กรอง, และเรียงลำดับข้อมูล
    - ลดการสแกนตารางทั้งหมด (Full Table Scan) โดยใช้โครงสร้างเช่น **B-Tree** (สำหรับข้อมูลทั่วไป) หรือ **Bitmap** (สำหรับข้อมูลที่มีความหลากหลายต่ำ)
    - อาจเพิ่มภาระเมื่อมีการอัปเดต, แทรก, หรือลบข้อมูล เพราะ INDEX ต้องได้รับการปรับปรุง
- **เมื่อใช้ INDEX**:
    - คอลัมน์ที่ใช้บ่อยใน `WHERE`, `JOIN`, `ORDER BY`, หรือ `GROUP BY`
    - คอลัมน์ที่มี**ความหลากหลายสูง** (High Cardinality) เช่น `employee_id` หรือ `email`
    - ตารางขนาดใหญ่ที่มีการค้นหาบ่อย
- **เมื่อไม่ควรใช้ INDEX**:
    - ตารางขนาดเล็ก (การสแกนทั้งตารางเร็วกว่า)
    - คอลัมน์ที่มี**ความหลากหลายต่ำ** (Low Cardinality) เช่น `gender` (ชาย/หญิง)
    - คอลัมน์ที่ไม่ค่อยใช้ใน Query
    - ตารางที่มีการอัปเดต, แทรก, หรือลบบ่อย เพราะ INDEX ต้องปรับปรุงทุกครั้ง

### ประเภทของ INDEX
1. **B-Tree Index** (ค่าเริ่มต้น):
    - เหมาะสำหรับคอลัมน์ที่มีความหลากหลายสูง (เช่น `employee_id`, `email`)
    - ใช้โครงสร้างต้นไม้สมดุล (Balanced Tree) เพื่อค้นหาเร็ว
    - รองรับการเปรียบเทียบ (`=`, `>`, `<`, `LIKE`, `BETWEEN`)
2. **Bitmap Index**:
    - เหมาะสำหรับคอลัมน์ที่มีความหลากหลายต่ำ (เช่น `gender`, `status`)
    - ใช้หน่วยความจำน้อยกว่า B-Tree ในบางกรณี
    - เหมาะสำหรับ Data Warehouse ที่มีการอัปเดตน้อย
3. **Unique Index**:
    - บังคับให้ค่าทั้งหมดในคอลัมน์หรือชุดคอลัมน์ไม่ซ้ำกัน
    - สร้างอัตโนมัติเมื่อกำหนด `PRIMARY KEY` หรือ `UNIQUE` Constraint
4. **Composite Index**:
    - สร้างบนคอลัมน์หลายคอลัมน์ (เช่น `first_name` + `last_name`)
    - เหมาะสำหรับ Query ที่ใช้คอลัมน์หลายตัวใน `WHERE` หรือ `JOIN`
5. **Function-Based Index**:
    - สร้างบนผลลัพธ์ของฟังก์ชัน (เช่น `UPPER(first_name)`)
    - ใช้เมื่อ Query มีการใช้ฟังก์ชันใน `WHERE`
6. **Clustered Index** (ในบางระบบ เช่น SQL Server):
    - จัดเรียงข้อมูลในตารางตามคอลัมน์ที่กำหนด (Oracle ไม่มี Clustered Index แบบนี้ แต่มี Index-Organized Table)
7. **Partitioned Index**:
    - แบ่ง INDEX ออกเป็นส่วนย่อย (Partitions) เพื่อจัดการข้อมูลขนาดใหญ่

### ไวยากรณ์พื้นฐาน
```sql
-- สร้าง INDEX
CREATE [UNIQUE | BITMAP] INDEX index_name
ON table_name (column1, column2, ...)
[TABLESPACE tablespace_name];

-- สร้าง Function-Based Index
CREATE INDEX index_name
ON table_name (function(column));

-- ลบ INDEX
DROP INDEX index_name;

-- ดู INDEX ที่มีอยู่ในตาราง
SELECT index_name, column_name
FROM user_ind_columns
WHERE table_name = 'TABLE_NAME';
```

- **index_name**: ชื่อของ INDEX
- **table_name**: ตารางที่ต้องการสร้าง INDEX
- **column1, column2, ...**: คอลัมน์ที่ต้องการสร้าง INDEX
- **TABLESPACE**: พื้นที่จัดเก็บ INDEX (ไม่บังคับ)

### ตัวอย่างการใช้งาน
สมมติมีตาราง:
1. **employees**:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    last_name VARCHAR2(50),
    salary NUMBER(10,2),
    department_id NUMBER(5),
    email VARCHAR2(100)
);
INSERT INTO employees VALUES (1, 'John', 'Doe', 50000, 10, 'john.doe@example.com');
INSERT INTO employees VALUES (2, 'Jane', 'Smith', 60000, 20, 'jane.smith@example.com');
INSERT INTO employees VALUES (3, 'Alice', 'Johnson', 55000, 10, 'alice.johnson@example.com');
INSERT INTO employees VALUES (4, 'Bob', 'Brown', 65000, 20, 'bob.brown@example.com');
INSERT INTO employees VALUES (5, 'Carol', 'Davis', 52000, 30, 'carol.davis@example.com');
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

#### 1. สร้าง B-Tree Index
สมมติ Query นี้ใช้บ่อย:
```sql
SELECT first_name, salary
FROM employees
WHERE department_id = 10;
```
สร้าง INDEX เพื่อเพิ่มความเร็ว:
```sql
CREATE INDEX idx_emp_dept_id ON employees (department_id);
```
- **คำอธิบาย**:
    - สร้าง B-Tree Index ชื่อ `idx_emp_dept_id` บนคอลัมน์ `department_id`
    - ช่วยให้ Query ที่กรองด้วย `department_id` เร็วกว่าเดิม
- **ตรวจสอบการใช้ INDEX**:
    - ใช้ `EXPLAIN PLAN` เพื่อดูว่า Oracle ใช้ INDEX หรือไม่:
      ```sql
      EXPLAIN PLAN FOR
      SELECT first_name, salary
      FROM employees
      WHERE department_id = 10;
      SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
      ```
    - **ผลลัพธ์** (ตัวอย่าง):
      ```
      --------------------------------------------------------------------------------
      | Id  | Operation                   | Name              | Rows  | Bytes | Cost  |
      --------------------------------------------------------------------------------
      |   0 | SELECT STATEMENT           |                   |     2 |    60 |     2 |
      |   1 |  TABLE ACCESS BY INDEX ROWID| EMPLOYEES         |     2 |    60 |     2 |
      |*  2 |   INDEX RANGE SCAN         | IDX_EMP_DEPT_ID   |     2 |       |     1 |
      --------------------------------------------------------------------------------
      ```
    - **อธิบาย**: Oracle ใช้ `INDEX RANGE SCAN` บน `idx_emp_dept_id` ทำให้ค้นหาเร็วขึ้น

#### 2. สร้าง Composite Index
สมมติ Query นี้ใช้บ่อย:
```sql
SELECT first_name, last_name, salary
FROM employees
WHERE department_id = 10 AND salary > 50000
ORDER BY salary;
```
สร้าง Composite Index:
```sql
CREATE INDEX idx_emp_dept_salary ON employees (department_id, salary);
```
- **คำอธิบาย**:
    - สร้าง INDEX บน `department_id` และ `salary`
    - ช่วยให้การกรองด้วย `department_id` และ `salary` รวมถึงการเรียงลำดับ (`ORDER BY salary`) เร็วขึ้น
    - **ลำดับคอลัมน์ใน INDEX** สำคัญ:
        - วางคอลัมน์ที่ใช้ใน `WHERE` บ่อยที่สุด (`department_id`) ไว้ก่อน
        - คอลัมน์ที่ใช้ใน `ORDER BY` หรือ `GROUP BY` ตามมา (`salary`)

#### 3. สร้าง Unique Index
สมมติต้องการให้ `email` ไม่ซ้ำกัน:
```sql
CREATE UNIQUE INDEX idx_emp_email ON employees (email);
```
- **คำอธิบาย**:
    - สร้าง Unique Index เพื่อบังคับให้ `email` ไม่ซ้ำ
    - หากพยายามแทรกข้อมูลที่มี `email` ซ้ำ จะเกิดข้อผิดพลาด (เช่น `ORA-00001: unique constraint violated`)
- **ตัวอย่างการแทรกที่ล้มเหลว**:
  ```sql
  INSERT INTO employees VALUES (6, 'David', 'Wilson', 58000, 10, 'john.doe@example.com');
  ```
    - **ผลลัพธ์**: ข้อผิดพลาด เพราะ `john.doe@example.com` มีอยู่แล้ว

#### 4. สร้าง Function-Based Index
สมมติ Query นี้ใช้บ่อย:
```sql
SELECT first_name, email
FROM employees
WHERE UPPER(last_name) = 'SMITH';
```
สร้าง Function-Based Index:
```sql
CREATE INDEX idx_emp_upper_last_name ON employees (UPPER(last_name));
```
- **คำอธิบาย**:
    - สร้าง INDEX บนผลลัพธ์ของ `UPPER(last_name)`
    - ช่วยให้ Query ที่ใช้ `UPPER(last_name)` ใน `WHERE` เร็วกว่าเดิม
- **หมายเหตุ**: ต้องใช้ฟังก์ชันที่ตรงกันใน Query (เช่น `UPPER(last_name)`) เพื่อให้ INDEX ทำงาน

#### 5. สร้าง Bitmap Index
สมมติคอลัมน์ `department_id` มีความหลากหลายต่ำและตารางมีการอัปเดตน้อย:
```sql
CREATE BITMAP INDEX idx_emp_dept_bitmap ON employees (department_id);
```
- **คำอธิบาย**:
    - สร้าง Bitmap Index สำหรับ `department_id` ซึ่งมีค่าไม่กี่ค่า (10, 20, 30)
    - เหมาะสำหรับการค้นหาใน Data Warehouse หรือตารางที่อ่านอย่างเดียว
- **ข้อควรระวัง**:
    - Bitmap Index ไม่เหมาะกับตารางที่มีการอัปเดตบ่อย เพราะการล็อก (Locking) อาจทำให้ประสิทธิภาพลดลง

### การจัดการ INDEX
- **ลบ INDEX**:
  ```sql
  DROP INDEX idx_emp_dept_id;
  ```
- **ดู INDEX ทั้งหมดในตาราง**:
  ```sql
  SELECT index_name, column_name
  FROM user_ind_columns
  WHERE table_name = 'EMPLOYEES';
  ```
- **ตรวจสอบการใช้งาน INDEX**:
    - ใช้ `EXPLAIN PLAN` หรือ `AUTOTRACE` เพื่อดูว่า Query ใช้ INDEX หรือไม่
    - ตัวอย่าง:
      ```sql
      SET AUTOTRACE ON;
      SELECT first_name, salary
      FROM employees
      WHERE department_id = 10;
      ```
- **ปรับปรุง INDEX (Rebuild)**:
    - หาก INDEX เสียหายหรือประสิทธิภาพลดลง:
      ```sql
      ALTER INDEX idx_emp_dept_id REBUILD;
      ```
- **ทำให้ INDEX ใช้งานไม่ได้ (Disable)**:
  ```sql
  ALTER INDEX idx_emp_dept_id UNUSABLE;
  ```
    - ใช้เมื่อต้องการหยุดการใช้งาน INDEX ชั่วคราว

### ข้อควรระวัง
- **ผลกระทบต่อประสิทธิภาพ**:
    - INDEX เพิ่มความเร็วในการค้นหา แต่**ช้าลงเมื่อแทรก, อัปเดต, หรือลบ**ข้อมูล เพราะต้องปรับปรุง INDEX
    - การมี INDEX มากเกินไปอาจใช้พื้นที่จัดเก็บมากและลดประสิทธิภาพการเขียนข้อมูล
- **การเลือกคอลัมน์**:
    - สร้าง INDEX บนคอลัมน์ที่ใช้บ่อยใน `WHERE`, `JOIN`, `ORDER BY`, หรือ `GROUP BY`
    - หลีกเลี่ยงการสร้าง INDEX บนคอลัมน์ที่มีความหลากหลายต่ำ (เช่น `gender`) เว้นแต่ใช้ Bitmap Index
- **ขนาดตาราง**:
    - ในตารางขนาดเล็ก (< 1,000 แถว) INDEX อาจไม่ช่วย เพราะ Full Table Scan เร็วกว่า
    - ในตารางขนาดใหญ่ INDEX มีประโยชน์มาก โดยเฉพาะเมื่อเลือกแถวเพียงเล็กน้อย
- **การใช้ INDEX โดย Optimizer**:
    - Oracle Optimizer ตัดสินใจว่าจะใช้ INDEX หรือไม่ตาม**สถิติของตาราง** (Table Statistics)
    - ต้องอัปเดตสถิติตารางเป็นประจำ:
      ```sql
      EXEC DBMS_STATS.GATHER_TABLE_STATS('schema_name', 'employees');
      ```
    - Query บางรูปแบบอาจทำให้ INDEX ไม่ถูกใช้ เช่น:
        - ใช้ฟังก์ชันบนคอลัมน์ที่มี INDEX (เช่น `WHERE UPPER(last_name) = 'SMITH'` ถ้าไม่มี Function-Based Index)
        - ใช้ `!=` หรือ `NOT IN` ในบางกรณี
        - Query ที่เลือกแถวส่วนใหญ่ของตาราง (Optimizer อาจเลือก Full Table Scan)
- **ระบบฐานข้อมูล**:
    - Oracle, PostgreSQL, SQL Server, MySQL รองรับ INDEX คล้ายกัน
    - **MySQL**: รองรับ B-Tree และ Hash Index แต่ Bitmap Index ไม่มี
    - **PostgreSQL**: รองรับ INDEX หลากหลาย เช่น GiST, GIN สำหรับข้อมูลพิเศษ (เช่น JSON)
    - **SQL Server**: มี Clustered Index ซึ่ง Oracle ใช้ Index-Organized Table แทน
- **พื้นที่จัดเก็บ**:
    - INDEX ใช้พื้นที่เพิ่มเติมในฐานข้อมูล ควรพิจารณาขนาดของ INDEX เมื่อสร้าง
    - ใช้ `TABLESPACE` เพื่อจัดการพื้นที่จัดเก็บ INDEX

### การเปรียบเทียบ INDEX กับ Full Table Scan
| คุณสมบัติ              | INDEX                              | Full Table Scan                    |
|------------------------|------------------------------------|------------------------------------|
| **ความเร็ว**           | เร็วสำหรับการค้นหาแถวจำนวนน้อย     | เร็วสำหรับตารางเล็กหรือเลือกแถวมาก |
| **การใช้งาน**          | WHERE, JOIN, ORDER BY, GROUP BY    | Query ที่เลือกแถวส่วนใหญ่          |
| **ผลกระทบการเขียน**    | ช้าลงเมื่อแทรก/อัปเดต/ลบ          | ไม่มีผลกระทบ                      |
| **พื้นที่จัดเก็บ**      | ใช้พื้นที่เพิ่ม                     | ไม่ใช้พื้นที่เพิ่ม                  |

### การใช้งานใน PL/SQL
สามารถใช้ INDEX เพื่อเพิ่มประสิทธิภาพ Query ใน PL/SQL เช่น:
```sql
CREATE INDEX idx_emp_salary ON employees (salary);

DECLARE
   CURSOR high_salary_employees IS
      SELECT first_name, salary
      FROM employees
      WHERE salary > 55000
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
- **อธิบาย**:
    - INDEX `idx_emp_salary` ช่วยให้การกรอง (`salary > 55000`) และการเรียงลำดับ (`ORDER BY salary DESC`) เร็วขึ้น

### ตัวอย่างสถานการณ์จริง
สมมติต้องการเพิ่มความเร็วให้ Query ที่ค้นหาพนักงานตาม `department_id` และ `email` และมีการ JOIN กับ `departments`:

1. **Query ที่ใช้บ่อย**:
   ```sql
   SELECT e.first_name, e.salary, d.department_name
   FROM employees e
   JOIN departments d
   ON e.department_id = d.department_id
   WHERE e.department_id = 20
   AND e.email = 'jane.smith@example.com';
   ```

2. **สร้าง INDEX**:
   ```sql
   CREATE INDEX idx_emp_dept_email ON employees (department_id, email);
   ```
    - INDEX นี้ช่วยให้การกรองด้วย `department_id` และ `email` เร็วขึ้น
    - `department_id` มาก่อนใน INDEX เพราะมักใช้ใน `WHERE` และ `JOIN`

3. **ตรวจสอบการใช้ INDEX**:
   ```sql
   EXPLAIN PLAN FOR
   SELECT e.first_name, e.salary, d.department_name
   FROM employees e
   JOIN departments d
   ON e.department_id = d.department_id
   WHERE e.department_id = 20
   AND e.email = 'jane.smith@example.com';
   SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
   ```
    - **ผลลัพธ์** (ตัวอย่าง):
      ```
      --------------------------------------------------------------------------------
      | Id  | Operation                   | Name              | Rows  | Bytes | Cost  |
      --------------------------------------------------------------------------------
      |   0 | SELECT STATEMENT           |                   |     1 |    80 |     2 |
      |   1 |  NESTED LOOPS             |                   |     1 |    80 |     2 |
      |   2 |   TABLE ACCESS BY INDEX ROWID| EMPLOYEES         |     1 |    60 |     1 |
      |*  3 |    INDEX RANGE SCAN        | IDX_EMP_DEPT_EMAIL|     1 |       |     1 |
      |   4 |   TABLE ACCESS BY INDEX ROWID| DEPARTMENTS       |     1 |    20 |     1 |
      |*  5 |    INDEX UNIQUE SCAN       | SYS_C007001       |     1 |       |     0 |
      --------------------------------------------------------------------------------
      ```
    - **อธิบาย**: Oracle ใช้ `INDEX RANGE SCAN` บน `idx_emp_dept_email` ทำให้ค้นหาเร็ว

4. **เพิ่ม Function-Based Index**:
   สมมติ Query อื่นใช้การค้นหาด้วย `UPPER(email)`:
   ```sql
   SELECT first_name, salary
   FROM employees
   WHERE UPPER(email) = 'JANE.SMITH@EXAMPLE.COM';
   ```
   สร้าง INDEX:
   ```sql
   CREATE INDEX idx_emp_upper_email ON employees (UPPER(email));
   ```

### การเลือก INDEX ให้เหมาะสม
1. **วิเคราะห์ Query**:
    - ดู Query ที่ใช้บ่อยในแอปพลิเคชันหรือรายงาน
    - ใช้เครื่องมือเช่น Oracle Enterprise Manager หรือ `V$SQL` เพื่อหา Query ที่ช้า
2. **เลือกคอลัมน์**:
    - คอลัมน์ที่ใช้ใน `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY`
    - คอลัมน์ที่มีความหลากหลายสูง (เช่น `employee_id`, `email`)
3. **ทดสอบประสิทธิภาพ**:
    - ใช้ `EXPLAIN PLAN` หรือ `AUTOTRACE` เพื่อตรวจสอบว่า INDEX ช่วยลด Cost หรือไม่
    - เปรียบเทียบเวลาการทำงานก่อนและหลังสร้าง INDEX
4. **พิจารณาค่าใช้จ่าย**:
    - INDEX ใช้พื้นที่จัดเก็บและเพิ่มเวลาในการเขียนข้อมูล
    - จำกัดจำนวน INDEX ในตารางที่มีการอัปเดตบ่อย
---

## `SEQUENCE` เพื่อสร้างเลขลำดับอัตโนมัติ
ใน SQL **SEQUENCE** คือวัตถุในฐานข้อมูลที่ใช้สำหรับ**สร้างเลขลำดับอัตโนมัติ** (Sequential Numbers) ซึ่งมักใช้ในการกำหนดค่าให้กับคอลัมน์ที่ต้องการเลขที่ไม่ซ้ำกัน เช่น Primary Key หรือรหัสเฉพาะ (Unique Identifier) ใน Oracle SQL, SEQUENCE มีประโยชน์อย่างมากเมื่อต้องการสร้างลำดับตัวเลขที่เพิ่มขึ้นหรือลดลงโดยอัตโนมัติ โดยไม่ต้องเขียนโค้ดเพิ่มเติม ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งาน SEQUENCE ในบริบทของ Oracle SQL (และเปรียบเทียบกับระบบฐานข้อมูลอื่น)

### คำอธิบายของ SEQUENCE
- **SEQUENCE** คือวัตถุที่สร้างและจัดการโดยฐานข้อมูลเพื่อสร้างลำดับตัวเลขตามกฎที่กำหนด
- **ลักษณะเด่น**:
    - สร้างเลขลำดับที่ไม่ซ้ำกัน (Unique) ได้อย่างรวดเร็ว
    - สามารถกำหนดจุดเริ่มต้น, ขนาดการเพิ่ม/ลด, ค่าสูงสุด/ต่ำสุด, และตัวเลือกอื่น ๆ
    - เหมาะสำหรับการกำหนดค่าให้ Primary Key หรือคอลัมน์ที่ต้องการรหัสเฉพาะ
    - รองรับการใช้งานในหลายตารางหรือหลายเซสชันพร้อมกัน
    - ไม่ขึ้นอยู่กับตาราง (Standalone Object) จึงสามารถใช้ได้ทั่วฐานข้อมูล
- **เมื่อใช้ SEQUENCE**:
    - สร้างรหัสพนักงาน, รหัสคำสั่งซื้อ, หรือรหัสเอกสาร
    - กำหนด Primary Key โดยอัตโนมัติ
    - ต้องการเลขลำดับที่เพิ่มขึ้นหรือลดลงอย่างต่อเนื่อง
- **ข้อจำกัด**:
    - SEQUENCE ไม่รับประกันว่าเลขลำดับจะต่อเนื่อง 100% (อาจมีช่องว่างเนื่องจากการ Rollback หรือ Cache)
    - ไม่สามารถย้อนกลับค่า SEQUENCE ที่ใช้ไปแล้วได้โดยตรง

### ไวยากรณ์พื้นฐาน
```sql
-- สร้าง SEQUENCE
CREATE SEQUENCE sequence_name
    [START WITH start_value]
    [INCREMENT BY increment_value]
    [MAXVALUE max_value | NOMAXVALUE]
    [MINVALUE min_value | NOMINVALUE]
    [CYCLE | NOCYCLE]
    [CACHE cache_size | NOCACHE]
    [ORDER | NOORDER];

-- เรียกใช้ SEQUENCE
SELECT sequence_name.NEXTVAL FROM dual;  -- ได้ค่าใหม่
SELECT sequence_name.CURRVAL FROM dual;  -- ได้ค่าปัจจุบัน

-- ลบ SEQUENCE
DROP SEQUENCE sequence_name;

-- ดู SEQUENCE ที่มีอยู่
SELECT sequence_name, min_value, max_value, increment_by, last_number
FROM user_sequences;
```

- **sequence_name**: ชื่อของ SEQUENCE
- **START WITH**: ค่าเริ่มต้นของลำดับ (ค่าเริ่มต้นคือ 1)
- **INCREMENT BY**: ขนาดการเพิ่ม/ลดของลำดับ (บวกสำหรับเพิ่ม, ลบสำหรับลด, ค่าเริ่มต้นคือ 1)
- **MAXVALUE / NOMAXVALUE**: ค่าสูงสุดของลำดับ (ค่าเริ่มต้นคือ 10^27 สำหรับเลขบวก)
- **MINVALUE / NOMINVALUE**: ค่าต่ำสุดของลำดับ (ค่าเริ่มต้นคือ 1 สำหรับเลขบวก)
- **CYCLE / NOCYCLE**: เริ่มต้นใหม่เมื่อถึงค่าสูงสุด/ต่ำสุด (CYCLE) หรือหยุด (NOCYCLE, ค่าเริ่มต้น)
- **CACHE / NOCACHE**: จำนวนค่า SEQUENCE ที่เก็บในหน่วยความจำเพื่อเพิ่มประสิทธิภาพ (ค่าเริ่มต้นคือ CACHE 20)
- **ORDER / NOORDER**: รับประกันลำดับการสร้างเลขในเซสชันหลายตัว (ORDER) หรือไม่ (NOORDER, ค่าเริ่มต้น)
- **NEXTVAL**: ดึงค่า SEQUENCE ถัดไปและเพิ่มลำดับ
- **CURRVAL**: ดึงค่า SEQUENCE ปัจจุบัน (ต้องเรียก `NEXTVAL` ในเซสชันก่อน)

### ตัวอย่างการใช้งาน
สมมติมีตาราง:
1. **employees**:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    salary NUMBER(10,2),
    department_id NUMBER(5)
);
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

#### 1. สร้าง SEQUENCE พื้นฐาน
```sql
CREATE SEQUENCE emp_seq
    START WITH 1
    INCREMENT BY 1
    MINVALUE 1
    MAXVALUE 999999
    NOCYCLE
    CACHE 20;
```
- **คำอธิบาย**:
    - สร้าง SEQUENCE ชื่อ `emp_seq`
    - เริ่มที่ 1, เพิ่มครั้งละ 1, สูงสุด 999999, ไม่วนซ้ำ, เก็บ 20 ค่าในแคช
- **ทดสอบการใช้**:
  ```sql
  SELECT emp_seq.NEXTVAL FROM dual;
  ```
    - **ผลลัพธ์**: `1`
  ```sql
  SELECT emp_seq.NEXTVAL FROM dual;
  ```
    - **ผลลัพธ์**: `2`
  ```sql
  SELECT emp_seq.CURRVAL FROM dual;
  ```
    - **ผลลัพธ์**: `2`

#### 2. ใช้ SEQUENCE ในการแทรกข้อมูล
```sql
INSERT INTO employees (employee_id, first_name, salary, department_id)
VALUES (emp_seq.NEXTVAL, 'John', 50000, 10);

INSERT INTO employees (employee_id, first_name, salary, department_id)
VALUES (emp_seq.NEXTVAL, 'Jane', 60000, 20);

INSERT INTO employees (employee_id, first_name, salary, department_id)
VALUES (emp_seq.NEXTVAL, 'Alice', 55000, 10);
```
- **เรียกดูข้อมูล**:
  ```sql
  SELECT * FROM employees;
  ```
- **ผลลัพธ์**:
  ```
  employee_id | first_name | salary | department_id
  ------------+------------+--------+----------------
  1          | John       | 50000  | 10
  2          | Jane       | 60000  | 20
  3          | Alice      | 55000  | 10
  ```
- **อธิบาย**:
    - `emp_seq.NEXTVAL` สร้าง `employee_id` อัตโนมัติ (1, 2, 3)

#### 3. ใช้ SEQUENCE ร่วมกับ TRIGGER
เพื่อให้ `employee_id` ถูกกำหนดโดยอัตโนมัติเมื่อแทรกข้อมูล:
```sql
CREATE OR REPLACE TRIGGER emp_trigger
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    IF :NEW.employee_id IS NULL THEN
        :NEW.employee_id := emp_seq.NEXTVAL;
    END IF;
END;
/
```
- **คำอธิบาย**:
    - สร้าง TRIGGER ชื่อ `emp_trigger` ที่ทำงานก่อนแทรกข้อมูล
    - ถ้า `employee_id` เป็น NULL จะกำหนดค่าโดยใช้ `emp_seq.NEXTVAL`
- **ทดสอบ**:
  ```sql
  INSERT INTO employees (first_name, salary, department_id)
  VALUES ('Bob', 65000, 20);

  INSERT INTO employees (first_name, salary, department_id)
  VALUES ('Carol', 52000, 30);
  ```
- **เรียกดูข้อมูล**:
  ```sql
  SELECT * FROM employees;
  ```
- **ผลลัพธ์**:
  ```
  employee_id | first_name | salary | department_id
  ------------+------------+--------+----------------
  1          | John       | 50000  | 10
  2          | Jane       | 60000  | 20
  3          | Alice      | 55000  | 10
  4          | Bob        | 65000  | 20
  5          | Carol      | 52000  | 30
  ```

#### 4. สร้าง SEQUENCE แบบ CYCLE
สมมติต้องการ SEQUENCE ที่วนซ้ำเมื่อถึงค่าสูงสุด:
```sql
CREATE SEQUENCE dept_seq
    START WITH 100
    INCREMENT BY 10
    MINVALUE 100
    MAXVALUE 130
    CYCLE
    CACHE 5;
```
- **คำอธิบาย**:
    - เริ่มที่ 100, เพิ่มครั้งละ 10, สูงสุด 130, วนกลับไป 100 เมื่อถึงค่าสูงสุด
- **ทดสอบ**:
  ```sql
  SELECT dept_seq.NEXTVAL FROM dual; -- 100
  SELECT dept_seq.NEXTVAL FROM dual; -- 110
  SELECT dept_seq.NEXTVAL FROM dual; -- 120
  SELECT dept_seq.NEXTVAL FROM dual; -- 130
  SELECT dept_seq.NEXTVAL FROM dual; -- 100 (วนซ้ำ)
  ```

#### 5. การใช้ SEQUENCE ใน PL/SQL
```sql
DECLARE
   v_emp_id NUMBER;
   v_first_name VARCHAR2(50) := 'David';
   v_salary NUMBER := 58000;
   v_dept_id NUMBER := 10;
BEGIN
   -- ดึงค่า SEQUENCE
   SELECT emp_seq.NEXTVAL INTO v_emp_id FROM dual;
   
   -- แทรกข้อมูล
   INSERT INTO employees (employee_id, first_name, salary, department_id)
   VALUES (v_emp_id, v_first_name, v_salary, v_dept_id);
   
   DBMS_OUTPUT.PUT_LINE('Inserted Employee ID: ' || v_emp_id);
   COMMIT;
END;
/
```
- **ผลลัพธ์**:
  ```
  Inserted Employee ID: 6
  ```
- **เรียกดูข้อมูล**:
  ```sql
  SELECT * FROM employees WHERE employee_id = 6;
  ```
    - **ผลลัพธ์**:
      ```
      employee_id | first_name | salary | department_id
      ------------+------------+--------+----------------
      6          | David      | 58000  | 10
      ```

### ข้อควรระวัง
- **ช่องว่างในลำดับ (Gaps)**:
    - SEQUENCE อาจมีช่องว่างในเลขลำดับ เนื่องจาก:
        - การ Rollback ธุรกรรม (Transaction) หลังจากเรียก `NEXTVAL`
        - การใช้ `CACHE` ซึ่งจองเลขลำดับไว้ล่วงหน้า
        - การข้ามค่าในเซสชันที่ล้มเหลว
    - ถ้าต้องการเลขต่อเนื่อง 100% อาจต้องใช้กลไกอื่น (เช่น เก็บค่าในตารางและอัปเดตด้วย Trigger)
- **CACHE vs NOCACHE**:
    - `CACHE` (ค่าเริ่มต้น 20) เก็บเลขลำดับในหน่วยความจำเพื่อเพิ่มประสิทธิภาพ แต่ถ้าเซิร์ฟเวอร์ล่ม อาจสูญเสียเลขในแคช
    - `NOCACHE` ป้องกันช่องว่างจากแคช แต่ช้ากว่า
    - ตัวอย่าง:
      ```sql
      CREATE SEQUENCE no_cache_seq START WITH 1 INCREMENT BY 1 NOCACHE;
      ```
- **CURRVAL**:
    - ต้องเรียก `NEXTVAL` ในเซสชันก่อนจึงจะใช้ `CURRVAL` ได้ มิฉะนั้นจะเกิดข้อผิดพลาด (เช่น `ORA-08002: sequence CURRVAL is not yet defined in this session`)
- **CYCLE**:
    - การใช้ `CYCLE` อาจทำให้เกิดค่าซ้ำในลำดับถ้าไม่ระวัง โดยเฉพาะเมื่อใช้กับ Primary Key
    - ควรใช้ `NOCYCLE` สำหรับ Primary Key เพื่อป้องกันข้อผิดพลาด `UNIQUE CONSTRAINT`
- **การจัดการ SEQUENCE**:
    - ไม่สามารถย้อนกลับค่า `NEXTVAL` ที่ใช้ไปแล้วได้
    - หากต้องการรีเซ็ต SEQUENCE ต้องลบและสร้างใหม่ หรือแก้ไขด้วย `ALTER SEQUENCE`:
      ```sql
      ALTER SEQUENCE emp_seq
          INCREMENT BY -100
          MINVALUE 1;
      SELECT emp_seq.NEXTVAL FROM dual; -- ลดค่าลง
      ALTER SEQUENCE emp_seq
          INCREMENT BY 1;
      ```
- **ประสิทธิภาพ**:
    - SEQUENCE มีประสิทธิภาพสูง เพราะออกแบบมาเพื่อสร้างเลขลำดับในสภาพแวดล้อมที่มีการใช้งานพร้อมกัน (Concurrent)
    - การใช้ `CACHE` ช่วยลดการล็อก (Locking) ในเซสชันหลายตัว
- **ระบบฐานข้อมูล**:
    - **Oracle**: รองรับ SEQUENCE อย่างสมบูรณ์ด้วย `NEXTVAL`, `CURRVAL`
    - **PostgreSQL**: รองรับ SEQUENCE คล้าย Oracle แต่ใช้ `nextval('sequence_name')` และ `currval('sequence_name')`
    - **MySQL**: ไม่มี SEQUENCE โดยตรง แต่ใช้ `AUTO_INCREMENT` ในตาราง หรือสร้างตารางจำลอง SEQUENCE
        - ตัวอย่างใน MySQL:
          ```sql
          CREATE TABLE my_sequence (id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY);
          INSERT INTO my_sequence VALUES (0);
          ```
    - **SQL Server**: รองรับ SEQUENCE ตั้งแต่เวอร์ชัน 2012 คล้าย Oracle
- **ความปลอดภัย**:
    - ต้องให้สิทธิ์การใช้งาน SEQUENCE:
      ```sql
      GRANT SELECT ON emp_seq TO user_name;
      ```

### การเปรียบเทียบ SEQUENCE กับ AUTO_INCREMENT
| คุณสมบัติ              | SEQUENCE (Oracle)                  | AUTO_INCREMENT (MySQL)             |
|------------------------|------------------------------------|------------------------------------|
| **การใช้งาน**          | Standalone Object ใช้ได้หลายตาราง  | ผูกกับคอลัมน์ในตารางเดียว         |
| **ความยืดหยุ่น**      | กำหนด START, INCREMENT, CYCLE ได้  | จำกัดที่เพิ่มครั้งละ 1, ไม่มี CYCLE |
| **การควบคุม**          | ใช้ NEXTVAL, CURRVAL ได้ใน Query   | ไม่มีวิธีควบคุมค่าใน Query โดยตรง |
| **ช่องว่าง**          | อาจมีช่องว่างจาก CACHE/Rollback   | อาจมีช่องว่างจาก Rollback        |
| **การใช้งานทั่วไป**    | Primary Key, รหัสเฉพาะทั่วไป       | Primary Key ในตารางเดียว           |

### การใช้งานใน PL/SQL
สามารถใช้ SEQUENCE ใน PL/SQL เพื่อสร้างรหัสอัตโนมัติ เช่น:
```sql
DECLARE
   v_count NUMBER := 5;
   v_first_name VARCHAR2(50);
   v_salary NUMBER;
   v_dept_id NUMBER;
BEGIN
   FOR i IN 1..v_count LOOP
      -- สร้างข้อมูลจำลอง
      v_first_name := 'Employee_' || emp_seq.NEXTVAL;
      v_salary := 50000 + (i * 1000);
      v_dept_id := 10 + (i MOD 3) * 10;
      
      INSERT INTO employees (employee_id, first_name, salary, department_id)
      VALUES (emp_seq.CURRVAL, v_first_name, v_salary, v_dept_id);
      
      DBMS_OUTPUT.PUT_LINE('Inserted Employee ID: ' || emp_seq.CURRVAL);
   END LOOP;
   COMMIT;
END;
/
```
- **ผลลัพธ์** (ตัวอย่าง):
  ```
  Inserted Employee ID: 6
  Inserted Employee ID: 7
  Inserted Employee ID: 8
  Inserted Employee ID: 9
  Inserted Employee ID: 10
  ```
- **เรียกดูข้อมูล**:
  ```sql
  SELECT * FROM employees;
  ```
    - **ผลลัพธ์** (รวมข้อมูลก่อนหน้า):
      ```
      employee_id | first_name     | salary | department_id
      ------------+----------------+--------+----------------
      1          | John           | 50000  | 10
      2          | Jane           | 60000  | 20
      3          | Alice          | 55000  | 10
      4          | Bob            | 65000  | 20
      5          | Carol          | 52000  | 30
      6          | Employee_6     | 51000  | 10
      7          | Employee_7     | 52000  | 20
      8          | Employee_8     | 53000  | 30
      9          | Employee_9     | 54000  | 10
      10         | Employee_10    | 55000  | 20
      ```

### ตัวอย่างสถานการณ์จริง
สมมติต้องการสร้างระบบจัดการคำสั่งซื้อ โดยให้แต่ละคำสั่งซื้อมี `order_id` ที่สร้างโดยอัตโนมัติ:

1. **สร้างตารางและ SEQUENCE**:
```sql
CREATE TABLE orders (
    order_id NUMBER(10) PRIMARY KEY,
    employee_id NUMBER(10),
    order_amount NUMBER(10,2),
    order_date DATE
);

CREATE SEQUENCE order_seq
    START WITH 1000
    INCREMENT BY 1
    MINVALUE 1000
    MAXVALUE 9999999
    NOCYCLE
    CACHE 50;
```

2. **สร้าง TRIGGER สำหรับการแทรก**:
```sql
CREATE OR REPLACE TRIGGER order_trigger
BEFORE INSERT ON orders
FOR EACH ROW
BEGIN
    IF :NEW.order_id IS NULL THEN
        :NEW.order_id := order_seq.NEXTVAL;
    END IF;
END;
/
```

3. **แทรกข้อมูล**:
```sql
INSERT INTO orders (employee_id, order_amount, order_date)
VALUES (1, 1500.50, SYSDATE);

INSERT INTO orders (employee_id, order_amount, order_date)
VALUES (2, 2500.75, SYSDATE);
```

4. **เรียกดูข้อมูล**:
```sql
SELECT * FROM orders;
```
- **ผลลัพธ์**:
  ```
  order_id | employee_id | order_amount | order_date
  ---------+------------+--------------+------------
  1000     | 1          | 1500.50      | 2025-04-23
  1001     | 2          | 2500.75      | 2025-04-23
  ```

5. **ใช้ SEQUENCE ใน Query ร่วมกับ JOIN**:
```sql
WITH new_order AS (
    SELECT order_seq.NEXTVAL AS new_order_id, 3 AS employee_id, 3000 AS order_amount, SYSDATE AS order_date
    FROM dual
)
SELECT no.new_order_id, e.first_name, no.order_amount, d.department_name
FROM new_order no
JOIN employees e ON no.employee_id = e.employee_id
JOIN departments d ON e.department_id = d.department_id;
```
- **ผลลัพธ์**:
  ```
  new_order_id | first_name | order_amount | department_name
  -------------+------------+--------------+----------------
  1002         | Alice      | 3000         | Sales
  ```
- **อธิบาย**:
    - ใช้ `order_seq.NEXTVAL` เพื่อจำลอง `order_id` ใหม่
    - JOIN กับ `employees` และ `departments` เพื่อดึงข้อมูลพนักงานและแผนก
---
