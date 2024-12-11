## ติดตั้ง JasperReports Server 9.0.0 ใน Docker ด้วย Ubuntu 24

### แนะนำเอกสาร
เอกสารนี้แสดงขั้นตอนการติดตั้ง JasperReports Server 9.0.0 บน Docker โดยใช้ Ubuntu เป็น Base Image และอธิบายการตั้งค่าที่จำเป็น

---

### ขั้นตอนขั้นแรกของ Docker Container

```bash
docker run -it --platform linux/amd64 --name jasper_server -p 8080:8080 ubuntu:latest
```
- สร้าง Docker Container และตั้งชื่อ container ว่า `jasper_server`
- พร้อม port 8080 

---

### ติดตั้งระบบใน Docker Container

1. อัพเดตและติดตั้งเครื่องมือที่จำเป็น

    ```bash
    apt-get update
    apt-get install -y openjdk-8-jdk libc6 wget curl lsb-release
    ```
    - ติดตั้ง Java (OpenJDK 8), libc6, curl, และ lsb-release

2. ตรวจสอบเวอร์ชันของระบบปฏิบัติการและ Java:
    ```bash
    lsb_release -a
    java -version
    uname -m
    ```

---

### ติดตั้ง Google Chrome

1. ดาวน์โหลด Google Chrome:
    ```bash
    wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
    ```

2. ติดตั้ง Google Chrome:
    ```bash
    dpkg -i google-chrome-stable_current_amd64.deb
    apt-get install -f
    ```
    - คำสั่ง `apt-get install -f`

---

### ติดตั้ง JasperReports Server

1. คัดลอกไฟล์ `.run` จาก Host ไปยัง Container:
    ```bash
    docker cp JasperReports-Server_9.0.0_linux_x86_64.run d1064e6c6f31:/opt/install_jasperserver.run
    ```

2. เปลี่ยนโฟลเดอร์และตั้งค่าไฟล์ให้รันได้:
    ```bash
    cd /opt/
    chmod +x install_jasperserver.run
    ```

3. รันไฟล์ติดตั้ง:
    ```bash
    ./install_jasperserver.run
    ```
    - หากมีการร้องขอ Chrome Executable ให้ใส่ `/usr/bin/google-chrome`

---

### เริ่ม JasperReports Server

1. ไปยังโฟลเดอร์ติดตั้ง JasperReports Server:
    ```bash
    cd jasperreports-server-9.0.0/
    ```

2. เริ่มต้น JasperReports Server:
    ```bash
    ./ctlscript.sh start
    ```

3. ตรวจสอบสถานะของ Server:
    ```bash
    ./ctlscript.sh status
    ```

4. ทดสอบ Server ด้วย cURL:
    ```bash
    curl http://localhost:8080/jasperserver-pro
    ```

---

### เข้าถึง JasperReports Server
1. เปิดเบราว์เซอร์และเข้า:
    ```
    http://localhost:8080/jasperserver-pro
    ```

2. ใช้ข้อมูลเข้าสู่ระบบต่อไปนี้:
    - **User:** jasperadmin
    - **Password:** jasperadmin

---

### สรุปการทดสอบประเด็น
- ตรวจสอบว่า Container ทำงานอยู่:
    ```bash
    docker ps
    ```
- หากเกิดปัญหา ให้ตรวจสอบ Log:
    ```bash
    tail -f /opt/jasperreports-server-9.0.0/apache-tomcat/logs/catalina.out
    ```

