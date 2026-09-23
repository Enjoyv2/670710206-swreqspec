# Tasks: จองคิวตรวจสุขภาพ (Booking)

- Feature: จองคิวตรวจสุขภาพ (Booking)
- Spec ID: SPEC-BKG-001
- อ้างอิง: [plan.md](./plan.md)
- วันที่: 2569-09-23

มีทั้งหมด 14 tasks โดย 5 tasks ต้องรอคำตอบ Open Question Q-02 เรื่องรูปแบบและวิธีออกหมายเลขคิว  
งานเรียงตามการพึ่งพา ตั้งแต่โมเดลข้อมูลและบริการหลังบ้าน ไปจนถึงหน้าจอและการต่อ API จริง โดยยังไม่เริ่มเขียนโค้ด

### T-01 สร้างตารางและ migration
- รองรับ: CON-TECH-01, DOM-PDPA-01, IF-HIS-01, FR-BKG-01, FR-BKG-02, FR-BKG-04, FR-BKG-06
- ตรวจด้วย: ไม่มี AC ตรง ๆ เป็นงานพื้นฐานของ T-03, T-05 และ T-09
- ไฟล์ที่แตะ: `backend/app/db/models.py`, `backend/app/db/session.py`, `backend/app/db/migrations/001_init.py`, `backend/tests/conftest.py`
- ต้องทำหลัง: ไม่มี
- เสร็จเมื่อ: migration สร้างตาราง `slots`, `bookings` และ `audit_logs` ได้ และตาราง `bookings` ไม่มีเลขบัตรประชาชน
- สถานะ: พร้อมทำ

### T-02 บังคับยืนยันตัวตนและตั้งค่าการรับส่งข้อมูล
- รองรับ: IF-IDP-01, NFR-SEC-01
- ตรวจด้วย: ไม่มี AC ตรง ๆ เป็นงานพื้นฐานของ endpoint ทุกตัว
- ไฟล์ที่แตะ: `backend/app/auth/idp.py`, `backend/app/main.py`, `backend/app/config.py`, `backend/tests/test_auth.py`
- ต้องทำหลัง: T-01
- เสร็จเมื่อ: endpoint ที่เข้าถึงข้อมูลผู้รับบริการตรวจผลยืนยันตัวตนก่อน และการตั้งค่า production บังคับใช้ TLS 1.2 ขึ้นไป
- สถานะ: พร้อมทำ

### T-03 สร้างบริการค้นหา HN จาก HIS
- รองรับ: IF-HIS-01, IF-IDP-01
- ตรวจด้วย: ไม่มี AC ตรง ๆ เป็นงานพื้นฐานของ T-05 และ T-09
- ไฟล์ที่แตะ: `backend/app/his/client.py`, `backend/app/main.py`, `backend/tests/test_his_lookup.py`
- ต้องทำหลัง: T-02
- เสร็จเมื่อ: `GET /patients/lookup` ส่งเลขบัตรต่อให้ HIS คืน HN และไม่บันทึกเลขบัตรลงฐานข้อมูลการจอง
- สถานะ: พร้อมทำ

### T-04 สร้างบริการและ API ช่วงเวลาว่าง
- รองรับ: FR-BKG-01, FR-BKG-06, CON-TECH-01
- ตรวจด้วย: ไม่มี AC ตรง ๆ เป็นงานพื้นฐานของ T-11 และ T-14
- ไฟล์ที่แตะ: `backend/app/slots/service.py`, `backend/app/slots/router.py`, `backend/app/main.py`, `backend/tests/test_slots.py`
- ต้องทำหลัง: T-01 และ T-02
- เสร็จเมื่อ: `GET /slots` คืนช่วงเวลาภายใน 30 วันพร้อมที่นั่งคงเหลือ และโหลดข้อมูลใหม่เมื่อเปลี่ยนแพ็กเกจ
- สถานะ: พร้อมทำ

### T-05 วัดประสิทธิภาพการค้นหาช่วงเวลา
- รองรับ: NFR-PERF-01, FR-BKG-01
- ตรวจด้วย: AC-BKG-05
- ไฟล์ที่แตะ: `backend/tests/test_AC_BKG_05.py`
- ต้องทำหลัง: T-04
- เสร็จเมื่อ: การทดสอบจำลองผู้ใช้พร้อมกัน 200 คนรายงานค่า p95 ของ `GET /slots` ไม่เกิน 2 วินาที
- สถานะ: พร้อมทำ

### T-06 สร้างการยืนยันการจองและตัดที่นั่ง
- รองรับ: FR-BKG-04, CON-TECH-01, IF-HIS-01
- ตรวจด้วย: AC-BKG-01
- ไฟล์ที่แตะ: `backend/app/booking/service.py`, `backend/app/booking/router.py`, `backend/app/main.py`, `backend/tests/test_AC_BKG_01.py`
- ต้องทำหลัง: T-01, T-02, T-03 และ T-04
- เสร็จเมื่อ: การยืนยันที่นั่งสุดท้ายบันทึก booking และทำให้ `remaining` เป็น 0 พร้อมคืนผลการจองตามสัญญา API
- สถานะ: รอ Q-02

### T-07 ป้องกันการจองซ้ำในวันเดียวกัน
- รองรับ: FR-BKG-02
- ตรวจด้วย: AC-BKG-02
- ไฟล์ที่แตะ: `backend/app/booking/service.py`, `backend/tests/test_AC_BKG_02.py`
- ต้องทำหลัง: T-06
- เสร็จเมื่อ: ผู้รับบริการที่มีคิวที่ยังไม่ได้ใช้ในวันเดียวกันได้รับการปฏิเสธและหมายเลขคิวเดิมตามสัญญา API
- สถานะ: รอ Q-02

### T-08 เสนอช่วงเวลาใกล้เคียงเมื่อเต็ม
- รองรับ: FR-BKG-03
- ตรวจด้วย: AC-BKG-03
- ไฟล์ที่แตะ: `backend/app/slots/service.py`, `backend/app/booking/service.py`, `backend/app/booking/router.py`, `backend/tests/test_AC_BKG_03.py`
- ต้องทำหลัง: T-04 และ T-06
- เสร็จเมื่อ: การจองที่เต็มตอบสถานะ 409 พร้อม 3 ช่วงที่ใกล้ที่สุดภายในวันเดียวกันและวันถัดไป และไม่สร้าง booking ซ้อน
- สถานะ: พร้อมทำ

### T-09 จัดการคิวส่งข้อความและการส่งซ้ำ
- รองรับ: FR-BKG-05, IF-NOT-01, NFR-REL-02
- ตรวจด้วย: AC-BKG-04
- ไฟล์ที่แตะ: `backend/app/notify/queue.py`, `backend/app/booking/service.py`, `backend/app/booking/router.py`, `backend/tests/test_AC_BKG_04.py`
- ต้องทำหลัง: T-06
- เสร็จเมื่อ: เมื่อระบบแจ้งเตือนไม่ตอบสนอง booking ยังถูกบันทึก มีงานในคิวส่งซ้ำ และกำหนดส่งครั้งแรกภายใน 5 นาที
- สถานะ: รอ Q-02

### T-10 บันทึก audit log ของการเข้าถึงข้อมูล
- รองรับ: DOM-PDPA-01, IF-IDP-01
- ตรวจด้วย: AC-BKG-06
- ไฟล์ที่แตะ: `backend/app/audit/middleware.py`, `backend/app/db/models.py`, `backend/app/main.py`, `backend/tests/test_AC_BKG_06.py`
- ต้องทำหลัง: T-01 และ T-02
- เสร็จเมื่อ: การเปิดดูข้อมูลการจองสร้าง audit log ที่มีผู้เข้าถึง เวลา HN และเก็บข้อมูลได้ไม่น้อยกว่า 1 ปี
- สถานะ: พร้อมทำ

### T-11 สร้างหน้าเลือกแพ็กเกจและช่วงเวลา
- รองรับ: FR-BKG-01, FR-BKG-06
- ตรวจด้วย: ไม่มี AC ตรง ๆ เป็นงานพื้นฐานของ T-14
- ไฟล์ที่แตะ: `frontend/src/pages/SlotPicker.jsx`, `frontend/src/api/client.js`, `frontend/src/App.jsx`, `frontend/src/__tests__/SlotPicker.test.jsx`
- ต้องทำหลัง: ไม่มี
- เสร็จเมื่อ: หน้าจอใช้ API จำลองแสดงวัน ช่วงเวลา ที่นั่งคงเหลือ และโหลดช่วงเวลาใหม่เมื่อเปลี่ยนแพ็กเกจ
- สถานะ: พร้อมทำ

### T-12 สร้างหน้ายืนยันและแสดงทางเลือกเมื่อเต็ม
- รองรับ: FR-BKG-03, FR-BKG-04
- ตรวจด้วย: AC-BKG-03
- ไฟล์ที่แตะ: `frontend/src/pages/ConfirmBooking.jsx`, `frontend/src/App.jsx`, `frontend/src/__tests__/AC-BKG-03.test.jsx`
- ต้องทำหลัง: T-11
- เสร็จเมื่อ: API จำลองตอบ 409 แล้วหน้าจอแสดงข้อความ "ช่วงเวลาเต็ม" และช่วงเวลาทางเลือก 3 รายการ
- สถานะ: พร้อมทำ

### T-13 สร้างหน้าแสดงผลการจอง
- รองรับ: FR-BKG-04, FR-BKG-05
- ตรวจด้วย: ไม่มี AC ตรง ๆ เป็นงานพื้นฐานของ T-14
- ไฟล์ที่แตะ: `frontend/src/pages/BookingResult.jsx`, `frontend/src/App.jsx`, `frontend/src/__tests__/BookingResult.test.jsx`
- ต้องทำหลัง: T-11
- เสร็จเมื่อ: หน้าจอ API จำลองแสดงผลการจองและหมายเลขคิวแม้สถานะการส่งข้อความจะล้มเหลว
- สถานะ: รอ Q-02

### T-14 ต่อหน้าจอกับ API จริงและตรวจ flow รวม
- รองรับ: FR-BKG-01, FR-BKG-02, FR-BKG-03, FR-BKG-04, FR-BKG-05, FR-BKG-06, IF-IDP-01
- ตรวจด้วย: AC-BKG-01, AC-BKG-02, AC-BKG-03, AC-BKG-04
- ไฟล์ที่แตะ: `frontend/src/api/client.js`, `frontend/src/App.jsx`, `frontend/src/pages/SlotPicker.jsx`, `frontend/src/pages/ConfirmBooking.jsx`, `frontend/src/pages/BookingResult.jsx`, `frontend/src/__tests__/flow.test.jsx`
- ต้องทำหลัง: T-07, T-08, T-09, T-11, T-12 และ T-13
- เสร็จเมื่อ: flow เลือกช่วงเวลา ยืนยัน จัดการ error และแสดงผลทำงานผ่าน API จริงตามสัญญาใน plan.md
- สถานะ: รอ Q-02

## ตารางตรวจความครบของ Acceptance Criteria

| AC ID | task ที่ตรวจ AC นี้ |
|---|---|
| AC-BKG-01 | T-06, T-14 |
| AC-BKG-02 | T-07, T-14 |
| AC-BKG-03 | T-08, T-12, T-14 |
| AC-BKG-04 | T-09, T-14 |
| AC-BKG-05 | T-05 |
| AC-BKG-06 | T-10 |

## ตารางตรวจความครบของ Constraints

| Constraint ID | task ที่ทำให้เป็นจริง |
|---|---|
| CON-TECH-01 | T-01, T-06 |
| DOM-PDPA-01 | T-01, T-10 |
| IF-IDP-01 | T-02, T-03, T-10, T-14 |
| IF-HIS-01 | T-01, T-03, T-06 |
| IF-NOT-01 | T-09 |

## สิ่งที่ยังไม่ทำ

- Q-02: หมายเลขคิวรีเซ็ตรายวันหรือนับต่อเนื่อง และมีรูปแบบอย่างไร (เช่น `A001`) ต้องถามเจ้าหน้าที่เวชระเบียน
- งานที่รอ Q-02: T-06, T-07, T-09, T-13 และ T-14
- จนกว่าจะได้คำตอบ ห้ามเดาวิธีออกเลขคิว รูปแบบเลขคิว หรือพฤติกรรมการแสดงเลขคิว
