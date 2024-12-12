# การเชื่อมต่อระหว่าง Jasper Studio บน macOS และ Jasper Server บน Docker (Ubuntu)

การเชื่อมต่อระหว่าง Jasper Studio บน macOS และ Jasper Server บน Docker (Ubuntu) สามารถทำได้โดยการกำหนดค่าการเชื่อมต่อผ่าน URL ของ Jasper Server ที่รันอยู่ใน Docker Container และตรวจสอบให้แน่ใจว่าทั้งสองระบบสามารถสื่อสารกันได้ผ่านเครือข่ายเดียวกัน

## ขั้นตอนการเชื่อมต่อ

### 1. ตรวจสอบและตั้งค่า Jasper Server บน Docker

#### ตรวจสอบ IP Address ของ Docker Container:
ใช้คำสั่งต่อไปนี้ใน Terminal เพื่อดู IP ของ Container:
```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_id>
```
หรือหากใช้ Docker Desktop สามารถดู IP Address ได้จาก UI ของ Docker

#### ตรวจสอบ Port การทำงานของ Jasper Server:
- ค่าเริ่มต้นของ Jasper Server คือ Port `8080`
- หากใช้ `docker run` ตรวจสอบให้แน่ใจว่าคุณได้ Map Port ดังนี้:
```bash
docker run -d -p 8080:8080 <jasper_server_image>
```

#### เปิดการเข้าถึงจากเครื่องภายนอก:
ตรวจสอบว่าคุณสามารถเข้าถึง Jasper Server จาก macOS ได้โดยเปิดเบราว์เซอร์และพิมพ์ URL:
```text
http://<docker_host_ip>:8080/jasperserver
```
หากใช้ Docker Desktop บน macOS, Docker จะ Map เป็น `http://localhost:8080/jasperserver` โดยอัตโนมัติ

### 2. ตั้งค่า Jasper Studio บน macOS

#### เปิด Jasper Studio:
เปิดโปรแกรม Jasper Studio บน macOS

#### เพิ่ม Repository ของ Jasper Server:
1. ไปที่ **Repository Explorer** (ปกติจะอยู่ด้านขวา)
2. คลิกขวา > **Create JasperReports Server Connection**
3. ตั้งค่าดังนี้:
   - **Repository Name:** ตั้งชื่อเช่น `My Jasper Server`
   - **URL:** ใส่ URL ของ Jasper Server เช่น
     ```text
     http://<docker_host_ip>:8080/jasperserver
     ```
     (ถ้ารันบน Docker Desktop ใช้ `http://localhost:8080/jasperserver`)
   - **Username:** ค่าเริ่มต้นคือ `jasperadmin`
   - **Password:** ค่าเริ่มต้นคือ `jasperadmin`
4. กดปุ่ม **Test Connection** เพื่อตรวจสอบว่า Jasper Studio สามารถเชื่อมต่อกับ Jasper Server ได้หรือไม่

#### หากเชื่อมต่อสำเร็จ:
คุณจะเห็นข้อความแจ้งว่าการเชื่อมต่อสำเร็จ

### 3. Troubleshooting

#### กรณีไม่สามารถเชื่อมต่อได้:
- ตรวจสอบว่า Container รันอยู่โดยใช้คำสั่ง:
  ```bash
  docker ps
  ```
- ตรวจสอบว่าค่า URL และ Port ถูกต้อง
- หากอยู่ในเครือข่ายที่มี Firewall ให้ตรวจสอบว่ามีการเปิด Port `8080`

#### กรณี URL เป็น localhost:
หาก macOS ไม่สามารถเข้าถึง Docker Container โดยตรง ให้ใช้ `docker0` bridge network หรือ Map Port ผ่าน Docker Desktop
