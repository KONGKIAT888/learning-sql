# บทที่ 10: การจัดการสิทธิ์และการควบคุมธุรกรรม

## สารบัญ

- [การใช้ GRANT, REVOKE เพื่อจัดการสิทธิ์ผู้ใช้](#การใช้-grant-revoke-เพื่อจัดการสิทธิ์ผู้ใช้)
- [การควบคุมธุรกรรมด้วย COMMIT, ROLLBACK, SAVEPOINT](#การควบคุมธุรกรรมด้วย-commit-rollback-savepoint)
- [ความเข้าใจเกี่ยวกับ ACID properties](#ความเข้าใจเกี่ยวกับ-acid-properties)

## การใช้ `GRANT`, `REVOKE` เพื่อจัดการสิทธิ์ผู้ใช้
ใน SQL คำสั่ง **GRANT** และ **REVOKE** ใช้สำหรับ**จัดการสิทธิ์การเข้าถึง** (Privileges) ของผู้ใช้หรือบทบาท (Role) ในฐานข้อมูล เพื่อควบคุมว่าใครสามารถเข้าถึงหรือดำเนินการใด ๆ กับวัตถุในฐานข้อมูล เช่น ตาราง, View, Sequence, หรือ Procedure ได้บ้าง คำสั่งเหล่านี้มีความสำคัญในด้านความปลอดภัยและการบริหารจัดการฐานข้อมูล โดยเฉพาะใน Oracle SQL ซึ่งมีการควบคุมสิทธิ์ที่ละเอียด ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และเปรียบเทียบกับระบบฐานข้อมูลทั่วไป)

### คำอธิบายของ GRANT และ REVOKE
- **GRANT**: ให้สิทธิ์การเข้าถึงหรือดำเนินการแก่ผู้ใช้หรือบทบาท
    - เช่น อนุญาตให้ผู้ใช้เลือก (SELECT), แทรก (INSERT), หรือเรียกใช้ (EXECUTE) วัตถุในฐานข้อมูล
    - สามารถให้สิทธิ์เฉพาะบางคอลัมน์หรือบาง操作ได้
- **REVOKE**: เพิกถอนสิทธิ์ที่เคยให้ไว้จากผู้ใช้หรือบทบาท
    - ใช้เมื่อต้องการจำกัดหรือยกเลิกการเข้าถึง
- **ประเภทของสิทธิ์**:
    - **Object Privileges**: สิทธิ์สำหรับวัตถุ เช่น `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `EXECUTE` (สำหรับ Procedure)
    - **System Privileges**: สิทธิ์ระดับระบบ เช่น `CREATE TABLE`, `CREATE USER`, `CREATE SESSION`
    - **Role Privileges**: สิทธิ์ที่มอบผ่านบทบาท (Role) เช่น `DBA`, `CONNECT`, `RESOURCE`
- **เมื่อใช้ GRANT/REVOKE**:
    - จำกัดการเข้าถึงข้อมูลที่ sensitive
    - มอบสิทธิ์ให้ผู้ใช้หรือทีมเฉพาะตามความรับผิดชอบ
    - เพิกถอนสิทธิ์เมื่อผู้ใช้เปลี่ยนบทบาทหรือออกจากองค์กร
    - จัดการการเข้าถึงในระบบที่มีผู้ใช้หลายคน

### ไวยากรณ์พื้นฐาน
#### GRANT
```sql
-- มอบ Object Privilege
GRANT privilege [, privilege ...]
ON object_name
TO user | role | PUBLIC
[WITH GRANT OPTION];

-- มอบ System Privilege
GRANT system_privilege [, system_privilege ...]
TO user | role | PUBLIC
[WITH ADMIN OPTION];

-- มอบ Role
GRANT role [, role ...]
TO user | role | PUBLIC
[WITH ADMIN OPTION];
```

- **privilege**: สิทธิ์ เช่น `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `EXECUTE`
- **object_name**: วัตถุ เช่น ตาราง, View, Sequence, Procedure
- **user | role | PUBLIC**: ผู้ใช้, บทบาท, หรือทุกคน (PUBLIC)
- **WITH GRANT OPTION**: อนุญาตให้ผู้รับสิทธิ์มอบสิทธิ์ต่อให้ผู้อื่น
- **WITH ADMIN OPTION**: อนุญาตให้ผู้รับมอบ System Privilege หรือ Role ให้ผู้อื่น

#### REVOKE
```sql
-- เพิกถอน Object Privilege
REVOKE privilege [, privilege ...]
ON object_name
FROM user | role | PUBLIC;

-- เพิกถอน System Privilege
REVOKE system_privilege [, system_privilege ...]
FROM user | role | PUBLIC;

-- เพิกถอน Role
REVOKE role [, role ...]
FROM user | role | PUBLIC;
```

- การเพิกถอนจะยกเลิกเฉพาะสิทธิ์ที่ระบุและไม่กระทบสิทธิ์อื่นที่ผู้ใช้มี

### ตัวอย่างการใช้งาน
สมมติมีตารางและผู้ใช้:
1. **ตาราง**:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR查看更多(50),
    salary NUMBER(10,2),
    department_id NUMBER(5)
);
INSERT INTO employees VALUES (1, 'John', 50000, 10);
INSERT INTO employees VALUES (2, 'Jane', 60000, 20);
INSERT INTO employees VALUES (3, 'Alice', 55000, 10);
COMMIT;

CREATE TABLE departments (
    department_id NUMBER(5) PRIMARY KEY,
    department_name VARCHAR2(50)
);
INSERT INTO departments VALUES (10, 'Sales');
INSERT INTO departments VALUES (20, 'IT');
COMMIT;
```

2. **สร้างผู้ใช้**:
```sql
CREATE USER hr_user IDENTIFIED BY hr_password;
CREATE USER manager_user IDENTIFIED BY mgr_password;
CREATE USER report_user IDENTIFIED BY rpt_password;
```

3. **ให้สิทธิ์การเชื่อมต่อ**:
```sql
GRANT CREATE SESSION TO hr_user, manager_user, report_user;
```
- **คำอธิบาย**: อนุญาตให้ผู้ใช้ทั้งสามคนล็อกอินเข้าสู่ฐานข้อมูล

#### 1. GRANT Object Privileges
สมมติต้องการให้ `hr_user` อ่านและอัปเดตข้อมูลในตาราง `employees`:
```sql
GRANT SELECT, UPDATE ON employees TO hr_user;
```
- **คำอธิบาย**: `hr_user` สามารถใช้ `SELECT` และ `UPDATE` บนตาราง `employees`
- **ทดสอบ** (ในเซสชันของ `hr_user`):
  ```sql
  SELECT first_name, salary FROM employees;
  UPDATE employees SET salary = salary * 1.1 WHERE department_id = 10;
  ```
    - **ผลลัพธ์**:
      ```
      first_name | salary
      ------------+--------
      John       | 55000
      Jane       | 60000
      Alice      | 60500
      ```

สมมติต้องการให้ `report_user` อ่านข้อมูลจาก `departments` เท่านั้น:
```sql
GRANT SELECT ON departments TO report_user;
```
- **คำอธิบาย**: `report_user` สามารถใช้ `SELECT` บน `departments` แต่ไม่มีสิทธิ์แก้ไข
- **ทดสอบ**:
  ```sql
  SELECT department_name FROM departments; -- สำเร็จ
  INSERT INTO departments VALUES (30, 'HR'); -- ล้มเหลว (ORA-01031: insufficient privileges)
  ```

#### 2. GRANT กับ WITH GRANT OPTION
สมมติต้องการให้ `manager_user` อ่านและแก้ไขตาราง `employees` และมอบสิทธิ์ต่อให้ผู้อื่น:
```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO manager_user WITH GRANT OPTION;
```
- **คำอธิบาย**:
    - `manager_user` มีสิทธิ์ `SELECT`, `INSERT`, `UPDATE`, `DELETE`
    - สามารถมอบสิทธิ์เหล่านี้ให้ผู้อื่นได้
- **ทดสอบ** (ในเซสชันของ `manager_user`):
  ```sql
  GRANT SELECT ON employees TO report_user;
  ```
    - **ผลลัพธ์**: `report_user` ได้รับสิทธิ์ `SELECT` บน `employees`
- **เรียกดูข้อมูลโดย `report_user`**:
  ```sql
  SELECT first_name FROM employees;
  ```

#### 3. GRANT System Privileges
สมมติต้องการให้ `hr_user` สร้างตารางใหม่:
```sql
GRANT CREATE TABLE TO hr_user;
```
- **คำอธิบาย**: `hr_user` สามารถสร้างตารางใน Schema ของตนเอง
- **ทดสอบ**:
  ```sql
  CREATE TABLE hr_user.my_table (id NUMBER, name VARCHAR2(50));
  ```

#### 4. GRANT Role
สมมติต้องการให้ `manager_user` มีบทบาท `RESOURCE` (รวมสิทธิ์เช่น `CREATE TABLE`, `CREATE SEQUENCE`):
```sql
GRANT RESOURCE TO manager_user;
```
- **คำอธิบาย**: `manager_user` ได้รับสิทธิ์ที่รวมอยู่ในบทบาท `RESOURCE`
- **ทดสอบ**:
  ```sql
  CREATE TABLE manager_user.projects (project_id NUMBER, project_name VARCHAR2(100));
  ```

สมมติสร้างบทบาทกำหนดเอง:
```sql
CREATE ROLE report_role;
GRANT SELECT ON employees TO report_role;
GRANT SELECT ON departments TO report_role;
GRANT report_role TO report_user;
```
- **คำอธิบาย**:
    - สร้างบทบาท `report_role` และให้สิทธิ์ `SELECT` บน `employees` และ `departments`
    - มอบบทบาทนี้ให้ `report_user`
- **ทดสอบ**:
  ```sql
  SELECT first_name FROM employees; -- สำเร็จ
  SELECT department_name FROM departments; -- สำเร็จ
  ```

#### 5. REVOKE สิทธิ์
สมมติต้องการยกเลิกสิทธิ์ `UPDATE` ของ `hr_user` บน `employees`:
```sql
REVOKE UPDATE ON employees FROM hr_user;
```
- **คำอธิบาย**: `hr_user` ยังมีสิทธิ์ `SELECT` แต่ไม่สามารถ `UPDATE` ได้อีก
- **ทดสอบ**:
  ```sql
  SELECT first_name FROM employees; -- สำเร็จ
  UPDATE employees SET salary = 60000; -- ล้มเหลว (ORA-01031: insufficient privileges)
  ```

สมมติต้องการยกเลิกบทบาท `report_role` จาก `report_user`:
```sql
REVOKE report_role FROM report_user;
```
- **คำอธิบาย**: `report_user` สูญเสียสิทธิ์ทั้งหมดที่ได้รับจาก `report_role`
- **ทดสอบ**:
  ```sql
  SELECT first_name FROM employees; -- ล้มเหลว (ถ้าไม่มีสิทธิ์อื่น)
  ```

#### 6. REVOKE WITH GRANT OPTION
สมมติต้องการยกเลิกสิทธิ์ที่ `manager_user` มอบต่อให้ผู้อื่น:
```sql
REVOKE SELECT ON employees FROM report_user;
```
- **คำอธิบาย**: เพิกถอนสิทธิ์ `SELECT` ที่ `manager_user` มอบให้ `report_user`
- **หมายเหตุ**: การเพิกถอนสิทธิ์จาก `manager_user` จะไม่กระทบสิทธิ์ที่ `manager_user` มอบให้ผู้อื่น เว้นแต่ใช้คำสั่ง `REVOKE ... FROM` โดยตรง

#### 7. GRANT สิทธิ์เฉพาะคอลัมน์
สมมติต้องการให้ `report_user` อ่านเฉพาะ `first_name` และ `department_id` ใน `employees`:
```sql
GRANT SELECT (first_name, department_id) ON employees TO report_user;
```
- **คำอธิบาย**: `report_user` เห็นเฉพาะคอลัมน์ `first_name` และ `department_id`
- **ทดสอบ**:
  ```sql
  SELECT first_name, department_id FROM employees; -- สำเร็จ
  SELECT salary FROM employees; -- ล้มเหลว (ORA-00904: invalid identifier)
  ```

#### 8. GRANT สิทธิ์สำหรับ SEQUENCE
สมมติมี SEQUENCE:
```sql
CREATE SEQUENCE emp_seq START WITH 1 INCREMENT BY 1;
```
ให้สิทธิ์ `hr_user` ใช้ SEQUENCE:
```sql
GRANT SELECT ON emp_seq TO hr_user;
```
- **ทดสอบ**:
  ```sql
  INSERT INTO employees (employee_id, first_name, salary, department_id)
  VALUES (emp_seq.NEXTVAL, 'David', 58000, 10);
  ```

### การจัดการสิทธิ์และการตรวจสอบ
- **ดูสิทธิ์ที่มอบให้**:
    - Object Privileges:
      ```sql
      SELECT grantee, table_name, privilege
      FROM dba_tab_privs
      WHERE table_name IN ('EMPLOYEES', 'DEPARTMENTS');
      ```
    - System Privileges:
      ```sql
      SELECT grantee, privilege
      FROM dba_sys_privs
      WHERE grantee IN ('HR_USER', 'MANAGER_USER', 'REPORT_USER');
      ```
    - Role Privileges:
      ```sql
      SELECT grantee, granted_role
      FROM dba_role_privs
      WHERE grantee IN ('HR_USER', 'MANAGER_USER', 'REPORT_USER');
      ```
- **ดูบทบาทและสิทธิ์ในบทบาท**:
  ```sql
  SELECT role, privilege
  FROM role_sys_privs
  WHERE role = 'REPORT_ROLE';
  ```
- **ถอนสิทธิ์ทั้งหมดจากผู้ใช้**:
    - ไม่มีคำสั่งเดียวที่ถอนทุกสิทธิ์ ต้องใช้ `REVOKE` สำหรับแต่ละสิทธิ์หรือบทบาท
    - ตัวอย่าง:
      ```sql
      REVOKE ALL ON employees FROM hr_user;
      REVOKE CREATE SESSION, CREATE TABLE FROM hr_user;
      REVOKE RESOURCE FROM manager_user;
      ```

### ข้อควรระวัง
- **WITH GRANT OPTION**:
    - ผู้ใช้ที่ได้รับสิทธิ์ด้วย `WITH GRANT OPTION` สามารถมอบสิทธิ์ต่อได้ ซึ่งอาจนำไปสู่การจัดการที่ควบคุมยาก
    - การเพิกถอนสิทธิ์จากผู้ให้จะไม่ยกเลิกสิทธิ์ที่มอบต่อให้ผู้อื่นโดยอัตโนมัติ
- **WITH ADMIN OPTION**:
    - คล้าย `WITH GRANT OPTION` แต่ใช้กับ System Privileges และ Role
    - ผู้ใช้ที่มี `WITH ADMIN OPTION` สามารถมอบ Role หรือ System Privilege ให้ผู้อื่นได้
- **PUBLIC**:
    - การใช้ `GRANT ... TO PUBLIC` ทำให้ทุกคนในฐานข้อมูลมีสิทธิ์นั้น ควรใช้อย่างระมัดระวัง
    - ตัวอย่าง:
      ```sql
      GRANT SELECT ON departments TO PUBLIC;
      ```
- **การเพิกถอน Role**:
    - การเพิกถอน Role จะยกเลิกทุกสิทธิ์ที่ได้รับผ่าน Role นั้น
    - ตรวจสอบว่าไม่มีสิทธิ์อื่นที่ยังคงอนุญาตให้เข้าถึงวัตถุ
- **ประสิทธิภาพ**:
    - การจัดการสิทธิ์จำนวนมากอาจเพิ่มความซับซ้อนในการบริหาร
    - ใช้ Role เพื่อลดการกำหนดสิทธิ์ซ้ำซ้อน
- **ระบบฐานข้อมูล**:
    - **Oracle**: รองรับการควบคุมสิทธิ์อย่างละเอียดด้วย Object, System, และ Role Privileges
    - **MySQL**: คล้าย Oracle แต่คำสั่งอาจแตกต่าง เช่น `GRANT ALL ON database_name.* TO 'user'@'host';`
        - ตัวอย่างใน MySQL:
          ```sql
          GRANT SELECT, INSERT ON employees TO 'hr_user'@'localhost';
          REVOKE INSERT ON employees FROM 'hr_user'@'localhost';
          ```
    - **PostgreSQL**: รองรับการควบคุมสิทธิ์คล้าย Oracle แต่ใช้ `GRANT ... ON SCHEMA` สำหรับจัดการระดับ Schema
        - ตัวอย่าง:
          ```sql
          GRANT SELECT ON employees TO hr_user;
          GRANT USAGE ON SEQUENCE emp_seq TO hr_user;
          ```
    - **SQL Server**: ใช้ `GRANT`, `DENY`, และ `REVOKE` คล้าย Oracle แต่มี `DENY` สำหรับปฏิเสธสิทธิ์โดยชัดเจน
- **ความปลอดภัย**:
    - หลีกเลี่ยงการให้สิทธิ์ที่ไม่จำเป็น (Principle of Least Privilege)
    - ตรวจสอบสิทธิ์ของผู้ใช้เป็นประจำด้วยมุมมองระบบ (System Views)
    - ใช้ Role เพื่อจัดการกลุ่มผู้ใช้ที่มีความรับผิดชอบคล้ายกัน

### การเปรียบเทียบ GRANT และ REVOKE
| คุณสมบัติ              | GRANT                              | REVOKE                             |
|------------------------|------------------------------------|------------------------------------|
| **วัตถุประสงค์**      | มอบสิทธิ์ให้ผู้ใช้หรือบทบาท        | เพิกถอนสิทธิ์จากผู้ใช้หรือบทบาท    |
| **ประเภทสิทธิ์**       | Object, System, Role               | Object, System, Role               |
| **ตัวเลือก**           | WITH GRANT/ADMIN OPTION            | ไม่มีตัวเลือกพิเศษ                 |
| **ผลกระทบ**            | เพิ่มการเข้าถึง                    | ลดหรือยกเลิกการเข้าถึง            |
| **การใช้งานทั่วไป**    | เพิ่มผู้ใช้ใหม่, มอบสิทธิ์ชั่วคราว | ลบผู้ใช้, จำกัดการเข้าถึง          |

### การใช้งานใน PL/SQL
สามารถใช้ `GRANT` และ `REVOKE` ใน PL/SQL เพื่อจัดการสิทธิ์แบบอัตโนมัติ เช่น:
```sql
DECLARE
   v_user VARCHAR2(30) := 'report_user';
BEGIN
   -- มอบสิทธิ์
   EXECUTE IMMEDIATE 'GRANT SELECT ON employees TO ' || v_user;
   DBMS_OUTPUT.PUT_LINE('Granted SELECT on employees to ' || v_user);
   
   -- รอ 10 วินาที (จำลองการทำงาน)
   DBMS_LOCK.SLEEP(10);
   
   -- เพิกถอนสิทธิ์
   EXECUTE IMMEDIATE 'REVOKE SELECT ON employees FROM ' || v_user;
   DBMS_OUTPUT.PUT_LINE('Revoked SELECT on employees from ' || v_user);
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/
```
- **ผลลัพธ์**:
  ```
  Granted SELECT on employees to report_user
  Revoked SELECT on employees from report_user
  ```
- **คำอธิบาย**:
    - ใช้ `EXECUTE IMMEDIATE` เพื่อรันคำสั่ง `GRANT` และ `REVOKE` แบบไดนามิก
    - เหมาะสำหรับการจัดการสิทธิ์ในสคริปต์หรือกระบวนการอัตโนมัติ

### ตัวอย่างสถานการณ์จริง
สมมติต้องการจัดการสิทธิ์สำหรับทีม HR, ผู้จัดการ, และทีมรายงาน:

1. **สร้าง Role และมอบสิทธิ์**:
```sql
-- สร้าง Role
CREATE ROLE hr_role;
CREATE ROLE manager_role;
CREATE ROLE report_role;

-- มอบสิทธิ์ให้ Role
GRANT SELECT, INSERT, UPDATE ON employees TO hr_role;
GRANT SELECT, UPDATE (salary) ON employees TO manager_role;
GRANT SELECT ON employees, departments TO report_role;

-- มอบ Role ให้ผู้ใช้
GRANT hr_role TO hr_user;
GRANT manager_role TO manager_user;
GRANT report_role TO report_user;
```

2. **ทดสอบการใช้งาน**:
- **hr_user**:
  ```sql
  SELECT first_name, salary FROM employees; -- สำเร็จ
  UPDATE employees SET salary = 55000 WHERE employee_id = 1; -- สำเร็จ
  ```
- **manager_user**:
  ```sql
  SELECT first_name FROM employees; -- สำเร็จ
  UPDATE employees SET salary = 56000 WHERE employee_id = 2; -- สำเร็จ
  UPDATE employees SET first_name = 'Janet'; -- ล้มเหลว (ไม่มีสิทธิ์อัปเดต first_name)
  ```
- **report_user**:
  ```sql
  SELECT first_name, department_id FROM employees; -- สำเร็จ
  SELECT department_name FROM departments; -- สำเร็จ
  INSERT INTO employees VALUES (4, 'Bob', 65000, 20); -- ล้มเหลว (ไม่มีสิทธิ์ INSERT)
  ```

3. **เพิกถอนสิทธิ์เมื่อไม่ต้องการ**:
   สมมติ `report_user` ไม่ต้องการเข้าถึง `employees` อีกต่อไป:
```sql
REVOKE SELECT ON employees FROM report_role;
```
- **ทดสอบ**:
  ```sql
  SELECT first_name FROM employees; -- ล้มเหลว (ORA-00942: table or view does not exist)
  SELECT department_name FROM departments; -- ยังสำเร็จ
  ```

4. **จำกัดการเข้าถึงด้วย VIEW และ GRANT**:
   สมมติต้องการให้ `report_user` เห็นเฉพาะข้อมูลพนักงานในแผนก IT:
```sql
CREATE VIEW emp_it_view AS
    SELECT e.first_name, e.salary, d.department_name
    FROM employees e
    JOIN departments d
    ON e.department_id = d.department_id
    WHERE d.department_name = 'IT'
    WITH READ ONLY;

GRANT SELECT ON emp_it_view TO report_user;
```
- **ทดสอบ**:
  ```sql
  SELECT * FROM emp_it_view;
  ```
    - **ผลลัพธ์**:
      ```
      first_name | salary | department_name
      ------------+--------+----------------
      Jane       | 60000  | IT
      ```

### การตรวจสอบและรักษาความปลอดภัย
- **ตรวจสอบสิทธิ์ที่มอบให้**:
    - ใช้มุมมองระบบ:
      ```sql
      SELECT grantee, table_name, privilege
      FROM dba_tab_privs
      WHERE grantee IN ('HR_USER', 'MANAGER_USER', 'REPORT_USER');
      ```
- **ตรวจสอบ Role**:
  ```sql
  SELECT grantee, granted_role
  FROM dba_role_privs
  WHERE grantee IN ('HR_USER', 'MANAGER_USER', 'REPORT_USER');
  ```
- **แนวทางปฏิบัติด้านความปลอดภัย**:
    - ใช้ Role เพื่อจัดการกลุ่มผู้ใช้ที่มีความรับผิดชอบคล้ายกัน
    - หลีกเลี่ยงการให้สิทธิ์ `PUBLIC` เว้นแต่จำเป็น
    - ใช้ `WITH READ ONLY` ใน VIEW เพื่อป้องกันการแก้ไข
    - ตรวจสอบและเพิกถอนสิทธิ์ของผู้ใช้ที่ไม่ได้ใช้งาน
    - ใช้การเข้ารหัสรหัสผ่านและนโยบายรหัสผ่านที่เข้มงวด
---

## การควบคุมธุรกรรมด้วย `COMMIT`, `ROLLBACK`, `SAVEPOINT`
ใน SQL การควบคุมธุรกรรม (Transaction Control) เป็นกระบวนการจัดการการเปลี่ยนแปลงข้อมูลในฐานข้อมูลเพื่อให้มั่นใจใน**ความถูกต้อง**และ**ความสอดคล้อง**ของข้อมูล โดยใช้คำสั่ง **COMMIT**, **ROLLBACK**, และ **SAVEPOINT** คำสั่งเหล่านี้ช่วยควบคุมการยืนยัน (Commit) หรือยกเลิก (Rollback) การเปลี่ยนแปลง รวมถึงการกำหนดจุดย้อนกลับ (Savepoint) ในระหว่างธุรกรรม ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### คำอธิบายของธุรกรรม (Transaction)
- **ธุรกรรม**คือชุดของคำสั่ง SQL (เช่น `INSERT`, `UPDATE`, `DELETE`) ที่ทำงานเป็นหน่วยเดียว เพื่อให้มั่นใจในคุณสมบัติ **ACID**:
    - **Atomicity**: ทุกคำสั่งในธุรกรรมสำเร็จทั้งหมด หรือยกเลิกทั้งหมด
    - **Consistency**: ฐานข้อมูลอยู่ในสถานะที่ถูกต้องก่อนและหลังธุรกรรม
    - **Isolation**: ธุรกรรมหนึ่งไม่รบกวนธุรกรรมอื่น
    - **Durability**: การเปลี่ยนแปลงที่ยืนยันแล้วจะถูกบันทึกถาวร
- **คำสั่งควบคุมธุรกรรม**:
    - **COMMIT**: ยืนยันการเปลี่ยนแปลงทั้งหมดในธุรกรรม ทำให้ถาวร
    - **ROLLBACK**: ยกเลิกการเปลี่ยนแปลงทั้งหมดในธุรกรรม กลับสู่สถานะก่อนหน้า
    - **SAVEPOINT**: กำหนดจุดย้อนกลับภายในธุรกรรม เพื่อให้สามารถย้อนกลับบางส่วนได้

### ไวยากรณ์พื้นฐาน
```sql
-- เริ่มธุรกรรม (โดยปริยายเมื่อรันคำสั่ง DML แรก)
INSERT INTO table_name ...;
UPDATE table_name ...;

-- กำหนดจุดย้อนกลับ
SAVEPOINT savepoint_name;

-- ยืนยันธุรกรรม
COMMIT;

-- ยกเลิกธุรกรรมทั้งหมด
ROLLBACK;

-- ยกเลิกบางส่วนไปยังจุดย้อนกลับ
ROLLBACK TO SAVEPOINT savepoint_name;
```

- **ธุรกรรมเริ่มต้น**เมื่อรันคำสั่ง DML (`INSERT`, `UPDATE`, `DELETE`) หรือ DDL ในบางกรณี
- **ธุรกรรมสิ้นสุด**เมื่อ:
    - รัน `COMMIT` (ยืนยันการเปลี่ยนแปลง)
    - รัน `ROLLBACK` (ยกเลิกการเปลี่ยนแปลง)
    - เกิดคำสั่ง DDL (เช่น `CREATE TABLE`) ซึ่งมักทำ Auto-Commit ใน Oracle
    - เซสชันสิ้นสุด (เช่น ออกจาก SQL*Plus) ซึ่งอาจทำ Auto-Commit หรือ Rollback ขึ้นอยู่กับการตั้งค่า

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
COMMIT;
```

#### 1. ใช้ COMMIT เพื่อยืนยันธุรกรรม
```sql
-- เริ่มธุรกรรม
INSERT INTO employees (employee_id, first_name, salary, department_id)
VALUES (4, 'Bob', 65000, 20);

UPDATE employees
SET salary = salary * 1.1
WHERE department_id = 10;

-- ตรวจสอบข้อมูล (ยังไม่ยืนยัน)
SELECT * FROM employees;
```
- **ผลลัพธ์** (ในเซสชันปัจจุบัน):
  ```
  employee_id | first_name | salary | department_id
  ------------+------------+--------+----------------
  1          | John       | 55000  | 10
  2          | Jane       | 60000  | 20
  3          | Alice      | 60500  | 10
  4          | Bob        | 65000  | 20
  ```
- **ยืนยันธุรกรรม**:
  ```sql
  COMMIT;
  ```
- **คำอธิบาย**:
    - การเปลี่ยนแปลง (แทรก Bob และเพิ่มเงินเดือนแผนก 10) กลายเป็นถาวร
    - เซสชันอื่นจะเห็นข้อมูลที่อัปเดตแล้ว
- **ตรวจสอบหลัง COMMIT**:
  ```sql
  SELECT * FROM employees;
  ```
    - **ผลลัพธ์**: เดียวกับด้านบน (ข้อมูลถาวร)

#### 2. ใช้ ROLLBACK เพื่อยกเลิกธุรกรรม
```sql
-- เริ่มธุรกรรม
INSERT INTO employees (employee_id, first_name, salary, department_id)
VALUES (5, 'Carol', 52000, 30);

UPDATE employees
SET salary = 70000
WHERE employee_id = 2;

-- ตรวจสอบข้อมูล (ยังไม่ยืนยัน)
SELECT * FROM employees WHERE employee_id IN (2, 5);
```
- **ผลลัพธ์**:
  ```
  employee_id | first_name | salary | department_id
  ------------+------------+--------+----------------
  2          | Jane       | 70000  | 20
  5          | Carol      | 52000  | 30
  ```
- **ยกเลิกธุรกรรม**:
  ```sql
  ROLLBACK;
  ```
- **ตรวจสอบหลัง ROLLBACK**:
  ```sql
  SELECT * FROM employees WHERE employee_id IN (2, 5);
  ```
- **ผลลัพธ์**:
  ```
  employee_id | first_name | salary | department_id
  ------------+------------+--------+----------------
  2          | Jane       | 60000  | 20
  ```
- **คำอธิบาย**:
    - การแทรก Carol และการอัปเดตเงินเดือนของ Jane ถูกยกเลิก
    - ข้อมูลกลับสู่สถานะก่อนเริ่มธุรกรรม (ตามที่ COMMIT ล่าสุด)

#### 3. ใช้ SAVEPOINT เพื่อย้อนกลับบางส่วน
```sql
-- เริ่มธุรกรรม
INSERT INTO employees (employee_id, first_name, salary, department_id)
VALUES (5, 'Carol', 52000, 30);

-- กำหนดจุดย้อนกลับ
SAVEPOINT after_insert;

UPDATE employees
SET salary = 70000
WHERE employee_id = 2;

-- ตรวจสอบข้อมูล
SELECT * FROM employees WHERE employee_id IN (2, 5);
```
- **ผลลัพธ์**:
  ```
  employee_id | first_name | salary | department_id
  ------------+------------+--------+----------------
  2          | Jane       | 70000  | 20
  5          | Carol      | 52000  | 30
  ```
- **ย้อนกลับไปยัง SAVEPOINT**:
  ```sql
  ROLLBACK TO SAVEPOINT after_insert;
  ```
- **ตรวจสอบหลัง ROLLBACK TO SAVEPOINT**:
  ```sql
  SELECT * FROM employees WHERE employee_id IN (2, 5);
  ```
- **ผลลัพธ์**:
  ```
  employee_id | first_name | salary | department_id
  ------------+------------+--------+----------------
  2          | Jane       | 60000  | 20
  5          | Carol      | 52000  | 30
  ```
- **ยืนยันธุรกรรม**:
  ```sql
  COMMIT;
  ```
- **คำอธิบาย**:
    - `SAVEPOINT after_insert` เก็บสถานะหลังแทรก Carol
    - `ROLLBACK TO SAVEPOINT after_insert` ยกเลิกการอัปเดตเงินเดือนของ Jane แต่คงการแทรก Carol
    - `COMMIT` ยืนยันการแทรก Carol ทำให้ถาวร

#### 4. การใช้ COMMIT และ ROLLBACK ใน PL/SQL
```sql
DECLARE
   v_error_code NUMBER;
   v_error_msg VARCHAR2(200);
BEGIN
   -- เริ่มธุรกรรม
   INSERT INTO employees (employee_id, first_name, salary, department_id)
   VALUES (6, 'David', 58000, 10);
   
   SAVEPOINT after_insert_david;
   
   UPDATE employees
   SET salary = 75000
   WHERE employee_id = 1;
   
   -- จำลองข้อผิดพลาด
   INSERT INTO employees (employee_id, first_name, salary, department_id)
   VALUES (6, 'Duplicate', 50000, 20); -- ละเมิด PRIMARY KEY
   
EXCEPTION
   WHEN DUP_VAL_ON_INDEX THEN
      v_error_code := SQLCODE;
      v_error_msg := SQLERRM;
      DBMS_OUTPUT.PUT_LINE('Error: ' || v_error_code || ' - ' || v_error_msg);
      
      -- ย้อนกลับไปยัง SAVEPOINT
      ROLLBACK TO SAVEPOINT after_insert_david;
      
      -- ยังคงยืนยันการเปลี่ยนแปลงที่ต้องการ
      COMMIT;
      DBMS_OUTPUT.PUT_LINE('Inserted David, rolled back other changes');
   WHEN OTHERS THEN
      ROLLBACK;
      DBMS_OUTPUT.PUT_LINE('Unexpected error: ' || SQLERRM);
END;
/
```
- **ผลลัพธ์**:
  ```
  Error: -1 - ORA-00001: unique constraint (SYS_C007001) violated
  Inserted David, rolled back other changes
  ```
- **ตรวจสอบข้อมูล**:
  ```sql
  SELECT * FROM employees WHERE employee_id = 6;
  ```
    - **ผลลัพธ์**:
      ```
      employee_id | first_name | salary | department_id
      ------------+------------+--------+----------------
      6          | David      | 58000  | 10
      ```
- **คำอธิบาย**:
    - การแทรก David สำเร็จและกำหนด `SAVEPOINT`
    - การอัปเดตเงินเดือนของ John สำเร็จชั่วคราว
    - การแทรกข้อมูลซ้ำทำให้เกิดข้อผิดพลาด (`DUP_VAL_ON_INDEX`)
    - `ROLLBACK TO SAVEPOINT` ยกเลิกการอัปเดตและการแทรกที่ผิดพลาด แต่คงการแทรก David
    - `COMMIT` ยืนยันการแทรก David

#### 5. การใช้ SAVEPOINT ในธุรกรรมที่ซับซ้อน
สมมติต้องการปรับปรุงข้อมูลในหลายตาราง:
```sql
-- เริ่มธุรกรรม
INSERT INTO departments (department_id, department_name)
VALUES (30, 'HR');

SAVEPOINT after_dept_insert;

INSERT INTO employees (employee_id, first_name, salary, department_id)
VALUES (7, 'Eve', 54000, 30);

SAVEPOINT after_emp_insert;

UPDATE employees
SET salary = salary * 1.05
WHERE department_id = 30;

-- ตรวจสอบข้อมูล
SELECT * FROM employees WHERE department_id = 30;
```
- **ผลลัพธ์**:
  ```
  employee_id | first_name | salary | department_id
  ------------+------------+--------+----------------
  7          | Eve        | 56700  | 30
  ```
- **ย้อนกลับบางส่วน**:
  ```sql
  ROLLBACK TO SAVEPOINT after_emp_insert;
  ```
- **ตรวจสอบ**:
  ```sql
  SELECT * FROM employees WHERE department_id = 30;
  ```
    - **ผลลัพธ์**:
      ```
      employee_id | first_name | salary | department_id
      ------------+------------+--------+----------------
      7          | Eve        | 54000  | 30
      ```
- **ย้อนกลับทั้งหมด**:
  ```sql
  ROLLBACK;
  ```
- **ตรวจสอบ**:
  ```sql
  SELECT * FROM departments WHERE department_id = 30;
  SELECT * FROM employees WHERE department_id = 30;
  ```
    - **ผลลัพธ์**: ไม่มีแถว (ทั้งสองตารางกลับสู่สถานะก่อนธุรกรรม)

### ข้อควรระวัง
- **Auto-Commit**:
    - ใน Oracle คำสั่ง DDL (เช่น `CREATE TABLE`, `ALTER TABLE`) จะทำ Auto-Commit ทำให้ธุรกรรมปัจจุบันถูกยืนยันโดยอัตโนมัติ
    - ตัวอย่าง:
      ```sql
      INSERT INTO employees VALUES (8, 'Frank', 59000, 10);
      CREATE TABLE temp_table (id NUMBER); -- Auto-Commit
      ROLLBACK; -- ไม่สามารถย้อนการแทรก Frank ได้
      ```
- **SAVEPOINT**:
    - SAVEPOINT มีขอบเขตภายในธุรกรรมเดียวและถูกล้างเมื่อ `COMMIT` หรือ `ROLLBACK` ทั้งหมด
    - ไม่สามารถย้อนกลับไปยัง SAVEPOINT ที่มาก่อนการ `COMMIT`
    - การใช้ `ROLLBACK TO SAVEPOINT` จะยกเลิกการเปลี่ยนแปลงหลัง SAVEPOINT เท่านั้น
- **CURRVAL และ SEQUENCE**:
    - การเรียก `NEXTVAL` จาก SEQUENCE ไม่ถูกยกเลิกโดย `ROLLBACK`
    - ตัวอย่าง:
      ```sql
      CREATE SEQUENCE emp_seq START WITH 100 INCREMENT BY 1;
      INSERT INTO employees VALUES (emp_seq.NEXTVAL, 'Grace', 57000, 20); -- emp_seq = 100
      ROLLBACK;
      INSERT INTO employees VALUES (emp_seq.NEXTVAL, 'Grace', 57000, 20); -- emp_seq = 101
      ```
    - **คำอธิบาย**: `NEXTVAL` ทำให้ SEQUENCE เพิ่มขึ้น แม้จะ `ROLLBACK` การแทรก
- **ประสิทธิภาพ**:
    - ธุรกรรมที่ยาวนาน (มีคำสั่ง DML จำนวนมาก) อาจใช้ทรัพยากรเยอะ เช่น Undo Tablespace
    - ควร `COMMIT` หรือ `ROLLBACK` เป็นระยะเพื่อปล่อยทรัพยากร
- **Isolation Level**:
    - Oracle ใช้ **Read Committed** เป็น Isolation Level เริ่มต้น ทำให้เซสชันอื่นไม่เห็นการเปลี่ยนแปลงจนกว่าจะ `COMMIT`
    - สามารถตั้งค่า Isolation Level อื่น เช่น `SERIALIZABLE`:
      ```sql
      SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
      ```
- **ระบบฐานข้อมูล**:
    - **Oracle**: รองรับ `COMMIT`, `ROLLBACK`, `SAVEPOINT` อย่างสมบูรณ์
    - **MySQL**: รองรับใน Storage Engine เช่น InnoDB แต่ MyISAM ไม่รองรับธุรกรรม
        - ตัวอย่างใน MySQL:
          ```sql
          START TRANSACTION;
          INSERT INTO employees VALUES (8, 'Frank', 59000, 10);
          SAVEPOINT sp1;
          UPDATE employees SET salary = 60000 WHERE employee_id = 8;
          ROLLBACK TO sp1;
          COMMIT;
          ```
    - **PostgreSQL**: คล้าย Oracle แต่ใช้ `BEGIN` เพื่อเริ่มธุรกรรมชัดเจน
        - ตัวอย่าง:
          ```sql
          BEGIN;
          INSERT INTO employees VALUES (8, 'Frank', 59000, 10);
          SAVEPOINT sp1;
          UPDATE employees SET salary = 60000;
          ROLLBACK TO sp1;
          COMMIT;
          ```
    - **SQL Server**: คล้าย Oracle แต่ใช้ `BEGIN TRANSACTION` และรองรับ `SAVE TRANSACTION` แทน `SAVEPOINT`
        - ตัวอย่าง:
          ```sql
          BEGIN TRANSACTION;
          INSERT INTO employees VALUES (8, 'Frank', 59000, 10);
          SAVE TRANSACTION sp1;
          UPDATE employees SET salary = 60000;
          ROLLBACK TRANSACTION sp1;
          COMMIT;
          ```
- **การล็อก (Locking)**:
    - คำสั่ง DML จะล็อกแถวที่เกี่ยวข้องจนกว่าจะ `COMMIT` หรือ `ROLLBACK`
    - อาจทำให้เกิด Deadlock ถ้ามีหลายธุรกรรมแข่งกัน
    - ตัวอย่าง:
      ```sql
      -- เซสชัน 1
      UPDATE employees SET salary = 58000 WHERE employee_id = 1;
      -- ยังไม่ COMMIT
  
      -- เซสชัน 2
      UPDATE employees SET salary = 59000 WHERE employee_id = 1; -- รอจนเซสชัน 1 COMMIT หรือ ROLLBACK
      ```

### การเปรียบเทียบ COMMIT, ROLLBACK, SAVEPOINT
| คุณสมบัติ              | COMMIT                            | ROLLBACK                          | SAVEPOINT                         |
|------------------------|-----------------------------------|-----------------------------------|-----------------------------------|
| **วัตถุประสงค์**      | ยืนยันการเปลี่ยนแปลงถาวร          | ยกเลิกการเปลี่ยนแปลงทั้งหมด        | กำหนดจุดย้อนกลับในธุรกรรม        |
| **ผลกระทบ**            | ทำให้ข้อมูลถาวรในฐานข้อมูล        | กลับสู่สถานะก่อนธุรกรรม           | อนุญาตให้ย้อนกลับบางส่วน         |
| **ขอบเขต**            | สิ้นสุดธุรกรรม                    | สิ้นสุดธุรกรรม                    | ภายในธุรกรรมเดียว                |
| **การใช้งานทั่วไป**    | เมื่อมั่นใจว่าการเปลี่ยนแปลงถูกต้อง| เมื่อเกิดข้อผิดพลาดหรือยกเลิก     | จัดการธุรกรรมที่ซับซ้อน           |

### การใช้งานใน PL/SQL
สามารถใช้ `COMMIT`, `ROLLBACK`, และ `SAVEPOINT` ใน PL/SQL เพื่อจัดการธุรกรรมที่ซับซ้อน เช่น:
```sql
CREATE OR REPLACE PROCEDURE update_salary(p_dept_id NUMBER, p_increase_pct NUMBER)
IS
BEGIN
   -- เริ่มธุรกรรม
   SAVEPOINT before_update;
   
   -- อัปเดตเงินเดือน
   UPDATE employees
   SET salary = salary * (1 + p_increase_pct / 100)
   WHERE department_id = p_dept_id;
   
   -- ตรวจสอบจำนวนแถวที่อัปเดต
   IF SQL%ROWCOUNT = 0 THEN
      RAISE_APPLICATION_ERROR(-20001, 'No employees found in department ' || p_dept_id);
   END IF;
   
   -- จำลองการตรวจสอบเพิ่มเติม
   IF p_increase_pct > 50 THEN
      ROLLBACK TO SAVEPOINT before_update;
      RAISE_APPLICATION_ERROR(-20002, 'Salary increase exceeds 50%');
   END IF;
   
   -- ยืนยันธุรกรรม
   COMMIT;
   DBMS_OUTPUT.PUT_LINE('Salary updated for department ' || p_dept_id);
EXCEPTION
   WHEN OTHERS THEN
      ROLLBACK;
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
      RAISE;
END;
/
```
- **ทดสอบ**:
  ```sql
  EXEC update_salary(10, 10); -- สำเร็จ
  EXEC update_salary(30, 10); -- ล้มเหลว (ไม่มีพนักงานในแผนก 30)
  EXEC update_salary(10, 60); -- ล้มเหลว (เพิ่มเกิน 50%)
  ```
- **ผลลัพธ์** (ตัวอย่าง):
  ```
  Salary updated for department 10
  Error: ORA-20001: No employees found in department 30
  Error: ORA-20002: Salary increase exceeds 50%
  ```
- **ตรวจสอบข้อมูล**:
  ```sql
  SELECT * FROM employees WHERE department_id = 10;
  ```
    - **ผลลัพธ์**:
      ```
      employee_id | first_name | salary | department_id
      ------------+------------+--------+----------------
      1          | John       | 55000  | 10
      3          | Alice      | 60500  | 10
      ```
- **คำอธิบาย**:
    - การอัปเดตแผนก 10 (เพิ่ม 10%) สำเร็จและยืนยัน
    - การอัปเดตแผนก 30 ล้มเหลว (ไม่มีพนักงาน) และ ROLLBACK
    - การอัปเดตแผนก 10 (เพิ่ม 60%) ล้มเหลว (เกิน 50%) และย้อนกลับไปยัง SAVEPOINT

### ตัวอย่างสถานการณ์จริง
สมมติต้องการจัดการการโอนย้ายพนักงานระหว่างแผนกและปรับเงินเดือนในธุรกรรมเดียว:

1. **เริ่มธุรกรรม**:
```sql
-- เพิ่มแผนกใหม่
INSERT INTO departments (department_id, department_name)
VALUES (30, 'HR');

SAVEPOINT after_dept_insert;

-- โอนย้ายพนักงาน
UPDATE employees
SET department_id = 30
WHERE employee_id = 2;

SAVEPOINT after_transfer;

-- ปรับเงินเดือน
UPDATE employees
SET salary = salary + 5000
WHERE department_id = 30;

-- ตรวจสอบข้อมูล
SELECT e.first_name, e.salary, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.department_id = 30;
```
- **ผลลัพธ์**:
  ```
  first_name | salary | department_name
  ------------+--------+----------------
  Jane       | 65000  | HR
  ```

2. **จำลองข้อผิดพลาด**:
```sql
-- พยายามแทรกแผนกที่ซ้ำ
INSERT INTO departments (department_id, department_name)
VALUES (30, 'Duplicate HR'); -- ละเมิด PRIMARY KEY
```
- **จัดการข้อผิดพลาดใน PL/SQL**:
```sql
DECLARE
   v_error_code NUMBER;
   v_error_msg VARCHAR2(200);
BEGIN
   INSERT INTO departments (department_id, department_name) VALUES (30, 'HR');
   SAVEPOINT after_dept_insert;
   UPDATE employees SET department_id = 30 WHERE employee_id = 2;
   SAVEPOINT after_transfer;
   UPDATE employees SET salary = salary + 5000 WHERE department_id = 30;
   
   -- จำลองข้อผิดพลาด
   INSERT INTO departments (department_id, department_name) VALUES (30, 'Duplicate HR');
EXCEPTION
   WHEN DUP_VAL_ON_INDEX THEN
      v_error_code := SQLCODE;
      v_error_msg := SQLERRM;
      DBMS_OUTPUT.PUT_LINE('Error: ' || v_error_code || ' - ' || v_error_msg);
      ROLLBACK TO SAVEPOINT after_transfer;
      COMMIT; -- ยืนยันการโอนย้ายและปรับเงินเดือน
      DBMS_OUTPUT.PUT_LINE('Transfer and salary update completed');
   WHEN OTHERS THEN
      ROLLBACK;
      DBMS_OUTPUT.PUT_LINE('Unexpected error: ' || SQLERRM);
END;
/
```
- **ผลลัพธ์**:
  ```
  Error: -1 - ORA-00001: unique constraint (SYS_C007002) violated
  Transfer and salary update completed
  ```
- **ตรวจสอบ**:
  ```sql
  SELECT e.first_name, e.salary, d.department_name
  FROM employees e
  JOIN departments d ON e.department_id = d.department_id
  WHERE e.department_id = 30;
  ```
    - **ผลลัพธ์**:
      ```
      first_name | salary | department_name
      ------------+--------+----------------
      Jane       | 65000  | HR
      ```
---

## ความเข้าใจเกี่ยวกับ `ACID` properties
คุณสมบัติ **ACID** เป็นหลักการพื้นฐานในการจัดการธุรกรรม (Transaction) ในระบบฐานข้อมูล เพื่อให้มั่นใจว่า**การทำงานของธุรกรรมมีความน่าเชื่อถือ**, **ถูกต้อง**, และ**ปลอดภัย** แม้ในสภาวะที่มีการทำงานพร้อมกัน (Concurrency) หรือเกิดข้อผิดพลาด คุณสมบัติ ACID ประกอบด้วย **Atomicity**, **Consistency**, **Isolation**, และ **Durability** ซึ่งแต่ละตัวมีบทบาทสำคัญในการรักษาความสมบูรณ์ของข้อมูล ต่อไปนี้คือคำอธิบายโดยละเอียดของแต่ละคุณสมบัติ พร้อมตัวอย่างในบริบทของ Oracle SQL และระบบฐานข้อมูลทั่วไป

### คุณสมบัติ ACID
1. **Atomicity (อะตอมมิซิตี้)**:
    - **คำอธิบาย**: รับประกันว่าธุรกรรมจะ**สำเร็จทั้งหมดหรือล้มเหลวทั้งหมด** คำสั่งทั้งหมดในธุรกรรมจะถูกดำเนินการเป็นหน่วยเดียว (Atomic Unit) ถ้ามีคำสั่งใดล้มเหลว ธุรกรรมทั้งหมดจะถูกยกเลิก (Rollback) และฐานข้อมูลจะกลับสู่สถานะเดิม
    - **กลไก**: ใช้คำสั่ง `COMMIT` และ `ROLLBACK` เพื่อควบคุม
    - **ตัวอย่าง**:
      สมมติต้องการโอนเงิน 1,000 จากบัญชี A ไปบัญชี B:
      ```sql
      CREATE TABLE accounts (
          account_id NUMBER PRIMARY KEY,
          account_name VARCHAR2(50),
          balance NUMBER(10,2)
      );
      INSERT INTO accounts VALUES (1, 'Alice', 5000);
      INSERT INTO accounts VALUES (2, 'Bob', 3000);
      COMMIT;
 
      -- เริ่มธุรกรรม
      UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;
      UPDATE accounts SET balance = balance + 1000 WHERE account_id = 2;
      COMMIT;
      ```
        - ถ้าทั้งสอง `UPDATE` สำเร็จ จะ `COMMIT` ทำให้ยอดเงินในบัญชี A ลดลงและบัญชี B เพิ่มขึ้น
        - ถ้ามีข้อผิดพลาด (เช่น การอัปเดตบัญชี B ล้มเหลว):
          ```sql
          UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;
          -- จำลองข้อผิดพลาด
          UPDATE accounts SET balance = balance + 1000 WHERE account_id = 999; -- ไม่มี account_id 999
          ```
            - ฐานข้อมูลจะ `ROLLBACK` การเปลี่ยนแปลงทั้งหมด (ยอดเงินในบัญชี A จะไม่เปลี่ยน)
        - **ผลลัพธ์หลัง ROLLBACK**:
          ```sql
          SELECT * FROM accounts;
          ```
          ```
          account_id | account_name | balance
          -----------+--------------+---------
          1         | Alice        | 5000
          2         | Bob          | 3000
          ```

2. **Consistency (ความสอดคล้อง)**:
    - **คำอธิบาย**: รับประกันว่าธุรกรรมจะนำฐานข้อมูลจาก**สถานะที่ถูกต้อง**ไปสู่**สถานะที่ถูกต้อง**อีกสถานะหนึ่ง โดยเคารพกฎของฐานข้อมูล เช่น Constraints (Primary Key, Foreign Key, Unique), Triggers, และ Data Integrity
    - **กลไก**: ฐานข้อมูลตรวจสอบ Constraints และกฎอื่น ๆ ก่อนยืนยันธุรกรรม
    - **ตัวอย่าง**:
      สมมติมีตาราง `employees` ที่มี Foreign Key Constraint:
      ```sql
      CREATE TABLE departments (
          department_id NUMBER PRIMARY KEY,
          department_name VARCHAR2(50)
      );
      CREATE TABLE employees (
          employee_id NUMBER PRIMARY KEY,
          first_name VARCHAR2(50),
          department_id NUMBER,
          CONSTRAINT fk_dept FOREIGN KEY (department_id) REFERENCES departments(department_id)
      );
      INSERT INTO departments VALUES (10, 'Sales');
      COMMIT;
      ```
        - ลองแทรกพนักงานในแผนกที่ไม่มีอยู่:
          ```sql
          INSERT INTO employees VALUES (1, 'John', 20); -- department_id 20 ไม่มี
          ```
            - **ผลลัพธ์**: ข้อผิดพลาด `ORA-02291: integrity constraint (FK_DEPT) violated - parent key not found`
            - ฐานข้อมูลป้องกันการแทรก เพราะจะละเมิด Foreign Key Constraint
        - แทรกข้อมูลที่ถูกต้อง:
          ```sql
          INSERT INTO employees VALUES (1, 'John', 10);
          COMMIT;
          ```
            - **ผลลัพธ์**: สำเร็จ เพราะ `department_id = 10` มีอยู่ใน `departments`
        - **คำอธิบาย**: Consistency รับประกันว่า Foreign Key Constraint ยังคงสมบูรณ์หลังธุรกรรม

3. **Isolation (การแยก)**:
    - **คำอธิบาย**: รับประกันว่าธุรกรรมแต่ละรายการทำงาน**แยกจากกัน** แม้จะมีหลายธุรกรรมรันพร้อมกัน (Concurrency) เพื่อป้องกันปัญหา เช่น Dirty Reads, Non-Repeatable Reads, หรือ Phantom Reads
    - **กลไก**: ใช้ **Locking** และ **Isolation Levels** เช่น Read Committed หรือ Serializable
    - **ตัวอย่าง**:
      สมมติมีสองเซสชันทำงานพร้อมกัน:
        - **เซสชัน 1**:
          ```sql
          UPDATE employees SET department_id = 20 WHERE employee_id = 1;
          -- ยังไม่ COMMIT
          ```
        - **เซสชัน 2**:
          ```sql
          SELECT department_id FROM employees WHERE employee_id = 1;
          ```
            - **ผลลัพธ์**: `department_id = 10` (ค่าเดิม) เพราะ Oracle ใช้ **Read Committed** Isolation Level ซึ่งป้องกัน Dirty Reads (การอ่านข้อมูลที่ยังไม่ยืนยัน)
        - **เซสชัน 1**:
          ```sql
          COMMIT;
          ```
        - **เซสชัน 2** (รัน SELECT อีกครั้ง):
          ```sql
          SELECT department_id FROM employees WHERE employee_id = 1;
          ```
            - **ผลลัพธ์**: `department_id = 20` (เห็นข้อมูลหลัง COMMIT)
        - **ตัวอย่าง Isolation Level อื่น (Serializable)**:
          ```sql
          SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
          SELECT SUM(salary) FROM employees;
          -- ธุรกรรมอื่นไม่สามารถแทรกหรืออัปเดต employees จนกว่าธุรกรรมนี้จะสิ้นสุด
          COMMIT;
          ```
            - **คำอธิบาย**: Serializable ป้องกัน Phantom Reads และ Non-Repeatable Reads โดยจำลองว่าธุรกรรมรันแบบอนุกรม (Serial)

4. **Durability (ความคงทน)**:
    - **คำอธิบาย**: รับประกันว่าการเปลี่ยนแปลงที่ยืนยันด้วย `COMMIT` จะถูก**บันทึกถาวร**ในฐานข้อมูล แม้จะเกิดปัญหา เช่น ไฟดับหรือระบบล่ม
    - **กลไก**: ใช้ **Write-Ahead Logging** (WAL) และการบันทึกลงดิสก์เพื่อกู้คืนข้อมูล
    - **ตัวอย่าง**:
      ```sql
      INSERT INTO employees VALUES (2, 'Jane', 20);
      COMMIT;
      ```
        - หลัง `COMMIT` ข้อมูลของ Jane ถูกบันทึกลงดิสก์
        - สมมติระบบล่มทันทีหลัง `COMMIT`:
            - เมื่อระบบกลับมา ฐานข้อมูลจะใช้ **Redo Log** เพื่อกู้คืนข้อมูล
            - **ตรวจสอบ**:
              ```sql
              SELECT * FROM employees WHERE employee_id = 2;
              ```
                - **ผลลัพธ์**:
                  ```
                  employee_id | first_name | department_id
                  ------------+------------+---------------
                  2          | Jane       | 20
                  ```
        - **คำอธิบาย**: Durability รับประกันว่า Jane ยังอยู่ในฐานข้อมูลหลังกู้คืน

### การประยุกต์ใช้ ACID ใน Oracle SQL
สมมติมีสถานการณ์โอนเงินระหว่างบัญชีเพื่อแสดงการทำงานของ ACID:
```sql
-- สร้างตาราง
CREATE TABLE accounts (
    account_id NUMBER PRIMARY KEY,
    account_name VARCHAR2(50),
    balance NUMBER(10,2),
    CONSTRAINT chk_balance CHECK (balance >= 0)
);
INSERT INTO accounts VALUES (1, 'Alice', 5000);
INSERT INTO accounts VALUES (2, 'Bob', 3000);
COMMIT;
```

#### ตัวอย่างธุรกรรม
```sql
BEGIN
   -- เริ่มธุรกรรม
   SAVEPOINT before_transfer;
   
   -- ลดยอดเงินจากบัญชี Alice
   UPDATE accounts
   SET balance = balance - 1000
   WHERE account_id = 1;
   
   -- เพิ่มยอดเงินให้บัญชี Bob
   UPDATE accounts
   SET balance = balance + 1000
   WHERE account_id = 2;
   
   -- จำลองข้อผิดพลาด: พยายามทำให้ยอดติดลบ
   UPDATE accounts
   SET balance = balance - 6000
   WHERE account_id = 1; -- ละเมิด CHECK CONSTRAINT (balance >= 0)
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
      ROLLBACK TO SAVEPOINT before_transfer;
      -- ไม่ COMMIT เพราะข้อผิดพลาด
END;
/
```
- **ผลลัพธ์**:
  ```
  Error: ORA-02290: check constraint (CHK_BALANCE) violated
  ```
- **ตรวจสอบ**:
  ```sql
  SELECT * FROM accounts;
  ```
    - **ผลลัพธ์**:
      ```
      account_id | account_name | balance
      -----------+--------------+---------
      1         | Alice        | 5000
      2         | Bob          | 3000
      ```

#### การเชื่อมโยงกับ ACID
1. **Atomicity**:
    - การโอนเงิน (ลดยอด Alice, เพิ่มยอด Bob) ถูกยกเลิกทั้งหมดเมื่อเกิดข้อผิดพลาด (ROLLBACK)
    - ไม่มีการเปลี่ยนแปลงบางส่วน (เช่น ลดยอด Alice แต่ไม่เพิ่มยอด Bob)
2. **Consistency**:
    - Constraint `balance >= 0` ป้องกันยอดติดลบ ทำให้ฐานข้อมูลอยู่ในสถานะที่ถูกต้อง
    - Foreign Key, Primary Key, และ Constraints อื่น ๆ ยังคงสมบูรณ์
3. **Isolation**:
    - เซสชันอื่นไม่เห็นยอดเงินที่เปลี่ยนแปลงจนกว่าจะ `COMMIT`
    - Oracle ใช้ Multi-Version Consistency Control (MVCC) เพื่อให้มั่นใจว่าเซสชันอื่นอ่านข้อมูลที่สอดคล้องกัน
4. **Durability**:
    - หาก `COMMIT` สำเร็จ (เช่น โอนเงินสำเร็จ) ข้อมูลจะถูกบันทึกลงดิสก์
    - Redo Log รับประกันว่าข้อมูลจะไม่สูญหายแม้ระบบล่ม

### ข้อควรระวังและการพิจารณา
- **Atomicity**:
    - ต้องออกแบบธุรกรรมให้ครอบคลุมทุกการเปลี่ยนแปลงที่เกี่ยวข้อง
    - การลืม `COMMIT` หรือ `ROLLBACK` อาจทำให้เกิดการล็อกหรือทรัพยากรค้าง
- **Consistency**:
    - Constraints และ Triggers ต้องได้รับการออกแบบอย่างรอบคอบเพื่อป้องกันสถานะที่ไม่ถูกต้อง
    - การละเมิด Constraint จะทำให้ธุรกรรมล้มเหลวและ ROLLBACK
- **Isolation**:
    - ระดับ Isolation ที่เข้มงวด (เช่น Serializable) อาจลดประสิทธิภาพในระบบที่มี Concurrency สูง
    - Oracle ใช้ **Read Committed** เป็นค่าเริ่มต้น ซึ่งอาจนำไปสู่ Non-Repeatable Reads หรือ Phantom Reads ในบางกรณี
    - การตั้งค่า Isolation Level:
      ```sql
      SET TRANSACTION ISOLATION LEVEL READ COMMITTED; -- ค่าเริ่มต้น
      SET TRANSACTION ISOLATION LEVEL SERIALIZABLE; -- เข้มงวดกว่า
      ```
- **Durability**:
    - ต้องมีการสำรองข้อมูล (Backup) และ Redo Log ที่เพียงพอเพื่อกู้คืนข้อมูล
    - ในบางระบบ (เช่น MySQL กับ InnoDB) การตั้งค่า `innodb_flush_log_at_trx_commit` อาจส่งผลต่อ Durability
- **ระบบฐานข้อมูล**:
    - **Oracle**: รองรับ ACID อย่างสมบูรณ์ด้วย Undo Segments, Redo Logs, และ MVCC
    - **MySQL**: รองรับ ACID เฉพาะใน Storage Engine เช่น InnoDB (MyISAM ไม่รองรับธุรกรรม)
        - ตัวอย่าง:
          ```sql
          START TRANSACTION;
          UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;
          UPDATE accounts SET balance = balance + 1000 WHERE account_id = 2;
          COMMIT;
          ```
    - **PostgreSQL**: รองรับ ACID ด้วย MVCC คล้าย Oracle
        - ตัวอย่าง:
          ```sql
          BEGIN;
          UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;
          UPDATE accounts SET balance = balance + 1000 WHERE account_id = 2;
          COMMIT;
          ```
    - **SQL Server**: รองรับ ACID ด้วย Transaction Log และ Locking
        - ตัวอย่าง:
          ```sql
          BEGIN TRANSACTION;
          UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;
          UPDATE accounts SET balance = balance + 1000 WHERE account_id = 2;
          COMMIT;
          ```
- **Concurrency Issues**:
    - **Dirty Reads**: อ่านข้อมูลที่ยังไม่ยืนยัน (Oracle ป้องกันด้วย Read Committed)
    - **Non-Repeatable Reads**: อ่านข้อมูลซ้ำในธุรกรรมเดียวกันแต่ได้ผลลัพธ์ต่างกัน
    - **Phantom Reads**: อ่านข้อมูลใหม่ที่แทรกโดยธุรกรรมอื่น
    - การใช้ Isolation Level ที่เหมาะสมช่วยลดปัญหาเหล่านี้

### การใช้งานใน PL/SQL
ตัวอย่างการใช้ PL/SQL เพื่อจัดการธุรกรรมโอนเงินพร้อมรับประกัน ACID:
```sql
CREATE OR REPLACE PROCEDURE transfer_funds(
    p_from_account NUMBER,
    p_to_account NUMBER,
    p_amount NUMBER
) IS
   v_from_balance NUMBER;
BEGIN
   -- ตรวจสอบยอดเงินคงเหลือ
   SELECT balance INTO v_from_balance
   FROM accounts
   WHERE account_id = p_from_account
   FOR UPDATE; -- ล็อกแถวเพื่อป้องกันการเปลี่ยนแปลง

   IF v_from_balance < p_amount THEN
      RAISE_APPLICATION_ERROR(-20001, 'Insufficient funds');
   END IF;

   -- เริ่มธุรกรรม
   SAVEPOINT before_transfer;

   -- ลดยอดเงินจากบัญชีต้นทาง
   UPDATE accounts
   SET balance = balance - p_amount
   WHERE account_id = p_from_account;

   -- เพิ่มยอดเงินให้บัญชีปลายทาง
   UPDATE accounts
   SET balance = balance + p_amount
   WHERE account_id = p_to_account;

   -- ยืนยันธุรกรรม
   COMMIT;
   DBMS_OUTPUT.PUT_LINE('Transfer of ' || p_amount || ' from account ' || p_from_account || ' to account ' || p_to_account || ' completed');
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
      ROLLBACK TO SAVEPOINT before_transfer;
      RAISE;
END;
/
```
- **ทดสอบ**:
  ```sql
  EXEC transfer_funds(1, 2, 1000); -- โอน 1000 จาก Alice ไป Bob
  ```
    - **ผลลัพธ์**:
      ```
      Transfer of 1000 from account 1 to account 2 completed
      ```
    - **ตรวจสอบ**:
      ```sql
      SELECT * FROM accounts;
      ```
        - **ผลลัพธ์**:
          ```
          account_id | account_name | balance
          -----------+--------------+---------
          1         | Alice        | 4000
          2         | Bob          | 4000
          ```
    - **ทดสอบข้อผิดพลาด**:
      ```sql
      EXEC transfer_funds(1, 2, 5000); -- ยอดเงินไม่พอ
      ```
        - **ผลลัพธ์**:
          ```
          Error: ORA-20001: Insufficient funds
          ```

#### การเชื่อมโยงกับ ACID
- **Atomicity**: การโอนเงินสำเร็จทั้งสอง `UPDATE` หรือยกเลิกทั้งหมดถ้ามีข้อผิดพลาด
- **Consistency**: Constraint `balance >= 0` และการตรวจสอบยอดเงินคงเหลือป้องกันสถานะที่ไม่ถูกต้อง
- **Isolation**: การใช้ `FOR UPDATE` ล็อกแถวเพื่อป้องกันการเปลี่ยนแปลงจากเซสชันอื่น
- **Durability**: หลัง `COMMIT` ข้อมูลถูกบันทึกลงดิสก์ด้วย Redo Log

### ตัวอย่างสถานการณ์จริง
สมมติมีระบบจัดการคำสั่งซื้อที่ต้องอัปเดตสต็อกสินค้าและบันทึกคำสั่งซื้อ:
```sql
CREATE TABLE products (
    product_id NUMBER PRIMARY KEY,
    product_name VARCHAR2(50),
    stock NUMBER
);
CREATE TABLE orders (
    order_id NUMBER PRIMARY KEY,
    product_id NUMBER,
    quantity NUMBER,
    order_date DATE
);
INSERT INTO products VALUES (1, 'Laptop', 10);
INSERT INTO products VALUES (2, 'Mouse', 50);
COMMIT;

CREATE SEQUENCE order_seq START WITH 1 INCREMENT BY 1;
```
- **ธุรกรรม**:
```sql
CREATE OR REPLACE PROCEDURE place_order(
    p_product_id NUMBER,
    p_quantity NUMBER
) IS
   v_stock NUMBER;
BEGIN
   -- ล็อกแถวสินค้า
   SELECT stock INTO v_stock
   FROM products
   WHERE product_id = p_product_id
   FOR UPDATE;

   IF v_stock < p_quantity THEN
      RAISE_APPLICATION_ERROR(-20001, 'Insufficient stock');
   END IF;

   SAVEPOINT before_order;

   -- ลดสต็อก
   UPDATE products
   SET stock = stock - p_quantity
   WHERE product_id = p_product_id;

   -- บันทึกคำสั่งซื้อ
   INSERT INTO orders (order_id, product_id, quantity, order_date)
   VALUES (order_seq.NEXTVAL, p_product_id, p_quantity, SYSDATE);

   COMMIT;
   DBMS_OUTPUT.PUT_LINE('Order placed for product ' || p_product_id);
EXCEPTION
   WHEN OTHERS THEN
      DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
      ROLLBACK TO SAVEPOINT before_order;
      RAISE;
END;
/
```
- **ทดสอบ**:
  ```sql
  EXEC place_order(1, 3); -- สั่งซื้อ Laptop 3 ชิ้น
  ```
    - **ผลลัพธ์**:
      ```
      Order placed for product 1
      ```
    - **ตรวจสอบ**:
      ```sql
      SELECT * FROM products WHERE product_id = 1;
      SELECT * FROM orders;
      ```
        - **ผลลัพธ์**:
          ```
          product_id | product_name | stock
          -----------+--------------+------
          1         | Laptop       | 7
    
          order_id | product_id | quantity | order_date
          ---------+------------+----------+------------
          1       | 1          | 3        | 2025-04-23
          ```
    - **ทดสอบข้อผิดพลาด**:
      ```sql
      EXEC place_order(1, 10); -- สต็อกไม่พอ
      ```
        - **ผลลัพธ์**:
          ```
          Error: ORA-20001: Insufficient stock
          ```

#### การเชื่อมโยงกับ ACID
- **Atomicity**: การลดสต็อกและบันทึกคำสั่งซื้อสำเร็จทั้งคู่หรือยกเลิกทั้งหมด
- **Consistency**: การตรวจสอบสต็อกและ Primary Key Constraint ป้องกันข้อมูลที่ไม่ถูกต้อง
- **Isolation**: การล็อกแถวด้วย `FOR UPDATE` ป้องกันการเปลี่ยนแปลงพร้อมกัน
- **Durability**: ข้อมูลคำสั่งซื้อและสต็อกถูกบันทึกถาวรหลัง `COMMIT`
---
