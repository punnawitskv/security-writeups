# AS04: Cryptographic Failures

Target: http://10.x.x.x:5004   
Flag: `THM{CRYPTO_FAILURE_H4RDCOD3D_K3Y}`

---

## 1. บทนำ

แลปนี้เป็นการฝึกหาช่องโหว่ประเภท `Cryptographic Failures` ซึ่งเกิดจากการใช้ระบบเข้ารหัสไม่ถูกต้อง และมีการเปิดเผย key ทำให้ผู้โจมตีสามารถถอดรหัสข้อมูลที่ควรเป็นความลับได้

---

## 2. การสำรวจหน้าเว็บ

ตรวจสอบเว็บไซต์ด้วย `curl` ดังนี้

```bash
curl http://10.48.178.175:5004
```

พบว่ามีการแสดง `encrypted document` และมีการโหลดไฟล์ JavaScript ชื่อ `decrypt.js` จากฝั่ง client

```bash
<body>
    <div class="container">
        <h1>\U0001f512 Secure Document Viewer</h1>
        <p>This confidential document is encrypted for security. Only authorized personnel can access the decryption key.</p>
        <p><em>Note: The decryption feature is currently unavailable. Contact your administrator for access.</em></p>
        
        <div class="encrypted-box" id="encrypted">
            <strong>Encrypted Document:</strong><br>
            Nzd42HZGgUIUlpILZRv0jeIXp1WtCErwR+j/w/lnKbmug31opX0BWy+pwK92rkhjwdf94mgHfLtF26X6B3pe2fhHXzIGnnvVruH7683KwvzZ6+QKybFWaedAEtknYkhe
        </div>
    </div>
    
    <script src="/static/js/decrypt.js"></script>
</body>
```

---

## 3. การตรวจสอบไฟล์ decrypt.js

จากการเปิดดูไฟล์ `decrypt.js` ด้วย `curl` ดังนี้

```bash
curl http://10.48.178.175:5004/static/js/decrypt.js
```

พบว่ามีการกำหนดค่า secret key
และโหมดการเข้ารหัสไว้ในไฟล์ JavaScript ดังนี้

```bash
// Configuration
const SECRET_KEY = "my-secret-key-16";
const ENCRYPTION_MODE = "ECB";
const KEY_SIZE = 128;
```

ซึ่งถือเป็นความผิดพลาดด้านความปลอดภัยเนื่องจาก key ไม่ควรถูกเก็บไว้ฝั่ง client และโหมด ECB เป็นโหมดที่ไม่ปลอดภัย

---

## 4. การถอดรหัสข้อมูล

ถอดรหัสข้อมูลด้วย `CyberChef` จากข้อมูลดังนี้

Recipe:
1. `From Base64`
2. `AES Decrypt`
    - Key (`UTF8`): `my-secret-key-16`
    - Mode: `ECB`
    - Input: `Raw`
    - Output: `Raw`

Input:
`Nzd42HZGgUIUlpILZRv0jeIXp1WtCErwR+j/w/lnKbmug31opX0BWy+pwK92rkhjwdf94mgHfLtF26X6B3pe2fhHXzIGnnvVruH7683KwvzZ6+QKybFWaedAEtknYkhe`

```bash
CONFIDENTIAL: The admin password is 'admin123'. Flag: THM{CRYPTO_FAILURE_H4RDCOD3D_K3Y}
```

---

## 5. Flag ที่ได้

`THM{CRYPTO_FAILURE_H4RDCOD3D_K3Y}`

---

## 6. สิ่งที่ได้เรียนรู้

- ไม่ควร hard-code secret หรือ key ในฝั่ง client
- การใช้โหมด ECB ทำให้การเข้ารหัสไม่ปลอดภัย
- การเข้ารหัสจะไม่มีประโยชน์ หากผู้โจมตีสามารถเข้าถึง key ได้