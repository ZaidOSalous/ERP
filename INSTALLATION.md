# دليل التثبيت - نظام إدارة المبيعات

## متطلبات النظام

### متطلبات الخادم
- **نظام التشغيل**: Ubuntu 20.04+ / CentOS 8+ / Windows 10+
- **المعالج**: 2 CPU cores أو أكثر
- **الذاكرة**: 4GB RAM كحد أدنى (8GB مُوصى به)
- **التخزين**: 20GB مساحة فارغة كحد أدنى
- **الشبكة**: اتصال إنترنت مستقر

### البرامج المطلوبة
- **Python**: 3.11 أو أحدث
- **Node.js**: 18.0 أو أحدث
- **PostgreSQL**: 14.0 أو أحدث (أو SQLite للتطوير)
- **Redis**: 6.0 أو أحدث (للتخزين المؤقت والمهام)
- **Git**: لإدارة الكود

## التثبيت على Ubuntu/Linux

### 1. تحديث النظام
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. تثبيت Python و pip
```bash
sudo apt install python3.11 python3.11-venv python3-pip -y
```

### 3. تثبيت Node.js
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 4. تثبيت PostgreSQL
```bash
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### 5. تثبيت Redis
```bash
sudo apt install redis-server -y
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

### 6. تثبيت Git
```bash
sudo apt install git -y
```

## إعداد قاعدة البيانات

### إنشاء قاعدة بيانات PostgreSQL
```bash
sudo -u postgres psql
```

```sql
CREATE DATABASE erp_sales_db;
CREATE USER erp_user WITH PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE erp_sales_db TO erp_user;
ALTER USER erp_user CREATEDB;
\q
```

## تحميل وإعداد المشروع

### 1. استنساخ المشروع
```bash
git clone https://github.com/your-username/erp-sales-module.git
cd erp-sales-module
```

### 2. إنشاء البيئة الافتراضية
```bash
python3.11 -m venv venv
source venv/bin/activate  # على Linux/Mac
# أو
venv\Scripts\activate     # على Windows
```

### 3. تثبيت تبعيات Python
```bash
pip install -r requirements.txt
```

### 4. إعداد متغيرات البيئة
```bash
cp .env.example .env
nano .env
```

أضف المتغيرات التالية في ملف `.env`:
```env
# Django Settings
SECRET_KEY=your-very-secret-key-here
DEBUG=False
ALLOWED_HOSTS=localhost,127.0.0.1,your-domain.com

# Database Settings
DB_ENGINE=django.db.backends.postgresql
DB_NAME=erp_sales_db
DB_USER=erp_user
DB_PASSWORD=your_secure_password
DB_HOST=localhost
DB_PORT=5432

# Redis Settings
REDIS_URL=redis://localhost:6379/0

# Email Settings (اختياري)
EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-app-password

# WhatsApp API Settings (اختياري)
WHATSAPP_API_URL=https://graph.facebook.com/v17.0
WHATSAPP_ACCESS_TOKEN=your-whatsapp-token

# Security Settings
SECURE_SSL_REDIRECT=False
SECURE_BROWSER_XSS_FILTER=True
SECURE_CONTENT_TYPE_NOSNIFF=True
```

### 5. تطبيق هجرات قاعدة البيانات
```bash
python manage.py makemigrations
python manage.py migrate
```

### 6. إنشاء مستخدم إداري
```bash
python manage.py createsuperuser
```

### 7. جمع الملفات الثابتة
```bash
python manage.py collectstatic --noinput
```

## إعداد Frontend (React)

### 1. الانتقال لمجلد Frontend
```bash
cd erp-frontend
```

### 2. تثبيت تبعيات Node.js
```bash
npm install
```

### 3. إعداد متغيرات البيئة للـ Frontend
```bash
cp .env.example .env.local
nano .env.local
```

أضف المتغيرات التالية:
```env
VITE_API_BASE_URL=http://localhost:8000/api
VITE_APP_NAME=نظام إدارة المبيعات
VITE_APP_VERSION=1.0.0
```

### 4. بناء المشروع للإنتاج
```bash
npm run build
```

## تشغيل النظام

### تشغيل Backend (Django)
```bash
# في مجلد المشروع الرئيسي
source venv/bin/activate
python manage.py runserver 0.0.0.0:8000
```

### تشغيل Frontend (React) - للتطوير
```bash
# في مجلد erp-frontend
npm run dev
```

### تشغيل Frontend (React) - للإنتاج
```bash
# بعد بناء المشروع، استخدم خادم ويب مثل Nginx
sudo apt install nginx -y
sudo cp -r dist/* /var/www/html/
sudo systemctl restart nginx
```

## إعداد الإنتاج

### 1. تثبيت Gunicorn
```bash
pip install gunicorn
```

### 2. إنشاء ملف خدمة systemd
```bash
sudo nano /etc/systemd/system/erp-sales.service
```

```ini
[Unit]
Description=ERP Sales Django Application
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/path/to/erp-sales-module
Environment="PATH=/path/to/erp-sales-module/venv/bin"
ExecStart=/path/to/erp-sales-module/venv/bin/gunicorn --workers 3 --bind unix:/path/to/erp-sales-module/erp_sales.sock erp_backend.wsgi:application
Restart=always

[Install]
WantedBy=multi-user.target
```

### 3. تفعيل وتشغيل الخدمة
```bash
sudo systemctl daemon-reload
sudo systemctl start erp-sales
sudo systemctl enable erp-sales
```

### 4. إعداد Nginx
```bash
sudo nano /etc/nginx/sites-available/erp-sales
```

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location = /favicon.ico { access_log off; log_not_found off; }
    
    location /static/ {
        root /path/to/erp-sales-module;
    }

    location /media/ {
        root /path/to/erp-sales-module;
    }

    location /api/ {
        include proxy_params;
        proxy_pass http://unix:/path/to/erp-sales-module/erp_sales.sock;
    }

    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;
    }
}
```

### 5. تفعيل موقع Nginx
```bash
sudo ln -s /etc/nginx/sites-available/erp-sales /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

## إعداد SSL (HTTPS)

### 1. تثبيت Certbot
```bash
sudo apt install certbot python3-certbot-nginx -y
```

### 2. الحصول على شهادة SSL
```bash
sudo certbot --nginx -d your-domain.com
```

### 3. تجديد تلقائي للشهادة
```bash
sudo crontab -e
```

أضف السطر التالي:
```bash
0 12 * * * /usr/bin/certbot renew --quiet
```

## إعداد النسخ الاحتياطية

### 1. نسخ احتياطية لقاعدة البيانات
```bash
#!/bin/bash
# backup_db.sh
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/database"
mkdir -p $BACKUP_DIR

pg_dump -h localhost -U erp_user erp_sales_db > $BACKUP_DIR/erp_sales_$DATE.sql
gzip $BACKUP_DIR/erp_sales_$DATE.sql

# حذف النسخ الأقدم من 30 يوم
find $BACKUP_DIR -name "*.sql.gz" -mtime +30 -delete
```

### 2. نسخ احتياطية للملفات
```bash
#!/bin/bash
# backup_files.sh
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/files"
PROJECT_DIR="/path/to/erp-sales-module"

mkdir -p $BACKUP_DIR
tar -czf $BACKUP_DIR/erp_files_$DATE.tar.gz -C $PROJECT_DIR media/ static/

# حذف النسخ الأقدم من 30 يوم
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete
```

### 3. جدولة النسخ الاحتياطية
```bash
sudo crontab -e
```

أضف الأسطر التالية:
```bash
# نسخة احتياطية يومية لقاعدة البيانات في الساعة 2:00 صباحاً
0 2 * * * /path/to/backup_db.sh

# نسخة احتياطية أسبوعية للملفات يوم الأحد في الساعة 3:00 صباحاً
0 3 * * 0 /path/to/backup_files.sh
```

## المراقبة والصيانة

### 1. مراقبة السجلات
```bash
# سجلات Django
tail -f /path/to/erp-sales-module/logs/django.log

# سجلات Nginx
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# سجلات النظام
sudo journalctl -u erp-sales -f
```

### 2. مراقبة الأداء
```bash
# استخدام الذاكرة والمعالج
htop

# مساحة القرص الصلب
df -h

# حالة قاعدة البيانات
sudo -u postgres psql -c "SELECT * FROM pg_stat_activity;"
```

### 3. تحديث النظام
```bash
# تحديث الكود
git pull origin main

# تحديث التبعيات
source venv/bin/activate
pip install -r requirements.txt

# تطبيق هجرات جديدة
python manage.py migrate

# جمع الملفات الثابتة
python manage.py collectstatic --noinput

# إعادة تشغيل الخدمة
sudo systemctl restart erp-sales
sudo systemctl restart nginx
```

## استكشاف الأخطاء

### مشاكل شائعة وحلولها

#### 1. خطأ في الاتصال بقاعدة البيانات
```bash
# تحقق من حالة PostgreSQL
sudo systemctl status postgresql

# تحقق من إعدادات الاتصال
sudo -u postgres psql -c "SELECT version();"
```

#### 2. خطأ في تحميل الملفات الثابتة
```bash
# تحقق من صلاحيات المجلدات
sudo chown -R www-data:www-data /path/to/erp-sales-module/static/
sudo chmod -R 755 /path/to/erp-sales-module/static/
```

#### 3. خطأ في تشغيل Gunicorn
```bash
# تحقق من سجلات الخدمة
sudo journalctl -u erp-sales -n 50

# تشغيل يدوي للاختبار
source venv/bin/activate
gunicorn --bind 0.0.0.0:8000 erp_backend.wsgi:application
```

#### 4. خطأ في Nginx
```bash
# تحقق من تكوين Nginx
sudo nginx -t

# إعادة تحميل التكوين
sudo systemctl reload nginx
```

## الأمان

### 1. تحديث النظام بانتظام
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. إعداد جدار الحماية
```bash
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 80
sudo ufw allow 443
```

### 3. تأمين PostgreSQL
```bash
# تغيير كلمة مرور postgres
sudo -u postgres psql -c "ALTER USER postgres PASSWORD 'new_secure_password';"

# تحديث pg_hba.conf
sudo nano /etc/postgresql/14/main/pg_hba.conf
```

### 4. تأمين Redis
```bash
# إضافة كلمة مرور لـ Redis
sudo nano /etc/redis/redis.conf
# أضف: requirepass your_redis_password
sudo systemctl restart redis-server
```

## الدعم والمساعدة

### موارد مفيدة
- **التوثيق الرسمي**: [رابط التوثيق]
- **المجتمع**: [رابط المنتدى]
- **الإبلاغ عن الأخطاء**: [رابط GitHub Issues]

### معلومات الاتصال
- **البريد الإلكتروني**: support@erp-sales.com
- **الهاتف**: +966-XX-XXX-XXXX

---

**تهانينا! تم تثبيت نظام إدارة المبيعات بنجاح** 🎉

للحصول على أفضل أداء، تأكد من مراقبة النظام بانتظام وتطبيق التحديثات الأمنية.

