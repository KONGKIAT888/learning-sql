# Oracle SQL & PL/SQL Learning Series

## สารบัญ

- [บทที่ 1: รู้จักกับ Oracle Database และ SQL คืออะไร](https://github.com/KONGKIAT888/learning-sql/blob/main/chapter-1.md)
- [บทที่ 2: การสร้างและจัดการตาราง (CREATE, ALTER, DROP)](https://github.com/KONGKIAT888/learning-sql/blob/main/chapter-2.md)
- [บทที่ 3: การเพิ่ม แก้ไข และลบข้อมูล (INSERT, UPDATE, DELETE)](https://github.com/KONGKIAT888/learning-sql/blob/main/chapter-3.md)
- [บทที่ 4: การอ่านข้อมูล (SELECT)](https://github.com/KONGKIAT888/learning-sql/blob/main/chapter-4.md)
- [บทที่ 5: ฟังก์ชันและเงื่อนไข](https://github.com/KONGKIAT888/learning-sql/blob/main/chapter-5.md)
- [บทที่ 6: การใช้ฟังก์ชันกลุ่ม (Aggregate Functions)](https://github.com/KONGKIAT888/learning-sql/blob/main/chapter-6.md)
- [บทที่ 7: การใช้ JOIN เชื่อมโยงหลายตาราง](https://github.com/KONGKIAT888/learning-sql/blob/main/chapter-7.md)
- [บทที่ 8: Subquery และ WITH Clause](https://github.com/KONGKIAT888/learning-sql/blob/main/chapter-8.md)
- [บทที่ 9: View, Index และ Sequence](https://github.com/KONGKIAT888/learning-sql/blob/main/chapter-9.md)
- [บทที่ 10: การจัดการสิทธิ์และการควบคุมธุรกรรม](https://github.com/KONGKIAT888/learning-sql/blob/main/chapter-10.md)

## 📘 บทนำ

เอกสารชุดนี้ถูกจัดทำขึ้นเพื่อเป็นแนวทางการเรียนรู้ภาษา SQL และ PL/SQL สำหรับฐานข้อมูล Oracle โดยเฉพาะ ซึ่งเหมาะสำหรับใช้ภายในทีมงานพัฒนาและดูแลระบบฐานข้อมูล (Database Implementation Team) รวมถึงผู้ที่สนใจเรียนรู้การใช้งาน SQL ในระดับองค์กร

เนื้อหาในเอกสารชุดนี้ถูกเรียบเรียงโดยเน้นความเข้าใจง่าย ค่อยเป็นค่อยไป และอิงจากประสบการณ์ในการใช้งานจริงในระบบองค์กร เพื่อให้สามารถนำไปประยุกต์ใช้ได้ทั้งในการพัฒนาแอปพลิเคชัน การเขียน Query เพื่อวิเคราะห์ข้อมูล หรือการเขียนโปรแกรมที่ใช้ภาษา PL/SQL ในระดับ Production

---

## 🎯 วัตถุประสงค์

- เพื่อให้ทีมงานเข้าใจหลักการใช้งาน SQL และ PL/SQL บน Oracle Database อย่างถูกต้อง
- เพื่อช่วยให้สามารถเขียน Query และ PL/SQL Script ได้อย่างมีประสิทธิภาพ
- เพื่อปูพื้นฐานที่มั่นคงก่อนเข้าสู่การพัฒนาและดูแลระบบฐานข้อมูลขนาดใหญ่
- เพื่อใช้เป็นคู่มืออ้างอิงภายในองค์กร

---

## 👨‍💼 กลุ่มเป้าหมาย

- **ทีมพัฒนา (Developers)** ที่ต้องเขียน SQL/PLSQL เพื่อทำงานร่วมกับ Oracle Database
- **ทีม Implement / Technical Consultant** ที่ต้องติดตั้งหรือดูแลระบบที่ใช้ฐานข้อมูล Oracle
- **นักวิเคราะห์ข้อมูล (Data Analyst)** ที่ต้องดึงข้อมูลเพื่อวิเคราะห์หรือจัดทำรายงาน
- **ผู้สนใจทั่วไป** ที่ต้องการปูพื้นฐานการเขียน SQL และเข้าใจการจัดการฐานข้อมูลแบบ RDBMS

---

## 🧩 โครงสร้างเนื้อหา

เอกสารแบ่งออกเป็น 10 บทเรียนหลัก ดังนี้:

1. **บทที่ 1:** รู้จักกับ Oracle Database และ SQL คืออะไร  
2. **บทที่ 2:** การสร้างและจัดการตาราง (CREATE, ALTER, DROP)  
3. **บทที่ 3:** การเพิ่ม แก้ไข และลบข้อมูล (INSERT, UPDATE, DELETE)  
4. **บทที่ 4:** การอ่านข้อมูล (SELECT) และคำสั่งพื้นฐาน  
5. **บทที่ 5:** ฟังก์ชันและการเขียนเงื่อนไขใน SQL  
6. **บทที่ 6:** การใช้ฟังก์ชันกลุ่มและการจัดกลุ่มข้อมูล (GROUP BY, HAVING)  
7. **บทที่ 7:** การเชื่อมโยงข้อมูลจากหลายตาราง (JOIN)  
8. **บทที่ 8:** การใช้ Subquery และ WITH Clause  
9. **บทที่ 9:** การสร้าง View, Index และ Sequence  
10. **บทที่ 10:** การควบคุมธุรกรรมและการจัดการสิทธิ์ผู้ใช้งาน  

> แต่ละบทเรียนจะประกอบด้วยคำอธิบายหลักการ ตัวอย่างโค้ด SQL และแนวทางการนำไปใช้งานจริง

---

## 💡 วิธีการใช้งานเอกสาร

- ศึกษาทีละบทตามลำดับจากพื้นฐานไปสู่ขั้นสูง
- สามารถคัดลอกตัวอย่าง SQL ไปทดลองใช้งานใน Oracle SQL Developer หรือ Oracle APEX ได้ทันที
- เอกสารนี้สามารถปรับปรุงและต่อยอดได้ตามความเหมาะสมกับโครงการในองค์กร
- ใช้ประกอบการอบรมภายใน หรือ onboarding สำหรับพนักงานใหม่

---

## 🛠 เครื่องมือที่แนะนำในการฝึก

- DBeaver Community
- Navicat Premium Lite 17
- Oracle Database Express Edition (XE)
- หรือโปรแกรมจัดการฐานข้อมูลอื่นๆ
---

## 📌 หมายเหตุเพิ่มเติม

- เอกสารนี้เป็นเวอร์ชันเริ่มต้น หากมีการปรับปรุง/เพิ่มเติมจะมีการอัปเดตเวอร์ชันในหัวเอกสาร
- เนื้อหานี้เน้นการใช้งาน Oracle SQL และ PL/SQL โดยเฉพาะ ไม่ครอบคลุม SQL ใน RDBMS อื่น (เช่น MySQL, PostgreSQL)
- หากมีคำถามหรือข้อเสนอแนะเพิ่มเติม สามารถติดต่อทีมผู้จัดทำเอกสารเพื่อแลกเปลี่ยนความรู้ร่วมกันได้

---

