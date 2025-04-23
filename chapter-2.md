# บทที่ 2: การสร้างและจัดการตาราง (CREATE, ALTER, DROP)

## สารบัญ

- [การสร้างตารางด้วย CREATE TABLE](#การสร้างตารางด้วย-create-table)
- [การเปลี่ยนโครงสร้างตารางด้วย ALTER TABLE](#การเปลี่ยนโครงสร้างตารางด้วย-alter-table)
- [การลบตารางด้วย DROP TABLE](#การลบตารางด้วย-drop-table)

## การสร้างตารางด้วย `CREATE TABLE`
การสร้างตารางใน SQL ใช้คำสั่ง **CREATE TABLE** เพื่อกำหนดโครงสร้างของตารางในฐานข้อมูล โดยระบุชื่อตาราง, คอลัมน์, ชนิดข้อมูล (Data Type), และข้อจำกัด (Constraints) ต่าง ๆ ตามต้องการ

### ไวยากรณ์พื้นฐาน:
```sql
CREATE TABLE table_name (
    column1 datatype [constraint],
    column2 datatype [constraint],
    ...
);
```

### อธิบายส่วนประกอบ:
- **table_name**: ชื่อของตาราง (ต้องไม่ซ้ำกับตารางอื่นในฐานข้อมูล)
- **column**: ชื่อคอลัมน์ในตาราง
- **datatype**: ชนิดข้อมูลของคอลัมน์ เช่น `INT`, `VARCHAR`, `DATE`, `DECIMAL`
- **constraint**: ข้อจำกัด เช่น `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`, `UNIQUE`, `CHECK`

### ตัวอย่างการสร้างตาราง:
1. **ตารางพื้นฐาน**:
   ```sql
   CREATE TABLE employees (
       employee_id INT,
       first_name VARCHAR(50),
       last_name VARCHAR(50),
       hire_date DATE
   );
   ```
    - สร้างตาราง `employees` ที่มีคอลัมน์ `employee_id` (ตัวเลข), `first_name` และ `last_name` (ข้อความ), และ `hire_date` (วันที่)

2. **ตารางที่มีข้อจำกัด (Constraints)**:
   ```sql
   CREATE TABLE employees (
       employee_id INT PRIMARY KEY,
       first_name VARCHAR(50) NOT NULL,
       last_name VARCHAR(50) NOT NULL,
       email VARCHAR(100) UNIQUE,
       salary DECIMAL(10,2) CHECK (salary > 0),
       department_id INT,
       FOREIGN KEY (department_id) REFERENCES departments(department_id)
   );
   ```
    - **PRIMARY KEY**: กำหนด `employee_id` เป็นคีย์หลัก (ไม่ซ้ำและไม่เป็น NULL)
    - **NOT NULL**: `first_name` และ `last_name` ต้องมีค่าเสมอ
    - **UNIQUE**: `email` ต้องไม่ซ้ำกัน
    - **CHECK**: `salary` ต้องมากกว่า 0
    - **FOREIGN KEY**: `department_id` อ้างอิงไปยังคอลัมน์ `department_id` ในตาราง `departments`

3. **ตารางที่มีค่าเริ่มต้น (Default Value)**:
   ```sql
   CREATE TABLE products (
       product_id INT PRIMARY KEY,
       product_name VARCHAR(100) NOT NULL,
       price DECIMAL(10,2) DEFAULT 0.00,
       created_at DATE DEFAULT CURRENT_DATE
   );
   ```
    - **DEFAULT**: หากไม่ระบุค่า `price` จะใช้ 0.00 และ `created_at` จะใช้วันที่ปัจจุบัน

### ข้อควรรู้:
- **ชนิดข้อมูล (Data Types)**: ขึ้นอยู่กับระบบฐานข้อมูล เช่น Oracle, MySQL, PostgreSQL ตัวอย่างทั่วไป:
    - `INT` หรือ `INTEGER`: ตัวเลขจำนวนเต็ม
    - `VARCHAR(n)`: ข้อความความยาวตัวแปร (n คือความยาวสูงสุด)
    - `DECIMAL(p,s)`: ตัวเลขทศนิยม (p คือจำนวนหลักทั้งหมด, s คือจำนวนทศนิยม)
    - `DATE`: วันที่
- **Constraints**:
    - `PRIMARY KEY`: คีย์หลักสำหรับระบุแถว
    - `FOREIGN KEY`: ลิงก์ไปยังตารางอื่น
    - `NOT NULL`: ห้ามเป็นค่าว่าง
    - `UNIQUE`: ค่าต้องไม่ซ้ำ
    - `CHECK`: ตรวจสอบเงื่อนไข
- ตรวจสอบว่าตารางชื่อเดียวกันมีอยู่แล้วหรือไม่ เพราะคำสั่ง `CREATE TABLE` จะเกิดข้อผิดพลาดถ้าชื่อซ้ำ
- ในบางระบบฐานข้อมูล สามารถเพิ่มตัวเลือก เช่น `IF NOT EXISTS` เพื่อป้องกันข้อผิดพลาด:
  ```sql
  CREATE TABLE IF NOT EXISTS employees (...);
  ```

### ตัวอย่างการใช้งานจริง:
สมมติต้องการสร้างตารางสำหรับระบบจัดการพนักงาน:
```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50) NOT NULL
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    hire_date DATE,
    salary DECIMAL(10,2) CHECK (salary >= 0),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);
```
- สร้างตาราง `departments` เพื่อเก็บข้อมูลแผนก
- สร้างตาราง `employees` ที่เชื่อมโยงกับ `departments` ผ่าน `department_id`
---

## การเปลี่ยนโครงสร้างตารางด้วย `ALTER TABLE`
คำสั่ง **ALTER TABLE** ใน SQL ใช้สำหรับ**เปลี่ยนแปลงโครงสร้าง**ของตารางที่มีอยู่ในฐานข้อมูล เช่น การเพิ่ม/ลบคอลัมน์, แก้ไขชนิดข้อมูล, เพิ่ม/ลบข้อจำกัด (Constraints) หรือเปลี่ยนชื่อตาราง ต่อไปนี้คือรายละเอียดและตัวอย่างการใช้งาน

### ไวยากรณ์พื้นฐาน:
```sql
ALTER TABLE table_name
[modification_action];
```

### การกระทำที่พบบ่อย:
1. **เพิ่มคอลัมน์ใหม่**:
   ```sql
   ALTER TABLE table_name
   ADD column_name datatype [constraint];
   ```
    - ตัวอย่าง: เพิ่มคอลัมน์ `phone_number` ในตาราง `employees`
      ```sql
      ALTER TABLE employees
      ADD phone_number VARCHAR(15);
      ```

2. **แก้ไขคอลัมน์ที่มีอยู่** (เปลี่ยนชนิดข้อมูลหรือข้อจำกัด):
   ```sql
   ALTER TABLE table_name
   MODIFY column_name new_datatype [new_constraint];
   ```
    - ตัวอย่าง: เปลี่ยนชนิดข้อมูลของ `phone_number` เป็น `VARCHAR(20)`
      ```sql
      ALTER TABLE employees
      MODIFY phone_number VARCHAR(20);
      ```
    - **หมายเหตุ**: การเปลี่ยนชนิดข้อมูลอาจถูกจำกัดถ้าข้อมูลในคอลัมน์ไม่เข้ากันกับชนิดใหม่

3. **ลบคอลัมน์**:
   ```sql
   ALTER TABLE table_name
   DROP COLUMN column_name;
   ```
    - ตัวอย่าง: ลบคอลัมน์ `phone_number`
      ```sql
      ALTER TABLE employees
      DROP COLUMN phone_number;
      ```

4. **เพิ่มข้อจำกัด (Constraints)**:
   ```sql
   ALTER TABLE table_name
   ADD CONSTRAINT constraint_name constraint_type (column_name);
   ```
    - ตัวอย่าง: เพิ่มข้อจำกัด `UNIQUE` ให้คอลัมน์ `email`
      ```sql
      ALTER TABLE employees
      ADD CONSTRAINT unique_email UNIQUE (email);
      ```
    - ตัวอย่าง: เพิ่ม `FOREIGN KEY`
      ```sql
      ALTER TABLE employees
      ADD CONSTRAINT fk_department FOREIGN KEY (department_id) REFERENCES departments(department_id);
      ```

5. **ลบข้อจำกัด**:
   ```sql
   ALTER TABLE table_name
   DROP CONSTRAINT constraint_name;
   ```
    - ตัวอย่าง: ลบข้อจำกัด `unique_email`
      ```sql
      ALTER TABLE employees
      DROP CONSTRAINT unique_email;
      ```

6. **เปลี่ยนชื่อคอลัมน์**:
   ```sql
   ALTER TABLE table_name
   RENAME COLUMN old_column_name TO new_column_name;
   ```
    - ตัวอย่าง: เปลี่ยนชื่อคอลัมน์ `first_name` เป็น `given_name`
      ```sql
      ALTER TABLE employees
      RENAME COLUMN first_name TO given_name;
      ```

7. **เปลี่ยนชื่อตาราง**:
   ```sql
   ALTER TABLE table_name
   RENAME TO new_table_name;
   ```
    - ตัวอย่าง: เปลี่ยนชื่อตาราง `employees` เป็น `staff`
      ```sql
      ALTER TABLE employees
      RENAME TO staff;
      ```

### ตัวอย่างการใช้งานจริง:
สมมติมีตาราง `employees` ดังนี้:
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    salary DECIMAL(10,2)
);
```

ต้องการปรับปรุงโครงสร้างตาราง:
```sql
-- เพิ่มคอลัมน์ใหม่
ALTER TABLE employees
ADD email VARCHAR(100);

-- แก้ไขชนิดข้อมูลของ salary
ALTER TABLE employees
MODIFY salary DECIMAL(12,2);

-- เพิ่มข้อจำกัด UNIQUE สำหรับ email
ALTER TABLE employees
ADD CONSTRAINT unique_email UNIQUE (email);

-- เพิ่มคอลัมน์ department_id และ FOREIGN KEY
ALTER TABLE employees
ADD department_id INT;

ALTER TABLE employees
ADD CONSTRAINT fk_department FOREIGN KEY (department_id) REFERENCES departments(department_id);

-- เปลี่ยนชื่อคอลัมน์
ALTER TABLE employees
RENAME COLUMN first_name TO given_name;

-- ลบคอลัมน์ที่ไม่ต้องการ
ALTER TABLE employees
DROP COLUMN last_name;

-- เปลี่ยนชื่อตาราง
ALTER TABLE employees
RENAME TO staff;
```

### ข้อควรระวัง:
- **ความเข้ากันได้ของข้อมูล**: การเปลี่ยนชนิดข้อมูลหรือเพิ่มข้อจำกัด (เช่น `NOT NULL`) ต้องตรวจสอบว่าข้อมูลที่มีอยู่เข้ากันได้ มิฉะนั้นอาจเกิดข้อผิดพลาด
    - ตัวอย่าง: หากเพิ่ม `NOT NULL` ให้คอลัมน์ที่มีข้อมูลว่าง ต้องอัปเดตข้อมูลก่อน
      ```sql
      UPDATE employees SET email = 'unknown@example.com' WHERE email IS NULL;
      ALTER TABLE employees MODIFY email VARCHAR(100) NOT NULL;
      ```
- **ข้อจำกัดของระบบฐานข้อมูล**: ไวยากรณ์อาจแตกต่างกันเล็กน้อยในระบบฐานข้อมูล เช่น Oracle, MySQL, PostgreSQL, SQL Server (เช่น Oracle ใช้ `ALTER TABLE ... DROP CONSTRAINT` แต่ MySQL อาจใช้ `ALTER TABLE ... DROP FOREIGN KEY`)
- **ผลกระทบต่อแอปพลิเคชัน**: การเปลี่ยนโครงสร้างตารางอาจกระทบกับโค้ดหรือแอปพลิเคชันที่ใช้งานตารางนั้น
- **การสำรองข้อมูล**: ควรสำรองข้อมูลก่อนทำการเปลี่ยนแปลงโครงสร้าง เพื่อป้องกันการสูญหายของข้อมูล

### การใช้งานใน PL/SQL:
ใน PL/SQL สามารถใช้ `ALTER TABLE` ภายในบล็อก PL/SQL ได้ เช่น:
```sql
BEGIN
   EXECUTE IMMEDIATE 'ALTER TABLE employees ADD status VARCHAR(10) DEFAULT ''Active''';
   DBMS_OUTPUT.PUT_LINE('Table altered successfully.');
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/
```
- ใช้ `EXECUTE IMMEDIATE` สำหรับคำสั่ง DDL ใน PL/SQL เนื่องจาก DDL ไม่สามารถรันโดยตรงในบล็อก PL/SQL
---

## การลบตารางด้วย `DROP TABLE`
คำสั่ง **DROP TABLE** ใน SQL ใช้สำหรับ**ลบตาราง**และข้อมูลทั้งหมดในตารางนั้นออกจากฐานข้อมูลอย่างถาวร รวมถึงโครงสร้างตาราง (คอลัมน์, ข้อจำกัด) และดัชนี (Indexes) ที่เกี่ยวข้อง คำสั่งนี้เป็นส่วนหนึ่งของ DDL (Data Definition Language) และไม่สามารถกู้คืนได้เว้นแต่จะมีการสำรองข้อมูลหรือใช้กลไกพิเศษของฐานข้อมูล

### ไวยากรณ์พื้นฐาน:
```sql
DROP TABLE table_name [option];
```

### อธิบายส่วนประกอบ:
- **table_name**: ชื่อตารางที่ต้องการลบ
- **option**: ตัวเลือกเพิ่มเติม (ขึ้นอยู่กับระบบฐานข้อมูล) เช่น:
    - `CASCADE`: ลบตารางและวัตถุที่เกี่ยวข้อง (เช่น Foreign Key Constraints) อัตโนมัติ
    - `RESTRICT`: ป้องกันการลบถ้ามีวัตถุที่เกี่ยวข้อง
    - `IF EXISTS`: ป้องกันข้อผิดพลาดถ้าตาราง不存在 (ใช้ได้ในบางระบบ เช่น MySQL, PostgreSQL)

### ตัวอย่างการใช้งาน:
1. **ลบตารางพื้นฐาน**:
   ```sql
   DROP TABLE employees;
   ```
    - ลบตาราง `employees` และข้อมูลทั้งหมดอย่างถาวร

2. **ใช้ IF EXISTS เพื่อป้องกันข้อผิดพลาด**:
   ```sql
   DROP TABLE IF EXISTS employees;
   ```
    - ถ้าตาราง `employees` ไม่มีอยู่ คำสั่งจะไม่เกิดข้อผิดพลาด (รองรับใน MySQL, PostgreSQL, SQL Server)

3. **ใช้ CASCADE เพื่อลบตารางที่มีความสัมพันธ์**:
   ```sql
   DROP TABLE employees CASCADE;
   ```
    - ลบตาราง `employees` และยกเลิกข้อจำกัด Foreign Key ที่ตารางอื่นอ้างอิงถึง (ใช้ใน Oracle, PostgreSQL)

4. **ลบหลายตารางพร้อมกัน** (รองรับในบางระบบ เช่น PostgreSQL):
   ```sql
   DROP TABLE employees, departments;
   ```

### ข้อควรระวัง:
- **การลบถาวร**: คำสั่ง `DROP TABLE` จะลบตารางและข้อมูลทั้งหมดอย่างถาวร ไม่สามารถกู้คืนได้เว้นแต่มีกลไกอย่าง Recycle Bin (ใน Oracle) หรือการสำรองข้อมูล
- **ความสัมพันธ์กับตารางอื่น**: ถ้าตารางมี Foreign Key อ้างอิงจากตารางอื่น อาจต้องใช้ `CASCADE` หรือลบข้อจำกัดก่อน
    - ตัวอย่าง: ถ้าตาราง `employees` มี `department_id` ที่เป็น Foreign Key อ้างอิง `departments` ต้องลบข้อจำกัดก่อน หรือใช้ `CASCADE`
- **สิทธิ์การใช้งาน**: ผู้ใช้ต้องมีสิทธิ์เพียงพอในการลบตาราง (เช่น `DROP TABLE` privilege ใน Oracle)
- **ผลกระทบต่อแอปพลิเคชัน**: การลบตารางอาจทำให้โค้ดหรือแอปพลิเคชันที่ใช้งานตารางนั้นเกิดข้อผิดพลาด
- **Recycle Bin ใน Oracle**: ใน Oracle ถ้าลบตารางโดยไม่ได้ใช้ `PURGE` ตารางจะถูกย้ายไปที่ Recycle Bin และสามารถกู้คืนได้
    - ตัวอย่างการลบถาวร:
      ```sql
      DROP TABLE employees PURGE;
      ```
    - ตัวอย่างการกู้คืน (ถ้ายังอยู่ใน Recycle Bin):
      ```sql
      FLASHBACK TABLE employees TO BEFORE DROP;
      ```

### การใช้งานใน PL/SQL:
ใน PL/SQL สามารถใช้ `DROP TABLE` ภายในบล็อก PL/SQL โดยใช้ `EXECUTE IMMEDIATE` เนื่องจากเป็นคำสั่ง DDL เช่น:
```sql
BEGIN
   EXECUTE IMMEDIATE 'DROP TABLE employees';
   DBMS_OUTPUT.PUT_LINE('Table dropped successfully.');
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/
```
- ใช้ `EXECUTE IMMEDIATE` เพื่อรันคำสั่ง DDL
- จัดการข้อผิดพลาดด้วยบล็อก `EXCEPTION`

### ความแตกต่างกับคำสั่งอื่น:
- **DROP TABLE** vs **TRUNCATE TABLE**:
    - `DROP TABLE`: ลบทั้งตารางและโครงสร้าง
    - `TRUNCATE TABLE`: ลบเฉพาะข้อมูลในตาราง แต่คงโครงสร้างไว้
      ```sql
      TRUNCATE TABLE employees;
      ```
- **DROP TABLE** vs **DELETE**:
    - `DELETE`: ลบข้อมูลบางส่วนหรือทั้งหมดในตาราง (DML) แต่โครงสร้างยังคงอยู่ และสามารถกู้คืนได้ด้วย `ROLLBACK`
      ```sql
      DELETE FROM employees;
      ```

### ตัวอย่างสถานการณ์จริง:
สมมติมีตาราง `employees` และ `departments` ที่เกี่ยวข้องกัน:
```sql
-- ลบตาราง departments ที่มี Foreign Key อ้างอิง
DROP TABLE departments CASCADE;

-- ลบตาราง employees อย่างถาวร
DROP TABLE employees PURGE;
```
---

## การใช้ `DESCRIBE` เพื่อตรวจสอบโครงสร้างตาราง
คำสั่ง **DESCRIBE** (หรือย่อว่า **DESC**) ใน SQL ใช้สำหรับ**ตรวจสอบโครงสร้าง**ของตารางในฐานข้อมูล โดยจะแสดงข้อมูลเกี่ยวกับคอลัมน์ของตาราง เช่น ชื่อคอลัมน์, ชนิดข้อมูล (Data Type), และข้อจำกัด (เช่น `NOT NULL`) คำสั่งนี้มีประโยชน์ในการตรวจสอบรายละเอียดของตารางก่อนทำการแก้ไขหรือใช้งาน โดยเฉพาะใน Oracle SQL และระบบฐานข้อมูลบางตัว

### ไวยากรณ์:
```sql
DESCRIBE table_name;
```
หรือ
```sql
DESC table_name;
```

### ผลลัพธ์:
เมื่อรันคำสั่ง `DESCRIBE` ระบบจะแสดง:
- **ชื่อคอลัมน์** (Column Name)
- **ชนิดข้อมูล** (Data Type) เช่น `VARCHAR2`, `NUMBER`, `DATE`
- **ข้อจำกัด** เช่น `NOT NULL` (ถ้ามี)
- **ขนาด** (ถ้ามี เช่น ความยาวของ `VARCHAR2` หรือความแม่นยำของ `NUMBER`)

**หมายเหตุ**: รูปแบบผลลัพธ์อาจแตกต่างกันเล็กน้อยขึ้นอยู่กับระบบฐานข้อมูล

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

1. **ตรวจสอบโครงสร้างตาราง**:
   ```sql
   DESCRIBE employees;
   ```
   หรือ
   ```sql
   DESC employees;
   ```

   **ผลลัพธ์** (ใน Oracle):
   ```
   Name           Null?    Type
   -------------- -------- ----------------
   EMPLOYEE_ID    NOT NULL NUMBER(10)
   FIRST_NAME     NOT NULL VARCHAR2(50)
   LAST_NAME      NOT NULL VARCHAR2(50)
   EMAIL                   VARCHAR2(100)
   SALARY                  NUMBER(10,2)
   HIRE_DATE               DATE
   ```
    - แสดงชื่อคอลัมน์, ข้อจำกัด `NOT NULL`, และชนิดข้อมูล
    - ข้อจำกัด `PRIMARY KEY` และ `UNIQUE` อาจไม่แสดงในผลลัพธ์โดยตรง ต้องตรวจสอบผ่านมุมมองระบบ (เช่น `USER_CONSTRAINTS`)

2. **ใช้ใน PL/SQL**:
   สามารถเรียก `DESCRIBE` ในเครื่องมืออย่าง SQL*Plus หรือ SQL Developer ได้โดยตรง แต่ในบล็อก PL/SQL คำสั่ง `DESCRIBE` ไม่สามารถใช้ในโค้ดได้ ต้องใช้มุมมองระบบแทน เช่น:
   ```sql
   BEGIN
       FOR col IN (SELECT column_name, data_type, data_length, nullable
                   FROM user_tab_columns
                   WHERE table_name = 'EMPLOYEES')
       LOOP
           DBMS_OUTPUT.PUT_LINE(col.column_name || ' ' || col.data_type || 
                                ' Nullable: ' || col.nullable);
       END LOOP;
   END;
   /
   ```
    - ใช้ตาราง `USER_TAB_COLUMNS` เพื่อดึงข้อมูลโครงสร้างตาราง

### ข้อควรรู้:
- **ข้อจำกัดของ DESCRIBE**: แสดงเฉพาะข้อมูลพื้นฐาน (ชื่อคอลัมน์, ชนิดข้อมูล, `NOT NULL`) ไม่แสดงรายละเอียดข้อจำกัดเช่น `PRIMARY KEY`, `FOREIGN KEY`, หรือ `CHECK` ต้องใช้มุมมองระบบ เช่น:
    - Oracle: `USER_CONSTRAINTS`, `USER_CONS_COLUMNS`
    - MySQL: `INFORMATION_SCHEMA.KEY_COLUMN_USAGE`
- **กรณีตารางไม่มีอยู่**: หากตารางไม่มีอยู่ คำสั่ง `DESCRIBE` จะส่งข้อผิดพลาด (เช่น ใน Oracle: `ORA-04043: object does not exist`)
- **การใช้งานในเครื่องมือ**: คำสั่ง `DESCRIBE` มักใช้ในเครื่องมือแบบ Interactive เช่น SQL*Plus, SQL Developer, หรือ MySQL CLI ไม่เหมาะสำหรับสคริปต์อัตโนมัติ (ควรใช้มุมมองระบบแทน)

### ตัวอย่างการตรวจสอบข้อจำกัดเพิ่มเติม (Oracle):
หากต้องการดู `PRIMARY KEY` หรือ `FOREIGN KEY`:
```sql
SELECT constraint_name, constraint_type, column_name
FROM user_constraints c
JOIN user_cons_columns cc ON c.constraint_name = cc.constraint_name
WHERE c.table_name = 'EMPLOYEES';
```
- `constraint_type`: `P` (Primary Key), `U` (Unique), `R` (Foreign Key), `C` (Check)

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการตรวจสอบตาราง `employees` ก่อนเพิ่มคอลัมน์ใหม่:
```sql
DESC employees;
-- ตรวจสอบว่าไม่มีคอลัมน์ phone_number
ALTER TABLE employees
ADD phone_number VARCHAR2(15);
DESC employees;
-- ตรวจสอบอีกครั้งเพื่อยืนยันว่าคอลัมน์ phone_number ถูกเพิ่ม
```
---
