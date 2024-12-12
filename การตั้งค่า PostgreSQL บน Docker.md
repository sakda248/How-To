# การตั้งค่า PostgreSQL บน Docker พร้อมข้อมูลตัวอย่างและการเชื่อมต่อจากภายนอก

## ขั้นตอนที่ 1: ติดตั้ง Docker บน macOS
1. ตรวจสอบว่า Docker ได้ติดตั้งแล้วหรือไม่ หากยังไม่ได้ติดตั้ง ให้ดาวน์โหลดและติดตั้ง Docker Desktop จาก [Docker’s official website](https://www.docker.com/products/docker-desktop)

2. ตรวจสอบการติดตั้ง Docker:
   ```bash
   docker --version
   ```

---

## ขั้นตอนที่ 2: ดาวน์โหลด Docker Image ของ PostgreSQL
1. ดึงภาพ Docker ของ PostgreSQL:
   ```bash
   docker pull postgres:latest
   ```

---

## ขั้นตอนที่ 3: เริ่มต้นคอนเทนเนอร์ PostgreSQL
1. รันคอนเทนเนอร์ PostgreSQL พร้อมการเชื่อมต่อพอร์ต 5432:
   ```bash
   docker run -d \
     --name postgres_sample \
     -e POSTGRES_USER=sampleuser \
     -e POSTGRES_PASSWORD=samplepass \
     -e POSTGRES_DB=sampledb \
     -p 5432:5432 \
     postgres:latest
   ```

2. ตรวจสอบว่าคอนเทนเนอร์กำลังทำงาน:
   ```bash
   docker ps
   ```

---

## ขั้นตอนที่ 4: ดาวน์โหลดข้อมูลตัวอย่าง
1. ดาวน์โหลดข้อมูลตัวอย่าง (Pagila):
   ```bash
   wget https://github.com/devrimgunduz/pagila/archive/refs/heads/master.zip
   ```

2. แตกไฟล์ข้อมูลตัวอย่าง:
   ```bash
   unzip master.zip
   cd pagila-master
   ```

---

## ขั้นตอนที่ 5: นำเข้าข้อมูลตัวอย่างไปยัง PostgreSQL
1. คัดลอกไฟล์ SQL ไปยังคอนเทนเนอร์ PostgreSQL:
   ```bash
   docker cp pagila-schema.sql postgres_sample:/pagila-schema.sql
   docker cp pagila-data.sql postgres_sample:/pagila-data.sql
   ```

2. รันสคริปต์เพื่อโหลด Schema และข้อมูล:
   ```bash
   docker exec -it postgres_sample psql -U sampleuser -d sampledb -f /pagila-schema.sql
   docker exec -it postgres_sample psql -U sampleuser -d sampledb -f /pagila-data.sql
   ```

---

## ขั้นตอนที่ 6: เปิดใช้งานการเชื่อมต่อจากภายนอก

### การตั้งค่า PostgreSQL ให้เชื่อมต่อจากภายนอก:
1. แก้ไขไฟล์ `postgresql.conf`:
   ```bash
   docker exec -it postgres_sample bash
   nano /var/lib/postgresql/data/postgresql.conf
   ```
   เพิ่มหรืออัปเดตบรรทัดดังนี้:
   ```text
   listen_addresses = '*'
   ```

2. แก้ไขไฟล์ `pg_hba.conf`:
   ```bash
   nano /var/lib/postgresql/data/pg_hba.conf
   ```
   เพิ่มบรรทัดดังนี้:
   ```text
   host    all             all             0.0.0.0/0               md5
   ```

3. รีสตาร์ทคอนเทนเนอร์:
   ```bash
   docker restart postgres_sample
   ```

---

## ขั้นตอนที่ 7: ทดสอบการเชื่อมต่อจากภายนอก

1. ทดสอบการเชื่อมต่อจาก macOS:
   ```bash
   psql -h localhost -p 5432 -U sampleuser -d sampledb
   ```

2. ใช้ข้อมูลการเชื่อมต่อดังนี้:
   - **Host**: `localhost`
   - **Port**: `5432`
   - **Username**: `sampleuser`
   - **Password**: `samplepass`
   - **Database**: `sampledb`

---

## ขั้นตอนที่ 8: การเชื่อมต่อผ่าน GUI Tools
1. เปิดโปรแกรม GUI เช่น **pgAdmin** หรือ **DBeaver**
2. กรอกข้อมูลการเชื่อมต่อ:
   - **Host**: `localhost`
   - **Port**: `5432`
   - **Username**: `sampleuser`
   - **Password**: `samplepass`
   - **Database**: `sampledb`

---

## ขั้นตอนที่ 9: ตรวจสอบข้อมูลตัวอย่าง
1. เข้าสู่ฐานข้อมูลและรันคำสั่งเพื่อตรวจสอบข้อมูล:
   ```sql
   SELECT * FROM actor LIMIT 5;
   ```

---

## ขั้นตอนที่ 10: การแก้ไขปัญหา

### ตรวจสอบบันทึกของ Docker
```bash
   docker logs postgres_sample
```

### ตรวจสอบว่าพอร์ตเปิดอยู่
```bash
   nc -zv localhost 5432
