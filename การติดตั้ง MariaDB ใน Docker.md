# การติดตั้ง MariaDB บน Docker พร้อมการตั้งค่าฐานข้อมูลตัวอย่างและรองรับการเชื่อมต่อภายนอก

## ขั้นตอนที่ 1: ดาวน์โหลด Docker Image ของ MariaDB
ใช้คำสั่งต่อไปนี้เพื่อดาวน์โหลด Docker Image ของ MariaDB:
```bash
docker pull mariadb:latest
```

---

## ขั้นตอนที่ 2: รันคอนเทนเนอร์ MariaDB
ใช้คำสั่งต่อไปนี้เพื่อรัน MariaDB คอนเทนเนอร์:
```bash
docker run -d \
  --name mariadb_sample \
  -e MARIADB_ROOT_PASSWORD=samplepass \
  -e MARIADB_DATABASE=sampledb \
  -e MARIADB_USER=sampleuser \
  -e MARIADB_PASSWORD=samplepass \
  -p 3306:3306 \
  mariadb:latest
```
- **`-e MARIADB_ROOT_PASSWORD`**: ตั้งรหัสผ่านสำหรับผู้ใช้ root
- **`-e MARIADB_DATABASE`**: สร้างฐานข้อมูลชื่อ `sampledb`
- **`-e MARIADB_USER`**: สร้างผู้ใช้ `sampleuser`
- **`-e MARIADB_PASSWORD`**: ตั้งรหัสผ่านสำหรับผู้ใช้ `sampleuser`
- **`-p 3306:3306`**: เปิดพอร์ต 3306 สำหรับการเชื่อมต่อ

ตรวจสอบว่าคอนเทนเนอร์กำลังทำงาน:
```bash
docker ps
```

---

## ขั้นตอนที่ 3: ดาวน์โหลดฐานข้อมูลตัวอย่าง
### ตัวอย่างฐานข้อมูล Sakila
1. ดาวน์โหลดไฟล์ฐานข้อมูล Sakila:
   ```bash
   wget [https://downloads.mariadb.com/MariaDB/mariadb-sakila-db/sakila-schema.sql](https://github.com/sakda248/How-To/blob/801f5ac341adef8dca151a6a971c910c8d0a956e/sakila-schema.sql)
   wget https://downloads.mariadb.com/MariaDB/mariadb-sakila-db/sakila-data.sql](https://github.com/sakda248/How-To/blob/1e33059d115005d1f511ebae190acd9aedf67c7b/sakila-data.sql)
   ```

---

## ขั้นตอนที่ 4: คัดลอกไฟล์ SQL ไปยังคอนเทนเนอร์
ใช้คำสั่งต่อไปนี้เพื่อคัดลอกไฟล์ไปยังคอนเทนเนอร์:
```bash
docker cp sakila-schema.sql mariadb_sample:/sakila-schema.sql
docker cp sakila-data.sql mariadb_sample:/sakila-data.sql
```

---

## ขั้นตอนที่ 5: โหลดข้อมูลตัวอย่างเข้า MariaDB
1. เข้าสู่คอนเทนเนอร์:
   ```bash
   docker exec -it mariadb_sample bash
   ```

2. เข้าสู่ MariaDB CLI:
   ```bash
   mariadb -u root -p
   ```
   ใส่รหัสผ่าน `samplepass`

3. รันไฟล์ `sakila-schema.sql` เพื่อสร้างโครงสร้างฐานข้อมูล:
   ```sql
   SOURCE /sakila-schema.sql;
   ```

4. รันไฟล์ `sakila-data.sql` เพื่อเพิ่มข้อมูลตัวอย่าง:
   ```sql
   SOURCE /sakila-data.sql;
   ```

5. ตรวจสอบข้อมูล:
   ```sql
   SHOW DATABASES;
   USE sakila;
   SHOW TABLES;
   SELECT * FROM actor LIMIT 5;
   ```

6. ออกจาก MariaDB CLI:
   ```sql
   EXIT;
   ```

7. ออกจากคอนเทนเนอร์:
   ```bash
   exit
   ```

---

## ขั้นตอนที่ 6: เปิดใช้งานการเชื่อมต่อภายนอก
1. เข้าสู่คอนเทนเนอร์:
   ```bash
   docker exec -it mariadb_sample bash
   ```

2. แก้ไขไฟล์การตั้งค่า MariaDB:
   ```bash
   vim /etc/mysql/my.cnf
   ```
   เพิ่มหรือแก้ไขบรรทัดต่อไปนี้:
   ```ini
   bind-address = 0.0.0.0
   ```

3. รีสตาร์ทบริการ MariaDB ภายในคอนเทนเนอร์:
   ```bash
   service mysql restart
   ```

4. ออกจากคอนเทนเนอร์:
   ```bash
   exit
   ```

---

## ขั้นตอนที่ 7: ให้สิทธิ์สำหรับการเข้าถึงจากภายนอก
เข้าสู่ MariaDB CLI:
```bash
docker exec -it mariadb_sample mariadb -u root -p
```

รันคำสั่งต่อไปนี้เพื่อให้สิทธิ์การเข้าถึง:
```sql
GRANT ALL PRIVILEGES ON *.* TO 'sampleuser'@'%' IDENTIFIED BY 'samplepass';
FLUSH PRIVILEGES;
EXIT;
```

---

## ขั้นตอนที่ 8: ทดสอบการเชื่อมต่อภายนอก
จากเครื่องโฮสต์หรือเครื่องอื่น ทดสอบการเชื่อมต่อ:
```bash
mariadb -h 127.0.0.1 -P 3306 -u sampleuser -p
```

รันคำสั่งเพื่อตรวจสอบข้อมูล:
```sql
USE sakila;
SELECT * FROM film LIMIT 5;
```

---

## ขั้นตอนที่ 9: ใช้เครื่องมือ GUI
สามารถใช้เครื่องมือเช่น **DBeaver**, **HeidiSQL** หรือ **phpMyAdmin** เพื่อเชื่อมต่อ:
- **Host**: `127.0.0.1`
- **Port**: `3306`
- **Username**: `sampleuser`
- **Password**: `samplepass`

---

## การแก้ปัญหา
1. **ตรวจสอบล็อก**:
   ```bash
   docker logs mariadb_sample
   ```

2. **ตรวจสอบพอร์ต**:
   ```bash
   nc -zv 127.0.0.1 3306
   ```

3. **ตั้งค่ากำแพงไฟร์วอลล์**:
   เปิดพอร์ต `3306` บนระบบของคุณ
