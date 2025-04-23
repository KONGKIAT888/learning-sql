# บทที่ 5: ฟังก์ชันและเงื่อนไข
## ฟังก์ชันเชิงตัวเลข: `ROUND`, `CEIL`, `FLOOR`
ใน SQL ฟังก์ชันเชิงตัวเลข เช่น **ROUND**, **CEIL**, และ **FLOOR** ใช้สำหรับจัดการและประมวลผลค่าตัวเลข โดยเฉพาะการปัดเศษหรือปรับค่าให้อยู่ในรูปแบบที่ต้องการ ฟังก์ชันเหล่านี้มีประโยชน์ในงานที่เกี่ยวข้องกับการคำนวณ เช่น การเงิน, สถิติ, หรือการแสดงผลข้อมูล ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### 1. ROUND
- **คำอธิบาย**: ปัดเศษตัวเลขให้เป็นจำนวนทศนิยมตามจำนวนตำแหน่งที่ระบุ ตามกฎการปัดเศษ (ถ้าส่วนทศนิยม ≥ 5 ปัดขึ้น, < 5 ปัดลง)
- **ไวยากรณ์**:
  ```sql
  ROUND(number, [decimal_places])
  ```
    - `number`: ค่าตัวเลขที่ต้องการปัดเศษ
    - `decimal_places`: จำนวนตำแหน่งทศนิยม (ค่าเริ่มต้น = 0)
        - ถ้าเป็นบวก: ปัดทศนิยม
        - ถ้าเป็นลบ: ปัดส่วนจำนวนเต็ม (เช่น -1 ปัดเป็นสิบ, -2 ปัดเป็นร้อย)
- **ตัวอย่าง**:
  ```sql
  SELECT ROUND(123.456, 2) AS rounded FROM dual;
  ```
    - ปัด 123.456 ให้เหลือ 2 ทศนิยม
    - **ผลลัพธ์**: `123.46`

  ```sql
  SELECT ROUND(123.456, 0) AS rounded FROM dual;
  ```
    - ปัดเป็นจำนวนเต็ม
    - **ผลลัพธ์**: `123`

  ```sql
  SELECT ROUND(123.456, -1) AS rounded FROM dual;
  ```
    - ปัดให้เป็นสิบ
    - **ผลลัพธ์**: `120`

### 2. CEIL (Ceiling)
- **คำอธิบาย**: ปัดตัวเลขขึ้นไปเป็นจำนวนเต็มที่มากกว่าหรือเท่ากับค่าที่ให้มา (ปัดขึ้นเสมอ)
- **ไวยากรณ์**:
  ```sql
  CEIL(number)
  ```
    - `number`: ค่าตัวเลขที่ต้องการปัด
    - ไม่รับพารามิเตอร์เพิ่มเติม (ปัดเป็นจำนวนเต็มเท่านั้น)
- **ตัวอย่าง**:
  ```sql
  SELECT CEIL(123.456) AS ceiling FROM dual;
  ```
    - ปัด 123.456 ขึ้นเป็นจำนวนเต็ม
    - **ผลลัพธ์**: `124`

  ```sql
  SELECT CEIL(-123.456) AS ceiling FROM dual;
  ```
    - ปัด -123.456 ขึ้น (ใกล้ 0 มากขึ้น)
    - **ผลลัพธ์**: `-123`

  ```sql
  SELECT CEIL(123.0) AS ceiling FROM dual;
  ```
    - ถ้าเป็นจำนวนเต็มอยู่แล้ว จะไม่เปลี่ยน
    - **ผลลัพธ์**: `123`

### 3. FLOOR
- **คำอธิบาย**: ปัดตัวเลขลงไปเป็นจำนวนเต็มที่น้อยกว่าหรือเท่ากับค่าที่ให้มา (ปัดลงเสมอ)
- **ไวยากรณ์**:
  ```sql
  FLOOR(number)
  ```
    - `number`: ค่าตัวเลขที่ต้องการปัด
    - ไม่รับพารามิเตอร์เพิ่มเติม (ปัดเป็นจำนวนเต็มเท่านั้น)
- **ตัวอย่าง**:
  ```sql
  SELECT FLOOR(123.456) AS floor FROM dual;
  ```
    - ปัด 123.456 ลงเป็นจำนวนเต็ม
    - **ผลลัพธ์**: `123`

  ```sql
  SELECT FLOOR(-123.456) AS floor FROM dual;
  ```
    - ปัด -123.456 ลง (ห่างจาก 0 มากขึ้น)
    - **ผลลัพธ์**: `-124`

  ```sql
  SELECT FLOOR(123.0) AS floor FROM dual;
  ```
    - ถ้าเป็นจำนวนเต็มอยู่แล้ว จะไม่เปลี่ยน
    - **ผลลัพธ์**: `123`

### ตัวอย่างการใช้งานจริง:
สมมติมีตาราง `employees` ดังนี้:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    salary NUMBER(10,2)
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 52345.678);
INSERT INTO employees VALUES (2, 'Jane', 61234.123);
INSERT INTO employees VALUES (3, 'Alice', 55000.999);
COMMIT;
```

#### 1. ใช้ ROUND เพื่อปัดเงินเดือน:
```sql
SELECT first_name, salary, ROUND(salary, 1) AS rounded_salary
FROM employees;
```
- ปัดเงินเดือนให้เหลือ 1 ทศนิยม
- **ผลลัพธ์**:
  ```
  first_name | salary     | rounded_salary
  ------------+------------+---------------
  John       | 52345.678  | 52345.7
  Jane       | 61234.123  | 61234.1
  Alice      | 55000.999  | 55001.0
  ```

#### 2. ใช้ CEIL เพื่อปัดเงินเดือนขึ้น:
```sql
SELECT first_name, salary, CEIL(salary / 1000) AS ceiling_thousands
FROM employees;
```
- หารเงินเดือนด้วย 1,000 แล้วปัดขึ้นเป็นจำนวนเต็ม (เช่น เพื่อดูเงินเดือนในหน่วยพัน)
- **ผลลัพธ์**:
  ```
  first_name | salary     | ceiling_thousands
  ------------+------------+------------------
  John       | 52345.678  | 53
  Jane       | 61234.123  | 62
  Alice      | 55000.999  | 56
  ```

#### 3. ใช้ FLOOR เพื่อปัดเงินเดือนลง:
```sql
SELECT first_name, salary, FLOOR(salary / 1000) AS floor_thousands
FROM employees;
```
- หารเงินเดือนด้วย 1,000 แล้วปัดลงเป็นจำนวนเต็ม
- **ผลลัพธ์**:
  ```
  first_name | salary     | floor_thousands
  ------------+------------+----------------
  John       | 52345.678  | 52
  Jane       | 61234.123  | 61
  Alice      | 55000.999  | 55
  ```

#### 4. รวมทั้งสามฟังก์ชัน:
```sql
SELECT first_name, 
       salary,
       ROUND(salary, 0) AS rounded,
       CEIL(salary) AS ceiling,
       FLOOR(salary) AS floor
FROM employees;
```
- แสดงเงินเดือนที่ปัดด้วยทั้งสามวิธี
- **ผลลัพธ์**:
  ```
  first_name | salary     | rounded  | ceiling  | floor
  ------------+------------+----------+----------+--------
  John       | 52345.678  | 52346    | 52346    | 52345
  Jane       | 61234.123  | 61234    | 61235    | 61234
  Alice      | 55000.999  | 55001    | 55001    | 55000
  ```

### ข้อควรระวัง:
- **ROUND กับค่าลบใน decimal_places**:
    - การใช้ `ROUND(number, -n)` จะปัดส่วนจำนวนเต็ม เช่น `ROUND(123.456, -2)` ปัดเป็นร้อย ได้ `100`
- **CEIL และ FLOOR กับจำนวนเต็ม**:
    - ถ้าค่าเป็นจำนวนเต็มอยู่แล้ว ทั้ง `CEIL` และ `FLOOR` จะคืนค่าเดิม
- **ความแม่นยำ**:
    - ฟังก์ชันเหล่านี้ทำงานกับชนิดข้อมูล `NUMBER` ได้ดี แต่ในบางระบบ (เช่น MySQL) อาจต้องระวังความแม่นยำของทศนิยมใน `FLOAT` หรือ `DOUBLE`
- **ระบบฐานข้อมูล**:
    - Oracle, MySQL, PostgreSQL, SQL Server รองรับ `ROUND`, `CEIL`, `FLOOR` แต่ชื่อหรือพฤติกรรมอาจแตกต่างเล็กน้อย:
        - MySQL: ใช้ `CEILING` แทน `CEIL`
        - SQL Server: `ROUND` รองรับพารามิเตอร์ที่สามเพื่อกำหนดวิธีปัด (เช่น truncate)
- **ประสิทธิภาพ**:
    - การใช้ฟังก์ชันเชิงตัวเลขในตารางขนาดใหญ่ควรคำนึงถึงประสิทธิภาพ โดยเฉพาะเมื่อใช้ใน `WHERE` หรือ `JOIN`

### การใช้งานใน PL/SQL:
สามารถใช้ฟังก์ชันเหล่านี้ใน PL/SQL เพื่อประมวลผลข้อมูล เช่น:
```sql
DECLARE
   v_salary NUMBER := 52345.678;
   v_rounded NUMBER;
   v_ceiling NUMBER;
   v_floor NUMBER;
BEGIN
   v_rounded := ROUND(v_salary, 1);
   v_ceiling := CEIL(v_salary);
   v_floor := FLOOR(v_salary);
   
   DBMS_OUTPUT.PUT_LINE('Original: ' || v_salary);
   DBMS_OUTPUT.PUT_LINE('Rounded: ' || v_rounded);
   DBMS_OUTPUT.PUT_LINE('Ceiling: ' || v_ceiling);
   DBMS_OUTPUT.PUT_LINE('Floor: ' || v_floor);
END;
/
```
- **ผลลัพธ์**:
  ```
  Original: 52345.678
  Rounded: 52345.7
  Ceiling: 52346
  Floor: 52345
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการปรับเงินเดือนให้เป็นจำนวนเต็มหรือปัดเป็นหลักพัน:
```sql
SELECT first_name,
       salary,
       ROUND(salary, 0) AS rounded_salary,
       FLOOR(salary / 1000) * 1000 AS floor_thousands
FROM employees
WHERE department_id = 10;
```
- ปัดเงินเดือนเป็นจำนวนเต็มและปัดลงเป็นหลักพันสำหรับพนักงานในแผนก 10
- **ผลลัพธ์** (สมมติเพิ่มข้อมูล `department_id`):
  ```
  first_name | salary     | rounded_salary | floor_thousands
  ------------+------------+---------------+----------------
  John       | 52345.678  | 52346         | 52000
  Alice      | 55000.999  | 55001         | 55000
  ```
---

## ฟังก์ชันเชิงสตริง: `UPPER`, `LOWER`, `SUBSTR`, `INSTR`
ใน SQL ฟังก์ชันเชิงสตริง เช่น **UPPER**, **LOWER**, **SUBSTR**, และ **INSTR** ใช้สำหรับจัดการและประมวลผลข้อมูลประเภทสตริง (เช่น `VARCHAR2`, `CHAR`) เพื่อเปลี่ยนรูปแบบ, แยกส่วน, หรือค้นหาข้อมูลภายในสตริง ฟังก์ชันเหล่านี้มีประโยชน์ในงานที่เกี่ยวข้องกับการจัดการข้อความ เช่น การค้นหา, การจัดรูปแบบข้อมูล, หรือการตรวจสอบข้อมูล ต่อไปนี้คือคำอธิบายและตัวอย่างการใช้งานในบริบทของ Oracle SQL (และระบบฐานข้อมูลทั่วไป)

### 1. UPPER
- **คำอธิบาย**: แปลงสตริงทั้งหมดเป็นตัวพิมพ์ใหญ่ (uppercase)
- **ไวยากรณ์**:
  ```sql
  UPPER(string)
  ```
    - `string`: สตริงที่ต้องการแปลง
- **ตัวอย่าง**:
  ```sql
  SELECT UPPER('Hello World') AS upper_string FROM dual;
  ```
    - **ผลลัพธ์**: `HELLO WORLD`

  ```sql
  SELECT first_name, UPPER(last_name) AS upper_last_name
  FROM employees;
  ```
    - แปลง `last_name` เป็นตัวพิมพ์ใหญ่
    - **ผลลัพธ์** (สมมติใช้ตาราง `employees`):
      ```
      first_name | upper_last_name
      -----------+----------------
      John       | DOE
      Jane       | SMITH
      ```

- **การใช้งาน**: เหมาะสำหรับการทำให้สตริงเป็นรูปแบบเดียวกัน เช่น การเปรียบเทียบที่ไม่สนใจตัวพิมพ์ใหญ่-เล็ก

### 2. LOWER
- **คำอธิบาย**: แปลงสตริงทั้งหมดเป็นตัวพิมพ์เล็ก (lowercase)
- **ไวยากรณ์**:
  ```sql
  LOWER(string)
  ```
    - `string`: สตริงที่ต้องการแปลง
- **ตัวอย่าง**:
  ```sql
  SELECT LOWER('Hello World') AS lower_string FROM dual;
  ```
    - **ผลลัพธ์**: `hello world`

  ```sql
  SELECT first_name, LOWER(email) AS lower_email
  FROM employees;
  ```
    - แปลง `email` เป็นตัวพิมพ์เล็ก
    - **ผลลัพธ์**:
      ```
      first_name | lower_email
      -----------+------------------------
      John       | john.doe@example.com
      Jane       | jane.smith@example.com
      ```

- **การใช้งาน**: ใช้ในการค้นหาหรือเปรียบเทียบสตริงที่ไม่สนใจตัวพิมพ์ใหญ่-เล็ก

### 3. SUBSTR (Substring)
- **คำอธิบาย**: แยกส่วนของสตริงตามตำแหน่งและความยาวที่ระบุ
- **ไวยากรณ์**:
  ```sql
  SUBSTR(string, start_position, [length])
  ```
    - `string`: สตริงที่ต้องการแยก
    - `start_position`: ตำแหน่งเริ่มต้น (เริ่มจาก 1; ถ้าเป็นลบ จะนับจากท้ายสตริง)
    - `length`: จำนวนอักขระที่ต้องการแยก (ถ้าไม่ระบุ จะแยกจนถึงท้ายสตริง)
- **ตัวอย่าง**:
  ```sql
  SELECT SUBSTR('Hello World', 1, 5) AS substring FROM dual;
  ```
    - แยก 5 ตัวอักษรแรก
    - **ผลลัพธ์**: `Hello`

  ```sql
  SELECT SUBSTR('Hello World', 7) AS substring FROM dual;
  ```
    - แยกตั้งแต่ตำแหน่งที่ 7 จนถึงท้ายสตริง
    - **ผลลัพธ์**: `World`

  ```sql
  SELECT SUBSTR('Hello World', -5, 5) AS substring FROM dual;
  ```
    - แยก 5 ตัวอักษรจากท้ายสตริง
    - **ผลลัพธ์**: `World`

  ```sql
  SELECT first_name, SUBSTR(email, 1, INSTR(email, '@') - 1) AS username
  FROM employees;
  ```
    - แยกส่วนชื่อผู้ใช้ก่อน `@` ใน `email`
    - **ผลลัพธ์**:
      ```
      first_name | username
      -----------+---------
      John       | john.doe
      Jane       | jane.smith
      ```

- **การใช้งาน**: ใช้แยกส่วนของสตริง เช่น ชื่อผู้ใช้จากอีเมล, รหัสย่อ, หรือส่วนของข้อความ

### 4. INSTR (In String)
- **คำอธิบาย**: ค้นหาตำแหน่งของสตริงย่อยภายในสตริง และคืนค่าตำแหน่งแรกที่พบ (นับจาก 1)
- **ไวยากรณ์**:
  ```sql
  INSTR(string, substring, [start_position], [occurrence])
  ```
    - `string`: สตริงที่ต้องการค้นหา
    - `substring`: สตริงย่อยที่ต้องการหาตำแหน่ง
    - `start_position`: ตำแหน่งเริ่มค้นหา (ค่าเริ่มต้น = 1; ถ้าเป็นลบ จะค้นจากท้าย)
    - `occurrence`: ครั้งที่ต้องการหา (ค่าเริ่มต้น = 1 เช่น ครั้งแรก)
- **ตัวอย่าง**:
  ```sql
  SELECT INSTR('Hello World', 'l') AS position FROM dual;
  ```
    - หาตำแหน่งแรกของ 'l'
    - **ผลลัพธ์**: `3`

  ```sql
  SELECT INSTR('Hello World', 'l', 4, 2) AS position FROM dual;
  ```
    - หาตำแหน่งของ 'l' ครั้งที่ 2 โดยเริ่มค้นจากตำแหน่งที่ 4
    - **ผลลัพธ์**: `9` (ตำแหน่งของ 'l' ใน "World")

  ```sql
  SELECT INSTR('Hello World', 'x') AS position FROM dual;
  ```
    - ถ้าไม่พบสตริงย่อย จะคืนค่า `0`
    - **ผลลัพธ์**: `0`

  ```sql
  SELECT first_name, INSTR(email, '@') AS at_position
  FROM employees;
  ```
    - หาตำแหน่งของ '@' ใน `email`
    - **ผลลัพธ์**:
      ```
      first_name | at_position
      -----------+------------
      John       | 9
      Jane       | 11
      ```

- **การใช้งาน**: ใช้ค้นหาตำแหน่งของอักขระหรือสตริงย่อย เช่น หาตำแหน่ง '@' ในอีเมล หรือตรวจสอบการมีอยู่ของคำ

### ตัวอย่างการใช้งานจริง:
สมมติมีตาราง `employees` ดังนี้:
```sql
CREATE TABLE employees (
    employee_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50),
    last_name VARCHAR2(50),
    email VARCHAR2(100)
);
```
และมีข้อมูล:
```sql
INSERT INTO employees VALUES (1, 'John', 'Doe', 'John.Doe@example.com');
INSERT INTO employees VALUES (2, 'Jane', 'Smith', 'Jane.Smith@example.com');
INSERT INTO employees VALUES (3, 'Alice', 'Brown', 'Alice.Brown@example.com');
COMMIT;
```

#### 1. แปลงชื่อเป็นตัวพิมพ์ใหญ่และเล็ก:
```sql
SELECT first_name, 
       UPPER(last_name) AS upper_last_name, 
       LOWER(email) AS lower_email
FROM employees;
```
- **ผลลัพธ์**:
  ```
  first_name | upper_last_name | lower_email
  ------------+----------------+------------------------
  John       | DOE            | john.doe@example.com
  Jane       | SMITH          | jane.smith@example.com
  Alice      | BROWN          | alice.brown@example.com
  ```

#### 2. แยกชื่อผู้ใช้จากอีเมล:
```sql
SELECT first_name, 
       SUBSTR(email, 1, INSTR(email, '@') - 1) AS username
FROM employees;
```
- แยกส่วนก่อน '@' ใน `email`
- **ผลลัพธ์**:
  ```
  first_name | username
  ------------+------------
  John       | John.Doe
  Jane       | Jane.Smith
  Alice      | Alice.Brown
  ```

#### 3. ค้นหาตำแหน่งและตัดสตริง:
```sql
SELECT first_name, 
       INSTR(email, '.') AS dot_position,
       SUBSTR(email, INSTR(email, '.') + 1, 
              INSTR(email, '@') - INSTR(email, '.') - 1) AS last_name_part
FROM employees;
```
- หาตำแหน่งของ '.' และแยกส่วนนามสกุลจาก `email`
- **ผลลัพธ์**:
  ```
  first_name | dot_position | last_name_part
  ------------+--------------+---------------
  John       | 5            | Doe
  Jane       | 5            | Smith
  Alice      | 6            | Brown
  ```

#### 4. รวมฟังก์ชันเพื่อค้นหาและแปลง:
```sql
SELECT first_name,
       UPPER(SUBSTR(email, 1, INSTR(email, '@') - 1)) AS upper_username
FROM employees
WHERE LOWER(last_name) LIKE 's%';
```
- แยกชื่อผู้ใช้จาก `email`, แปลงเป็นตัวพิมพ์ใหญ่, และกรองเฉพาะนามสกุลที่ขึ้นต้นด้วย 's'
- **ผลลัพธ์**:
  ```
  first_name | upper_username
  ------------+---------------
  Jane       | JANE.SMITH
  ```

### ข้อควรระวัง:
- **NULL**: ถ้าสตริงเป็น `NULL` ฟังก์ชันจะคืนค่า `NULL` (ยกเว้นบางกรณี เช่น `INSTR` คืน `0` ถ้าไม่พบ)
  ```sql
  SELECT UPPER(NULL) FROM dual; -- คืน NULL
  ```
- **ตำแหน่งใน SUBSTR/INSTR**: ใน Oracle การนับตำแหน่งเริ่มจาก 1 (ไม่ใช่ 0 เหมือนบางภาษาโปรแกรม)
- **ตัวพิมพ์ใหญ่-เล็ก**: `UPPER` และ `LOWER` ไม่มีผลกับอักขระที่ไม่ใช่ตัวอักษร เช่น ตัวเลขหรือสัญลักษณ์
- **ประสิทธิภาพ**: การใช้ฟังก์ชันสตริงใน `WHERE` หรือ `JOIN` อาจทำให้การค้นหาช้า ควรใช้ดัชนีหรือปรับแต่ง Query ถ้าจำเป็น
- **ระบบฐานข้อมูล**:
    - **Oracle**: ใช้ `SUBSTR`, `INSTR` ตามที่อธิบาย
    - **MySQL**: ใช้ `SUBSTRING` หรือ `SUBSTR`, และ `INSTR` คล้ายกัน
    - **PostgreSQL**: ใช้ `SUBSTRING` และ `POSITION` แทน `INSTR`
    - **SQL Server**: ใช้ `SUBSTRING` และ `CHARINDEX` แทน `INSTR`
    - ตัวอย่างใน MySQL:
      ```sql
      SELECT SUBSTRING('Hello World', 1, 5); -- คืน 'Hello'
      SELECT LOCATE('l', 'Hello World'); -- คล้าย INSTR, คืน 3
      ```

### การใช้งานใน PL/SQL:
สามารถใช้ฟังก์ชันเหล่านี้ใน PL/SQL เพื่อประมวลผลสตริง เช่น:
```sql
DECLARE
   v_email VARCHAR2(100) := 'John.Doe@example.com';
   v_username VARCHAR2(50);
   v_domain VARCHAR2(50);
BEGIN
   v_username := UPPER(SUBSTR(v_email, 1, INSTR(v_email, '@') - 1));
   v_domain := LOWER(SUBSTR(v_email, INSTR(v_email, '@') + 1));
   
   DBMS_OUTPUT.PUT_LINE('Username: ' || v_username);
   DBMS_OUTPUT.PUT_LINE('Domain: ' || v_domain);
END;
/
```
- **ผลลัพธ์**:
  ```
  Username: JOHN.DOE
  Domain: example.com
  ```

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการตรวจสอบอีเมลและแยกส่วนประกอบ:
```sql
SELECT first_name,
       email,
       UPPER(SUBSTR(email, 1, INSTR(email, '@') - 1)) AS upper_username,
       LOWER(SUBSTR(email, INSTR(email, '@') + 1)) AS domain
FROM employees
WHERE INSTR(email, 'example.com') > 0;
```
- แยกชื่อผู้ใช้และโดเมนจาก `email`, แปลงชื่อผู้ใช้เป็นตัวพิมพ์ใหญ่, โดเมนเป็นตัวพิมพ์เล็ก, และกรองเฉพาะอีเมลที่ใช้โดเมน 'example.com'
- **ผลลัพธ์**:
  ```
  first_name | email                    | upper_username | domain
  ------------+--------------------------+---------------+--------------
  John       | John.Doe@example.com     | JOHN.DOE      | example.com
  Jane       | Jane.Smith@example.com   | JANE.SMITH    | example.com
  Alice      | Alice.Brown@example.com  | ALICE.BROWN   | example.com
  ```
---

## เงื่อนไข `CASE`, `DECODE`
ใน SQL และ PL/SQL เงื่อนไข **CASE** และ **DECODE** ใช้สำหรับกำหนด**ตรรกะเงื่อนไข** (Conditional Logic) เพื่อแปลงหรือจัดการข้อมูลตามเงื่อนไขที่กำหนด ทั้งสองช่วยให้สามารถกำหนดผลลัพธ์ที่แตกต่างกันขึ้นอยู่กับค่าในคอลัมน์หรือนิพจน์ โดย **CASE** มีความยืดหยุ่นมากกว่าและเป็นมาตรฐาน SQL ส่วน **DECODE** เป็นฟังก์ชันเฉพาะของ Oracle SQL

### 1. CASE
- **คำอธิบาย**: เงื่อนไข `CASE` ทำงานเหมือนโครงสร้าง `IF-THEN-ELSE` ในภาษาโปรแกรม โดยสามารถใช้ในคำสั่ง `SELECT`, `WHERE`, `ORDER BY`, หรือภายใน PL/SQL เพื่อกำหนดผลลัพธ์ตามเงื่อนไข
- **ประเภท**:
    1. **Simple CASE**: เปรียบเทียบค่ากับชุดของเงื่อนไข
    2. **Searched CASE**: ใช้เงื่อนไขที่ซับซ้อน (เช่น การเปรียบเทียบด้วย `>`, `<`, `LIKE`)

- **ไวยากรณ์**:
    - **Simple CASE**:
      ```sql
      CASE expression
          WHEN value1 THEN result1
          WHEN value2 THEN result2
          ...
          [ELSE result_else]
      END
      ```
    - **Searched CASE**:
      ```sql
      CASE
          WHEN condition1 THEN result1
          WHEN condition2 THEN result2
          ...
          [ELSE result_else]
      END
      ```

- **ตัวอย่าง**:
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
  COMMIT;
  ```

  #### Simple CASE:
  ```sql
  SELECT first_name, department_id,
         CASE department_id
             WHEN 10 THEN 'Sales'
             WHEN 20 THEN 'IT'
             ELSE 'Other'
         END AS dept_name
  FROM employees;
  ```
    - แปลง `department_id` เป็นชื่อแผนก
    - **ผลลัพธ์**:
      ```
      first_name | department_id | dept_name
      -----------+---------------+----------
      John       | 10            | Sales
      Jane       | 20            | IT
      Alice      | 10            | Sales
      Bob        | 20            | IT
      ```

  #### Searched CASE:
  ```sql
  SELECT first_name, salary,
         CASE
             WHEN salary < 55000 THEN 'Low'
             WHEN salary BETWEEN 55000 AND 60000 THEN 'Medium'
             ELSE 'High'
         END AS salary_level
  FROM employees;
  ```
    - จัดหมวดหมู่เงินเดือนตามช่วง
    - **ผลลัพธ์**:
      ```
      first_name | salary   | salary_level
      -----------+----------+-------------
      John       | 50000.00 | Low
      Jane       | 60000.00 | Medium
      Alice      | 55000.00 | Medium
      Bob        | 65000.00 | High
      ```

  #### ใช้ใน ORDER BY:
  ```sql
  SELECT first_name, salary
  FROM employees
  ORDER BY CASE
              WHEN salary < 55000 THEN 1
              WHEN salary <= 60000 THEN 2
              ELSE 3
           END;
  ```
    - เรียงลำดับตามระดับเงินเดือน (ต่ำ, กลาง, สูง)
    - **ผลลัพธ์**:
      ```
      first_name | salary
      -----------+--------
      John       | 50000.00
      Jane       | 60000.00
      Alice      | 55000.00
      Bob        | 65000.00
      ```

### 2. DECODE
- **คำอธิบาย**: ฟังก์ชันเฉพาะของ Oracle SQL ใช้สำหรับแปลงค่าตามเงื่อนไข คล้าย **Simple CASE** แต่มีความยืดหยุ่นน้อยกว่า เพราะรองรับเฉพาะการเปรียบเทียบค่าเท่ากัน (`=`) เท่านั้น
- **ไวยากรณ์**:
  ```sql
  DECODE(expression, value1, result1, value2, result2, ..., [default_result])
  ```
    - `expression`: ค่าหรือคอลัมน์ที่ต้องการเปรียบเทียบ
    - `value1, result1`: ถ้า `expression = value1` จะคืน `result1`
    - `default_result`: ค่าที่คืนถ้าไม่ตรงกับ `value` ใด ๆ (ถ้าไม่ระบุ คืน `NULL`)

- **ตัวอย่าง**:
  ```sql
  SELECT first_name, department_id,
         DECODE(department_id,
                10, 'Sales',
                20, 'IT',
                'Other') AS dept_name
  FROM employees;
  ```
    - แปลง `department_id` เป็นชื่อแผนก
    - **ผลลัพธ์**: เดียวกับตัวอย่าง **Simple CASE** ข้างต้น
      ```
      first_name | department_id | dept_name
      -----------+---------------+----------
      John       | 10            | Sales
      Jane       | 20            | IT
      Alice      | 10            | Sales
      Bob        | 20            | IT
      ```

  ```sql
  SELECT first_name, salary,
         DECODE(SIGN(salary - 55000),
                -1, 'Low',
                0, 'Medium',
                1, 'High') AS salary_level
  FROM employees;
  ```
    - ใช้ `SIGN` เพื่อจำลองเงื่อนไขเปรียบเทียบ (`<`, `=`, `>`) และจัดหมวดหมู่เงินเดือน
    - **ผลลัพธ์**:
      ```
      first_name | salary   | salary_level
      -----------+----------+-------------
      John       | 50000.00 | Low
      Jane       | 60000.00 | High
      Alice      | 55000.00 | Medium
      Bob        | 65000.00 | High
      ```

### การเปรียบเทียบ CASE กับ DECODE
| คุณสมบัติ              | CASE                              | DECODE                          |
|------------------------|-----------------------------------|---------------------------------|
| **ประเภท**            | มาตรฐาน SQL (ใช้ได้ทุก DBMS)      | เฉพาะ Oracle                   |
| **ความยืดหยุ่น**     | รองรับเงื่อนไขซับซ้อน (>, <, LIKE, AND) | จำกัดที่เปรียบเทียบ `=` เท่านั้น |
| **อ่านง่าย**          | อ่านง่ายกว่าในเงื่อนไขซับซ้อน    | อ่านง่ายในเงื่อนไขง่าย ๆ      |
| **ประสิทธิภาพ**       | เทียบเท่าหรือดีกว่าในบางกรณี     | เทียบเท่าในกรณีง่าย ๆ         |
| **การใช้งาน**         | ใช้ใน `SELECT`, `WHERE`, `ORDER BY`, PL/SQL | ใช้ใน `SELECT` และ PL/SQL เท่านั้น |

- **เมื่อใช้ CASE**:
    - เงื่อนไขซับซ้อน เช่น การเปรียบเทียบช่วง (`BETWEEN`), การใช้ `LIKE`, หรือหลายเงื่อนไข
    - ต้องการเขียนโค้ดที่ย้ายไปใช้ใน DBMS อื่นได้
- **เมื่อใช้ DECODE**:
    - เงื่อนไขง่าย ๆ เช่น การแปลงรหัสเป็นชื่อ
    - อยู่ใน Oracle และต้องการโค้ดสั้น ๆ

### การใช้งานใน PL/SQL
ทั้ง `CASE` และ `DECODE` สามารถใช้ใน PL/SQL เพื่อควบคุมตรรกะ เช่น:

#### ตัวอย่าง CASE:
```sql
DECLARE
   v_salary NUMBER := 58000;
   v_salary_level VARCHAR2(20);
BEGIN
   v_salary_level := CASE
                        WHEN v_salary < 55000 THEN 'Low'
                        WHEN v_salary <= 60000 THEN 'Medium'
                        ELSE 'High'
                     END;
   DBMS_OUTPUT.PUT_LINE('Salary Level: ' || v_salary_level);
END;
/
```
- **ผลลัพธ์**: `Salary Level: Medium`

#### ตัวอย่าง DECODE:
```sql
DECLARE
   v_dept_id NUMBER := 10;
   v_dept_name VARCHAR2(50);
BEGIN
   v_dept_name := DECODE(v_dept_id,
                         10, 'Sales',
                         20, 'IT',
                         'Other');
   DBMS_OUTPUT.PUT_LINE('Department: ' || v_dept_name);
END;
/
```
- **ผลลัพธ์**: `Department: Sales`

### ตัวอย่างสถานการณ์จริง:
สมมติต้องการจัดหมวดหมู่พนักงานตามเงินเดือนและแสดงชื่อแผนก:

#### ใช้ CASE:
```sql
SELECT first_name, salary, department_id,
       CASE
           WHEN salary < 55000 THEN 'Low'
           WHEN salary BETWEEN 55000 AND 60000 THEN 'Medium'
           ELSE 'High'
       END AS salary_level,
       CASE department_id
           WHEN 10 THEN 'Sales'
           WHEN 20 THEN 'IT'
           ELSE 'Other'
       END AS dept_name
FROM employees
WHERE department_id IN (10, 20)
ORDER BY salary DESC;
```
- **ผลลัพธ์**:
  ```
  first_name | salary   | department_id | salary_level | dept_name
  -----------+----------+---------------+-------------+----------
  Bob        | 65000.00 | 20            | High        | IT
  Jane       | 60000.00 | 20            | Medium      | IT
  Alice      | 55000.00 | 10            | Medium      | Sales
  John       | 50000.00 | 10            | Low         | Sales
  ```

#### ใช้ DECODE:
```sql
SELECT first_name, salary, department_id,
       DECODE(SIGN(salary - 55000),
              -1, 'Low',
              0, 'Medium',
              1, 'High') AS salary_level,
       DECODE(department_id,
              10, 'Sales',
              20, 'IT',
              'Other') AS dept_name
FROM employees
WHERE department_id IN (10, 20)
ORDER BY salary DESC;
```
- **ผลลัพธ์**: เดียวกับด้านบน

### ข้อควรระวัง:
- **CASE**:
    - ต้องระบุ `END` ปิดเงื่อนไขเสมอ
    - ถ้าไม่มี `ELSE` และเงื่อนไขไม่ตรง จะคืน `NULL`
    - รองรับเงื่อนไขซับซ้อน แต่โค้ดอาจยาวถ้าเงื่อนไขเยอะ
- **DECODE**:
    - จำกัดเฉพาะ Oracle ถ้าใช้ในระบบอื่น (เช่น MySQL, PostgreSQL) ต้องเปลี่ยนเป็น `CASE`
    - ถ้าไม่มี `default_result` และค่าไม่ตรง จะคืน `NULL`
    - การใช้กับเงื่อนไขที่ซับซ้อน (เช่น `>`, `<`) ต้องรวมกับฟังก์ชันอื่น เช่น `SIGN`
- **ประสิทธิภาพ**: ทั้ง `CASE` และ `DECODE` มีประสิทธิภาพใกล้เคียงกันใน Oracle แต่ `CASE` อาจดีกว่าในเงื่อนไขซับซ้อน
- **NULL**: ทั้งสองจัดการ `NULL` อย่างระวัง เช่น `DECODE(NULL, NULL, 'Match')` จะคืน `'Match'` เพราะ `NULL` ถือว่าเท่ากันใน `DECODE`

### การใช้งานในระบบฐานข้อมูลอื่น:
- **MySQL, PostgreSQL, SQL Server**:
    - รองรับ `CASE` (ทั้ง Simple และ Searched) แต่ไม่มี `DECODE`
    - ตัวอย่างใน MySQL:
      ```sql
      SELECT first_name,
             CASE department_id
                 WHEN 10 THEN 'Sales'
                 WHEN 20 THEN 'IT'
                 ELSE 'Other'
             END AS dept_name
      FROM employees;
      ```
- **การแทน DECODE**:
    - ใช้ `CASE` หรือฟังก์ชันอื่น เช่น `IF` ใน MySQL:
      ```sql
      SELECT first_name,
             IF(department_id = 10, 'Sales', IF(department_id = 20, 'IT', 'Other')) AS dept_name
      FROM employees;
      ```
      
### ตัวอย่าง PL/SQL รวมเงื่อนไข:
```sql
BEGIN
   FOR emp IN (SELECT employee_id, first_name, salary, department_id FROM employees) LOOP
      DBMS_OUTPUT.PUT_LINE(
         'Name: ' || emp.first_name || ', Salary Level: ' ||
         CASE
            WHEN emp.salary < 55000 THEN 'Low'
            WHEN emp.salary <= 60000 THEN 'Medium'
            ELSE 'High'
         END || ', Department: ' ||
         DECODE(emp.department_id, 10, 'Sales', 20, 'IT', 'Other')
      );
   END LOOP;
END;
/
```
- **ผลลัพธ์**:
  ```
  Name: John, Salary Level: Low, Department: Sales
  Name: Jane, Salary Level: Medium, Department: IT
  Name: Alice, Salary Level: Medium, Department: Sales
  Name: Bob, Salary Level: High, Department: IT
  ```
---
