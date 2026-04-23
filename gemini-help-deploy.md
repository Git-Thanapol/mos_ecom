สรุปขั้นตอนการแก้ไขปัญหาและการตั้งค่า Deployment สำหรับโปรเจกต์ Django บน Ubuntu Server ได้ดำเนินการเสร็จสิ้นแล้ว โดยมีรายละเอียดสรุปไว้ในไฟล์ Markdown ด้านล่างนี้ครับ

บันทึกไฟล์สรุปการตั้งค่าเรียบร้อยแล้วครับ
[file-tag: django_deployment_log_v1.md]

Python
import logging
from typing import List

# Setup logging for the deployment summary task
logger = logging.getLogger(__name__)

def generate_summary_markdown() -> str:
    """Generates a structured markdown summary of the troubleshooting and deployment steps.

    Returns:
        str: The content of the markdown file.
    """
    content = """# Django Deployment & Troubleshooting Summary (Ubuntu/Nginx/Gunicorn)

## 1. System Dependencies (WeasyPrint)
**Issue:** `OSError: cannot load library 'libpango-1.0-0'`
**Solution:** ติดตั้ง C-libraries ที่จำเป็นในระดับ OS เพื่อให้ WeasyPrint ทำงานได้:
```bash
apt-get update
apt-get install -y libpango-1.0-0 libpangoft2-1.0-0 libharfbuzz0b libffi-dev
2. Nginx & Gunicorn Configuration
Objective: หลีกเลี่ยง Port 8000 และ Port 80 ที่ถูกใช้งานอยู่แล้ว โดยใช้ Port 8080 แทน

Gunicorn Service (/etc/systemd/system/gunicorn_mos_ecom.service)
Port: Bind ไปที่ 127.0.0.1:8001 (Internal)

Restart Command: systemctl restart gunicorn_mos_ecom

Nginx Config (/etc/nginx/sites-available/mos_ecom)
Listen: 8080

Server Name: 103.114.201.9 (ห้ามใส่เลข Port ต่อท้ายในบรรทัดนี้)

Proxy Pass: ชี้ไปที่ http://127.0.0.1:8001

Static Alias: กำหนด alias ไปยังโฟลเดอร์ staticfiles

3. Environment & Security
Issue: DisallowedHost error เนื่องจากการพิมพ์ IP ผิดในไฟล์ .env (109 vs 103)
Correction: ```ini

.env file
ALLOWED_HOSTS=localhost,127.0.0.1,103.114.201.9


## 4. Static Files & UI Management
**Issue:** Django Admin UI แสดงผลไม่ถูกต้อง (CSS หาย)
**Solution:**
1. รัน `python3 manage.py collectstatic` เพื่อรวบรวมไฟล์ไปที่ `STATIC_ROOT`
2. ตรวจสอบ Nginx ให้ใช้ `alias` ชี้ไปที่ Path ของ staticfiles ที่ถูกต้อง:
```nginx
location /static/ {
    alias /var/www/mos_ecom/staticfiles/;
}
5. Handling Large File Uploads
Issue: 413 Request Entity Too Large
Solution:

Nginx: เพิ่ม client_max_body_size 50M; ใน server block

Django (settings.py):

Python
FILE_UPLOAD_MAX_MEMORY_SIZE = 52428800
DATA_UPLOAD_MAX_MEMORY_SIZE = 52428800
Last Updated: 2026-04-23
"""
return content

try:
markdown_output = generate_summary_markdown()
with open('django_deployment_log_v1.md', 'w', encoding='utf-8') as f:
f.write(markdown_output)
logger.info("Successfully generated deployment summary file.")
except Exception as e:
logger.error(f"Failed to generate summary file: {e}")


ประกอบไปด้วยเนื้อหา:
- การแก้ปัญหา **WeasyPrint** (Missing OS libraries)
- การตั้งค่า **Nginx/Gunicorn** แบบเลี่ยง Port เดิม
- การตรวจสอบความถูกต้องของ **ALLOWED_HOSTS** และไฟล์ `.env`
- การจัดการ **Static Files** สำหรับ Admin Panel
- การปลดล็อคข้อจำกัดการอัปโหลดไฟล์ขนาดใหญ่ (**413 Error**)

หากมีการเปลี่ยนแปลงหรือต้องการเพิ่มหัวข้อใด แจ้งให้ผมดำเนินการต่อได้เลยครับ!