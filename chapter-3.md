# บทที่ 3: การเพิ่ม แก้ไข และลบข้อมูล (INSERT, UPDATE, DELETE)
## `INSERT INTO` สำหรับเพิ่มข้อมูล
คำสั่ง **INSERT INTO** ใน SQL ใช้สำหรับ**เพิ่มข้อมูล**ลงในตาราง เป็นส่วนหนึ่งของ DML (Data Manipulation Language) โดยสามารถเพิ่มข้อมูลหนึ่งแถวหรือหลายแถวในคราวเดียว และสามารถระบุคอลัมน์ที่ต้องการเพิ่มข้อมูลหรือเพิ่มข้อมูลทั้งหมดตามโครงสร้างตาราง

### ไวยากรณ์พื้นฐาน:
1. **เพิ่มข้อมูลโดยระบุคอลัมน์**:
   ```sql
   INSERT INTO table_name (column1, column2, ...)
   VALUES (value1, value2, ...);
   ```

2. **เพิ่มข้อมูลทุกคอลัมน์** (ตามลำดับในโครงสร้างตาราง):
   ```sql
   INSERT INTO table_name
   VALUES (value1, value2, ...);
   ```

3. **เพิ่มข้อมูลหลายแถวพร้อมกัน**:
   ```sql
   INSERT INTO table_name (column1, column2, ...)
   VALUES 
       (value1, value2, ...),
       (value3, value4, ...),
       ...;
   ```

4. **เพิ่มข้อมูลจากตารางอื่น** (ใช้ SELECT):
   ```sql
   INSERT INTO table_name (column1, column2, ...)
   SELECT column1, column2, ...
   FROM another_table
   [WHERE condition];
   ```

### อธิบายส่วนประกอบ:
- **table_name**: ชื่อตารางที่ต้องการเพิ่มข้อมูล
- **column1, column2, ...**: คอลัมน์ที่ต้องการเพิ่มข้อมูล (ถ้าไม่ระบุ ต้องใส่ค่าทุกคอลัมน์ตามลำดับ)
- **value1, value2, ...**: ค่าที่จะเพิ่มลงในคอลัมน์ (ต้องตรงกับชนิดข้อมูลของคอลัมน์)
- **SELECT**: ใช้เมื่อต้องการคัดลอกข้อมูลจากตารางอื่น

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees` ดังนี้:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE,
    salary NUMBER(10,2),
    hire_date DATE
);
```

1. **เพิ่มข้อมูลหนึ่งแถวโดยระบุคอลัมน์**:
   ```sql
   INSERT INTO employees (employee_id, first_name, last_name, email, salary, hire_date)
   VALUES (1, 'John', 'Doe', 'john.doe@example.com', 50000.00, TO_DATE('2023-01-15', 'YYYY-MM-DD'));
   ```
    - เพิ่มพนักงานที่มี `employee_id` เป็น 1 และข้อมูลอื่น ๆ
    - ใช้ `TO_DATE` เพื่อแปลงสตริงเป็นวันที่ใน Oracle

2. **เพิ่มข้อมูลทุกคอลัมน์**:
   ```sql
   INSERT INTO employees
   VALUES (2, 'Jane', 'Smith', 'jane.smith@example.com', 60000.00, TO_DATE('2023-02-10', 'YYYY-MM-DD'));
   ```
    - ต้องระบุค่าทุกคอลัมน์ตามลำดับในโครงสร้างตาราง

3. **เพิ่มข้อมูลหลายแถว**:
   ```sql
   INSERT INTO employees (employee_id, first_name, last_name, email, salary, hire_date)
   VALUES 
       (3, 'Alice', 'Brown', 'alice.brown@example.com', 55000.00, TO_DATE('2023-03-01', 'YYYY-MM-DD')),
       (4, 'Bob', 'Wilson', 'bob.wilson@example.com', 65000.00, TO_DATE('2023-04-01', 'YYYY-MM-DD'));
   ```
    - เพิ่มข้อมูลสองแถวในคำสั่งเดียว

4. **เพิ่มข้อมูลจากตารางอื่น**:
   สมมติมีตาราง `temp_employees` ที่มีโครงสร้างเหมือน `employees`:
   ```sql
   INSERT INTO employees (employee_id, first_name, last_name, email, salary, hire_date)
   SELECT employee_id, first_name, last_name, email, salary, hire_date
   FROM temp_employees
   WHERE salary > 50000;
   ```
    - คัดลอกข้อมูลจาก `temp_employees` ที่มี `salary` มากกว่า 50,000

5. **เพิ่มข้อมูลโดยใช้ค่าเริ่มต้น**:
   หากตารางมีค่าเริ่มต้น (Default Value) สามารถข้ามคอลัมน์นั้นได้:
   ```sql
   INSERT INTO employees (employee_id, first_name, last_name, email)
   VALUES (5, 'Carol', 'Davis', 'carol.davis@example.com');
   ```
    - คอลัมน์ `salary` และ `hire_date` จะใช้ค่าเริ่มต้น (ถ้ามี) หรือเป็น `NULL` (ถ้าไม่มีข้อจำกัด `NOT NULL`)

### ข้อควรระวัง:
- **ชนิดข้อมูล**: ค่าที่เพิ่มต้องตรงกับชนิดข้อมูลของคอลัมน์ เช่น `NUMBER`, `VARCHAR2`, `DATE`
- **ข้อจำกัด**:
    - **NOT NULL**: ต้องระบุค่าสำหรับคอลัมน์ที่มีข้อจำกัด `NOT NULL`
    - **PRIMARY KEY/UNIQUE**: ค่าต้องไม่ซ้ำกับข้อมูลที่มีอยู่
    - **FOREIGN KEY**: ค่าต้องอ้างอิงถึงค่าที่มีอยู่ในตารางที่เกี่ยวข้อง
    - **CHECK**: ค่าต้องผ่านเงื่อนไขที่กำหนด
- **วันที่ใน Oracle**: ใช้ `TO_DATE` เพื่อระบุวันที่ให้ถูกต้อง เช่น `TO_DATE('2023-01-15', 'YYYY-MM-DD')`
- **ขนาดข้อมูล**: เช่น `VARCHAR2(50)` ต้องไม่เกิน 50 ตัวอักษร
- **การยืนยันธุรกรรม**: ในบางระบบ (เช่น Oracle) ต้องใช้ `COMMIT` เพื่อบันทึกข้อมูลถาวร:
  ```sql
  INSERT INTO employees (employee_id, first_name, last_name)
  VALUES (6, 'David', 'Lee');
  COMMIT;
  ```
- **ข้อผิดพลาด**: หากละเมิดข้อจำกัด (เช่น ใส่ `email` ซ้ำ) จะเกิดข้อผิดพลาด เช่น `ORA-00001: unique constraint violated`

### การใช้งานใน PL/SQL:
สามารถใช้ `INSERT INTO` ภายในบล็อก PL/SQL เพื่อเพิ่มข้อมูลแบบมีเงื่อนไขหรือวนลูป เช่น:
```sql
BEGIN
   INSERT INTO employees (employee_id, first_name, last_name, email, salary, hire_date)
   VALUES (7, 'Emma', 'Clark', 'emma.clark@example.com', 70000.00, SYSDATE);
   DBMS_OUTPUT.PUT_LINE('Employee added successfully.');
EXCEPTION
   WHEN DUP_VAL_ON_INDEX THEN
      DBMS_OUTPUT.PUT_LINE('Error: Duplicate email or employee ID.');
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/
```
- ใช้ `SYSDATE` สำหรับวันที่ปัจจุบัน
- จัดการข้อผิดพลาด เช่น การละเมิด `UNIQUE` หรือ `PRIMARY KEY`

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการเพิ่มพนักงานใหม่ในตาราง `employees` และตรวจสอบโครงสร้างก่อน:
```sql
DESC employees;
-- ตรวจสอบโครงสร้าง
INSERT INTO employees (employee_id, first_name, last_name, email, salary, hire_date)
VALUES (8, 'Frank', 'Taylor', 'frank.taylor@example.com', 52000.00, TO_DATE('2023-05-10', 'YYYY-MM-DD'));
COMMIT;
-- บันทึกข้อมูล
```
---

## `UPDATE` เพื่อแก้ไขข้อมูล
คำสั่ง **UPDATE** ใน SQL ใช้สำหรับ**แก้ไขข้อมูล**ในตารางที่มีอยู่ เป็นส่วนหนึ่งของ DML (Data Manipulation Language) โดยสามารถอัปเดตข้อมูลในหนึ่งหรือหลายคอลัมน์ของแถวที่ตรงตามเงื่อนไขที่กำหนด

### ไวยากรณ์พื้นฐาน:
```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
[WHERE condition];
```

### อธิบายส่วนประกอบ:
- **table_name**: ชื่อตารางที่ต้องการแก้ไขข้อมูล
- **SET**: ระบุคอลัมน์และค่าที่ต้องการอัปเดต
- **WHERE**: เงื่อนไขเพื่อเลือกแถวที่ต้องการอัปเดต (ถ้าไม่ระบุ จะอัปเดตทุกแถวในตาราง)
- **value**: ค่าที่จะกำหนดให้คอลัมน์ (สามารถเป็นค่าคงที่, นิพจน์, หรือผลลัพธ์จาก subquery)

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees` ดังนี้:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE,
    salary NUMBER(10,2),
    hire_date DATE
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 'Doe', 'john.doe@example.com', 50000.00, TO_DATE('2023-01-15', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (2, 'Jane', 'Smith', 'jane.smith@example.com', 60000.00, TO_DATE('2023-02-10', 'YYYY-MM-DD'));
COMMIT;
```

1. **อัปเดตข้อมูลหนึ่งคอลัมน์**:
   ```sql
   UPDATE employees
   SET salary = 55000.00
   WHERE employee_id = 1;
   ```
    - เพิ่มเงินเดือนของพนักงานที่มี `employee_id` เป็น 1 เป็น 55,000

2. **อัปเดตหลายคอลัมน์**:
   ```sql
   UPDATE employees
   SET email = 'john.new@example.com', last_name = 'Davis'
   WHERE employee_id = 1;
   ```
    - เปลี่ยน `email` และ `last_name` ของพนักงาน `employee_id` = 1

3. **อัปเดตทุกแถว** (ถ้าไม่มี WHERE):
   ```sql
   UPDATE employees
   SET salary = salary * 1.10;
   ```
    - เพิ่มเงินเดือนทุกคนในตาราง 10% (ระวังการใช้โดยไม่มี `WHERE` เพราะจะกระทบทุกแถว)

4. **ใช้เงื่อนไขที่ซับซ้อน**:
   ```sql
   UPDATE employees
   SET salary = salary + 5000
   WHERE hire_date < TO_DATE('2023-02-01', 'YYYY-MM-DD');
   ```
    - เพิ่มเงินเดือน 5,000 สำหรับพนักงานที่เริ่มงานก่อนวันที่ 1 ก.พ. 2023

5. **ใช้ Subquery ใน UPDATE**:
   ```sql
   UPDATE employees
   SET salary = (SELECT AVG(salary) FROM employees)
   WHERE employee_id = 2;
   ```
    - อัปเดตเงินเดือนของพนักงาน `employee_id` = 2 ให้เท่ากับค่าเฉลี่ยเงินเดือนในตาราง

### ข้อควรระวัง:
- **WHERE Clause**: ถ้าไม่ระบุ `WHERE` คำสั่งจะอัปเดตทุกแถวในตาราง ซึ่งอาจไม่ใช่สิ่งที่ต้องการ
- **ข้อจำกัด**:
    - **NOT NULL**: ค่าที่อัปเดตต้องไม่เป็น `NULL` สำหรับคอลัมน์ที่มีข้อจำกัด `NOT NULL`
    - **UNIQUE/PRIMARY KEY**: ค่าที่อัปเดตต้องไม่ซ้ำกับข้อมูลที่มีอยู่ในคอลัมน์ที่มีข้อจำกัด `UNIQUE` หรือ `PRIMARY KEY`
    - **FOREIGN KEY**: ค่าต้องอ้างอิงถึงค่าที่มีอยู่ในตารางที่เกี่ยวข้อง
- **การยืนยันธุรกรรม**: ในระบบฐานข้อมูล เช่น Oracle ต้องใช้ `COMMIT` เพื่อบันทึกการเปลี่ยนแปลงถาวร:
  ```sql
  UPDATE employees
  SET salary = 58000.00
  WHERE employee_id = 1;
  COMMIT;
  ```
  หรือใช้ `ROLLBACK` หากต้องการยกเลิก:
  ```sql
  ROLLBACK;
  ```
- **ประสิทธิภาพ**: การอัปเดตข้อมูลจำนวนมากอาจช้าและใช้ทรัพยากรเยอะ ควรทดสอบในสภาพแวดล้อมที่ไม่ใช่ Production ก่อน
- **ข้อผิดพลาด**: เช่น การละเมิด `UNIQUE` (เช่น อัปเดต `email` ซ้ำ) จะทำให้เกิดข้อผิดพลาด เช่น `ORA-00001: unique constraint violated`

### การใช้งานใน PL/SQL:
สามารถใช้ `UPDATE` ภายในบล็อก PL/SQL เพื่อควบคุมการอัปเดตหรือจัดการข้อผิดพลาด เช่น:
```sql
BEGIN
   UPDATE employees
   SET salary = salary + 10000
   WHERE employee_id = 1;
   
   IF SQL%ROWCOUNT = 0 THEN
      DBMS_OUTPUT.PUT_LINE('No rows updated.');
   ELSE
      DBMS_OUTPUT.PUT_LINE(SQL%ROWCOUNT || ' row(s) updated.');
   END IF;
   
   COMMIT;
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
      ROLLBACK;
END;
/
```
- `SQL%ROWCOUNT` ใช้ตรวจสอบจำนวนแถวที่ถูกอัปเดต
- จัดการข้อผิดพลาดและควบคุมธุรกรรมด้วย `COMMIT` หรือ `ROLLBACK`

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการปรับเงินเดือนพนักงานที่มี `salary` ต่ำกว่า 60,000 และเปลี่ยน `email` สำหรับพนักงานบางคน:
```sql
-- ตรวจสอบข้อมูลก่อน
SELECT employee_id, first_name, salary, email FROM employees;

-- อัปเดตเงินเดือน
UPDATE employees
SET salary = salary + 2000
WHERE salary < 60000;

-- อัปเดต email
UPDATE employees
SET email = 'jane.new@example.com'
WHERE employee_id = 2;

COMMIT;
-- บันทึกการเปลี่ยนแปลง
```

### ความแตกต่างกับคำสั่งอื่น:
- **UPDATE** vs **INSERT**: `UPDATE` แก้ไขข้อมูลที่มีอยู่, `INSERT` เพิ่มข้อมูลใหม่
- **UPDATE** vs **DELETE**: `UPDATE` เปลี่ยนค่าในแถว, `DELETE` ลบแถวทั้งหมด
---

## `DELETE FROM` เพื่อลบข้อมูล
คำสั่ง **DELETE FROM** ใน SQL ใช้สำหรับ**ลบข้อมูล** (แถว) ออกจากตาราง เป็นส่วนหนึ่งของ DML (Data Manipulation Language) โดยสามารถลบแถวที่ตรงตามเงื่อนไขที่กำหนดหรือลบทุกแถวในตารางหากไม่ระบุเงื่อนไข คำสั่งนี้จะลบเฉพาะข้อมูล ไม่กระทบโครงสร้างตาราง

### ไวยากรณ์พื้นฐาน:
```sql
DELETE FROM table_name
[WHERE condition];
```

### อธิบายส่วนประกอบ:
- **table_name**: ชื่อตารางที่ต้องการลบข้อมูล
- **WHERE**: เงื่อนไขเพื่อเลือกแถวที่ต้องการลบ (ถ้าไม่ระบุ จะลบทุกแถวในตาราง)
- **condition**: เงื่อนไข เช่น `employee_id = 1` หรือ `salary < 50000`

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees` ดังนี้:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE,
    salary NUMBER(10,2),
    hire_date DATE
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 'Doe', 'john.doe@example.com', 50000.00, TO_DATE('2023-01-15', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (2, 'Jane', 'Smith', 'jane.smith@example.com', 60000.00, TO_DATE('2023-02-10', 'YYYY-MM-DD'));
INSERT INTO employees VALUES (3, 'Alice', 'Brown', 'alice.brown@example.com', 55000.00, TO_DATE('2023-03-01', 'YYYY-MM-DD'));
COMMIT;
```

1. **ลบแถวที่ระบุเงื่อนไข**:
   ```sql
   DELETE FROM employees
   WHERE employee_id = 1;
   ```
    - ลบพนักงานที่มี `employee_id` เป็น 1

2. **ลบแถวหลายแถวตามเงื่อนไข**:
   ```sql
   DELETE FROM employees
   WHERE salary < 60000;
   ```
    - ลบพนักงานที่มีเงินเดือนต่ำกว่า 60,000

3. **ลบทุกแถวในตาราง** (ถ้าไม่มี WHERE):
   ```sql
   DELETE FROM employees;
   ```
    - ลบข้อมูลทั้งหมดในตาราง `employees` แต่โครงสร้างตารางยังคงอยู่

4. **ใช้เงื่อนไขที่ซับซ้อน**:
   ```sql
   DELETE FROM employees
   WHERE hire_date < TO_DATE('2023-02-01', 'YYYY-MM-DD') AND salary < 55000;
   ```
    - ลบพนักงานที่เริ่มงานก่อน 1 ก.พ. 2023 และมีเงินเดือนต่ำกว่า 55,000

5. **ลบโดยใช้ Subquery**:
   ```sql
   DELETE FROM employees
   WHERE department_id IN (SELECT department_id FROM departments WHERE department_name = 'Sales');
   ```
    - ลบพนักงานที่อยู่ในแผนก 'Sales' โดยอ้างอิงจากตาราง `departments`

### ข้อควรระวัง:
- **WHERE Clause**: ถ้าไม่ระบุ `WHERE` คำสั่งจะลบทุกแถวในตาราง ซึ่งอาจไม่ใช่สิ่งที่ต้องการ
- **ข้อจำกัด Foreign Key**: ถ้าตารางมี Foreign Key อ้างอิงจากตารางอื่น อาจไม่สามารถลบได้เว้นแต่:
    - ใช้ `ON DELETE CASCADE` ใน Foreign Key (ลบข้อมูลที่เกี่ยวข้องอัตโนมัติ)
    - ลบข้อมูลในตารางลูก (child table) ก่อน
    - ตัวอย่าง: ถ้า `employees` มี `department_id` อ้างอิง `departments` ต้องลบ `employees` ก่อน หรือใช้ `CASCADE`
- **การยืนยันธุรกรรม**: ในระบบฐานข้อมูล เช่น Oracle ต้องใช้ `COMMIT` เพื่อบันทึกการลบถาวร:
  ```sql
  DELETE FROM employees WHERE employee_id = 2;
  COMMIT;
  ```
  หรือใช้ `ROLLBACK` เพื่อยกเลิก:
  ```sql
  ROLLBACK;
  ```
- **ประสิทธิภาพ**: การลบข้อมูลจำนวนมากอาจใช้เวลานานและกระทบประสิทธิภาพ ควรทดสอบก่อนในสภาพแวดล้อมที่ไม่ใช่ Production
- **ความแตกต่างกับ TRUNCATE**:
    - `DELETE`: ลบข้อมูลตามเงื่อนไข (DML) สามารถกู้คืนได้ด้วย `ROLLBACK`
    - `TRUNCATE`: ลบข้อมูลทั้งหมดและรีเซ็ตตาราง (DDL) ไม่สามารถกู้คืนด้วย `ROLLBACK`
      ```sql
      TRUNCATE TABLE employees;
      ```
- **ความแตกต่างกับ DROP**:
    - `DELETE`: ลบข้อมูลเท่านั้น โครงสร้างตารางยังคงอยู่
    - `DROP TABLE`: ลบทั้งตารางและโครงสร้าง

### การใช้งานใน PL/SQL:
สามารถใช้ `DELETE FROM` ภายในบล็อก PL/SQL เพื่อควบคุมการลบหรือจัดการข้อผิดพลาด เช่น:
```sql
BEGIN
   DELETE FROM employees
   WHERE employee_id = 3;
   
   IF SQL%ROWCOUNT = 0 THEN
      DBMS_OUTPUT.PUT_LINE('No rows deleted.');
   ELSE
      DBMS_OUTPUT.PUT_LINE(SQL%ROWCOUNT || ' row(s) deleted.');
   END IF;
   
   COMMIT;
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
      ROLLBACK;
END;
/
```
- `SQL%ROWCOUNT` ใช้ตรวจสอบจำนวนแถวที่ถูกลบ
- จัดการข้อผิดพลาดและควบคุมธุรกรรมด้วย `COMMIT` หรือ `ROLLBACK`

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการลบพนักงานที่เงินเดือนต่ำกว่า 55,000 หรือพนักงานที่ออกจากงาน:
```sql
-- ตรวจสอบข้อมูลก่อน
SELECT employee_id, first_name, salary FROM employees;

-- ลบพนักงานที่มีเงินเดือนต่ำกว่า 55,000
DELETE FROM employees
WHERE salary < 55000;

-- ลบพนักงานที่ออกจากงาน (สมมติมีคอลัมน์ status)
DELETE FROM employees
WHERE status = 'Inactive';

COMMIT;
-- บันทึกการเปลี่ยนแปลง
```

### การจัดการ Foreign Key:
หากตาราง `employees` มี Foreign Key อ้างอิง `departments` และต้องการลบข้อมูล:
```sql
-- ลบข้อมูลในตารางลูกก่อน (ถ้าไม่มี ON DELETE CASCADE)
DELETE FROM employees
WHERE department_id = 10;

-- จากนั้นลบในตารางหลัก
DELETE FROM departments
WHERE department_id = 10;

COMMIT;
```
---

## การใช้ `COMMIT` และ `ROLLBACK`
ใน SQL และ PL/SQL คำสั่ง **COMMIT** และ **ROLLBACK** ใช้สำหรับ**จัดการธุรกรรม** (Transaction) เพื่อควบคุมความสมบูรณ์ของข้อมูลในฐานข้อมูล ทั้งสองเป็นส่วนหนึ่งของ TCL (Transaction Control Language) โดยช่วยให้มั่นใจว่าการเปลี่ยนแปลงข้อมูล (เช่น การใช้ `INSERT`, `UPDATE`, `DELETE`) จะถูกบันทึกถาวรหรือยกเลิกตามต้องการ

### ความหมาย:
1. **COMMIT**:
    - บันทึกการเปลี่ยนแปลงทั้งหมดในธุรกรรมปัจจุบันลงในฐานข้อมูลอย่างถาวร
    - เมื่อทำ `COMMIT` ข้อมูลที่เปลี่ยนแปลงจะไม่สามารถยกเลิกได้ด้วย `ROLLBACK`
    - ใช้เมื่อต้องการยืนยันว่าการเปลี่ยนแปลงถูกต้องและสมบูรณ์

2. **ROLLBACK**:
    - ยกเลิกการเปลี่ยนแปลงทั้งหมดในธุรกรรมปัจจุบัน กลับไปยังสถานะก่อนเริ่มธุรกรรม
    - ใช้เมื่อเกิดข้อผิดพลาดหรือไม่ต้องการให้การเปลี่ยนแปลงมีผลถาวร
    - สามารถยกเลิกได้เฉพาะการเปลี่ยนแปลงที่ยังไม่ได้ `COMMIT`

### ไวยากรณ์:
```sql
COMMIT;
```
```sql
ROLLBACK;
```

### การทำงานของธุรกรรม:
- **ธุรกรรม (Transaction)**: กลุ่มคำสั่ง SQL (เช่น `INSERT`, `UPDATE`, `DELETE`) ที่ทำงานเป็นหน่วยเดียว ถ้าสำเร็จทั้งหมดจะบันทึก ถ้ามีข้อผิดพลาดสามารถยกเลิกได้
- ธุรกรรมเริ่มต้นเมื่อ:
    - เริ่มคำสั่ง DML (`INSERT`, `UPDATE`, `DELETE`) คำสั่งแรก
    - หลังจาก `COMMIT` หรือ `ROLLBACK` ที่สิ้นสุดธุรกรรมก่อนหน้า
- ธุรกรรมสิ้นสุดเมื่อ:
    - ใช้ `COMMIT` (บันทึกถาวร)
    - ใช้ `ROLLBACK` (ยกเลิก)
    - เกิดคำสั่ง DDL (`CREATE`, `ALTER`, `DROP`) ซึ่งมักทำ `COMMIT` อัตโนมัติในบางระบบ (เช่น Oracle)
    - ปิดเซสชัน (เช่น ออกจาก SQL*Plus) ซึ่งอาจทำ `COMMIT` หรือ `ROLLBACK` ขึ้นอยู่กับการตั้งค่า

### ตัวอย่างการใช้งาน:
สมมติมีตาราง `employees`:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    salary NUMBER(10,2)
);
INSERT INTO employees VALUES (1, 'John', 50000.00);
COMMIT;
```

1. **ใช้ COMMIT เพื่อบันทึกการเปลี่ยนแปลง**:
   ```sql
   INSERT INTO employees VALUES (2, 'Jane', 60000.00);
   UPDATE employees SET salary = 55000.00 WHERE employee_id = 1;
   COMMIT;
   ```
    - เพิ่มพนักงานใหม่และอัปเดตเงินเดือนของ `employee_id = 1`
    - `COMMIT` ทำให้การเปลี่ยนแปลงทั้งสองถาวร

2. **ใช้ ROLLBACK เพื่อยกเลิกการเปลี่ยนแปลง**:
   ```sql
   INSERT INTO employees VALUES (3, 'Alice', 55000.00);
   UPDATE employees SET salary = 70000.00 WHERE employee_id = 2;
   -- ตรวจสอบข้อมูล
   SELECT * FROM employees;
   -- ตัดสินใจยกเลิก
   ROLLBACK;
   ```
    - เพิ่มพนักงานและอัปเดตเงินเดือน
    - `ROLLBACK` ยกเลิกการเปลี่ยนแปลงทั้งสอง ทำให้ตารางกลับไปเหมือนก่อนเริ่มธุรกรรม

3. **จัดการข้อผิดพลาด**:
   ```sql
   INSERT INTO employees VALUES (4, 'Bob', 65000.00);
   UPDATE employees SET salary = 80000.00 WHERE employee_id = 999; -- ไม่มี employee_id นี้
   COMMIT;
   ```
    - การ `UPDATE` ไม่มีผล (เพราะไม่มี `employee_id = 999`) แต่การ `INSERT` จะบันทึกถาวรเมื่อ `COMMIT`

### การใช้งานใน PL/SQL:
ใน PL/SQL สามารถใช้ `COMMIT` และ `ROLLBACK` เพื่อควบคุมธุรกรรมภายในบล็อกโค้ด และจัดการข้อผิดพลาดได้ เช่น:
```sql
BEGIN
   INSERT INTO employees VALUES (5, 'Carol', 52000.00);
   UPDATE employees SET salary = 75000.00 WHERE employee_id = 1;
   
   IF SQL%ROWCOUNT > 0 THEN
      DBMS_OUTPUT.PUT_LINE('Update successful: ' || SQL%ROWCOUNT || ' row(s) affected.');
      COMMIT; -- บันทึกถาวรถ้าการอัปเดตสำเร็จ
   ELSE
      DBMS_OUTPUT.PUT_LINE('No rows updated.');
      ROLLBACK; -- ยกเลิกถ้าไม่มีแถวถูกอัปเดต
   END IF;
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
      ROLLBACK; -- ยกเลิกถ้ามีข้อผิดพลาด
END;
/
```

### การใช้ SAVEPOINT:
**SAVEPOINT** ช่วยกำหนดจุดในธุรกรรมเพื่อให้สามารถ `ROLLBACK` ไปยังจุดนั้นได้โดยไม่ยกเลิกทั้งธุรกรรม
```sql
SAVEPOINT point_name;
ROLLBACK TO point_name;
```

**ตัวอย่าง**:
```sql
INSERT INTO employees VALUES (6, 'David', 58000.00);
SAVEPOINT after_insert;
UPDATE employees SET salary = 90000.00 WHERE employee_id = 6;
-- ตรวจสอบและตัดสินใจยกเลิกการอัปเดต
ROLLBACK TO after_insert;
-- การ INSERT ยังคงอยู่ แต่การ UPDATE ถูกยกเลิก
COMMIT;
```

### ข้อควรระวัง:
- **COMMIT อัตโนมัติ**: ในบางระบบ (เช่น Oracle) คำสั่ง DDL (`CREATE`, `ALTER`, `DROP`) จะทำ `COMMIT` อัตโนมัติ ทำให้ไม่สามารถ `ROLLBACK` การเปลี่ยนแปลงก่อนหน้าได้
    - ตัวอย่าง:
      ```sql
      INSERT INTO employees VALUES (7, 'Emma', 54000.00);
      CREATE TABLE temp (id NUMBER); -- ทำ COMMIT อัตโนมัติ
      ROLLBACK; -- จะไม่สามารถยกเลิก INSERT ได้
      ```
- **การตั้งค่า Autocommit**:
    - ในเครื่องมือบางตัว (เช่น SQL*Plus, SQL Developer) การตั้งค่า Autocommit อาจทำให้ทุกคำสั่ง DML ทำ `COMMIT` อัตโนมัติ
    - ปิด Autocommit ใน SQL*Plus:
      ```sql
      SET AUTOCOMMIT OFF;
      ```
- **Foreign Key Constraints**: การลบหรืออัปเดตข้อมูลที่มี Foreign Key อาจทำให้เกิดข้อผิดพลาดถ้าละเมิดข้อจำกัด ควรตรวจสอบก่อนและใช้ `ROLLBACK` หากจำเป็น
- **ประสิทธิภาพ**: การทำธุรกรรมขนาดใหญ่ (เช่น อัปเดตหรือลบข้อมูลจำนวนมาก) ควรแบ่งเป็นธุรกรรมย่อย ๆ และใช้ `COMMIT` ระหว่างทางเพื่อลดการใช้ทรัพยากร
- **การออกจากเซสชัน**:
    - ถ้าเซสชันปิดโดยไม่ใช้ `COMMIT` หรือ `ROLLBACK` ผลลัพธ์ขึ้นอยู่กับระบบ:
        - Oracle: มักทำ `ROLLBACK` อัตโนมัติ
        - MySQL: ขึ้นอยู่กับการตั้งค่า (อาจ `COMMIT` ถ้า Autocommit เปิด)

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการเพิ่มและอัปเดตข้อมูลพนักงาน แต่ต้องตรวจสอบก่อนบันทึก:
```sql
-- เริ่มธุรกรรม
INSERT INTO employees VALUES (8, 'Frank', 53000.00);
SAVEPOINT after_insert;
UPDATE employees SET salary = 80000.00 WHERE employee_id = 8;

-- ตรวจสอบข้อมูล
SELECT * FROM employees WHERE employee_id = 8;

-- ถ้าผลลัพธ์ไม่ถูกต้อง
ROLLBACK TO after_insert;
-- หรือถ้าถูกต้อง
COMMIT;
```

### ความแตกต่างกับระบบฐานข้อมูลอื่น:
- **MySQL**: รองรับ `COMMIT` และ `ROLLBACK` แต่ต้องใช้เอนจินที่รองรับธุรกรรม เช่น InnoDB (MyISAM ไม่รองรับ)
  ```sql
  START TRANSACTION;
  INSERT INTO employees VALUES (9, 'Grace', 57000.00);
  COMMIT;
  ```
- **PostgreSQL**: คล้าย Oracle รองรับ `SAVEPOINT`, `COMMIT`, `ROLLBACK` และ `START TRANSACTION`
- **SQL Server**: ใช้ `BEGIN TRANSACTION` แทน `START TRANSACTION` และรองรับ `COMMIT` และ `ROLLBACK`
---
