# Honda ECU Flasher (Android, USB-Serial)

โปรเจกต์โครงเริ่มต้นสำหรับแอป Android ที่เชื่อมต่อ ECU มอเตอร์ไซค์ Honda ผ่านสาย
USB-to-Serial (FTDI / CP2102 / CH340 ฯลฯ) ผ่าน USB-OTG แล้วเขียนไฟล์รีแมพ (.bin) ลง ECU

## สิ่งที่ทำให้แล้ว (พร้อมใช้)
- ขอสิทธิ์และเชื่อมต่ออุปกรณ์ USB-serial (ใช้ไลบรารี `usb-serial-for-android`)
- อ่าน baud rate / เปิดพอร์ตสื่อสาร
- หน้าจอเลือกไฟล์ .bin จากเครื่อง
- โครง state machine สำหรับขั้นตอน flash (connect → init → security access →
  erase → write → verify)
- แสดง progress / log บนหน้าจอ
- หน้าจอ **"ดูข้อมูลสด" (Live Data)** — วนอ่านค่าจาก ECU ทุก 200ms และแสดง RPM,
  อุณหภูมิ, ตำแหน่งคันเร่ง, องศาจุดระเบิด (`LiveDataActivity.kt` +
  `LiveDataReader.kt`)

## สิ่งที่ "ยังไม่ใส่" เพราะเป็นข้อมูลเฉพาะรุ่น/ผมยืนยันความถูกต้องไม่ได้
ไฟล์ `HondaEcuProtocol.kt` มีฟังก์ชัน stub (TODO) สำหรับ:
1. **Init sequence** (K-Line 5-baud init หรือ fast-init ตามรุ่น ECU)
2. **Seed-Key security access algorithm** — สูตรคำนวณ key จาก seed ต่างกันในแต่ละ
   generation ของ ECU Honda (PGM-FI รุ่นเก่า vs รุ่นใหม่)
3. **Memory map / erase-write command** — ที่อยู่หน่วยความจำ, ขนาด block, และ
   checksum algorithm ของไฟล์ .bin
4. **Timing/response validation** ระหว่างขั้นตอน flash
5. **Live data request frame + การแปลง response** ใน `LiveDataReader.kt` —
   รูปแบบคำสั่งขอข้อมูล (PID/Mode) และตำแหน่ง byte ที่ตรงกับ RPM/อุณหภูมิ/
   คันเร่ง/องศาจุดระเบิด ต่างกันในแต่ละ ECU เช่นกัน

ค่าพวกนี้ **ห้ามเดา** — ถ้าใส่ผิดแล้วสั่ง erase/write จริง มีโอกาสทำให้ ECU
ใช้งานไม่ได้ถาวร (bricked) ต้องได้มาจากแหล่งที่เชื่อถือได้ เช่น:
- Log การสื่อสารจริงจากเครื่องมือ tuning ที่ถูกกฎหมายและคุณมีสิทธิ์ใช้งาน (เช่น
  ทำ reverse-engineer จาก log ของฮาร์ดแวร์ dongle ที่คุณซื้อมาเอง)
- เอกสาร service/workshop manual ของ Honda สำหรับรุ่นนั้น ๆ
- ชุมชัน/ทีมงานที่ทำ ECU รุ่นนั้นมาก่อนและยืนยันค่าถูกต้องแล้ว

## หมายเหตุด้านกฎหมาย
การรีแมป ECU อาจกระทบมาตรฐานไอเสีย/การรับรองรถ และผิดกฎหมายในบางพื้นที่หากใช้
บนรถที่วิ่งถนนสาธารณะ — ควรตรวจสอบกฎหมายในพื้นที่ของคุณก่อนใช้งานจริง

## ขั้นตอนต่อไป
1. เปิดโปรเจกต์นี้ใน Android Studio
2. เพิ่ม dependency `usb-serial-for-android` ตามที่ระบุใน `app/build.gradle`
3. ใส่ค่า protocol เฉพาะรุ่น ECU ของคุณใน `HondaEcuProtocol.kt`
4. ทดสอบกับ ECU สำรอง/ที่ไม่ได้ติดตั้งในรถจริงก่อนเสมอ
