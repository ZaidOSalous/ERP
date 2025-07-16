# نظام إدارة المبيعات - ERP Sales Module

## نظرة عامة

نظام إدارة المبيعات هو وحدة شاملة من نظام تخطيط موارد المؤسسة (ERP) مصممة خصيصاً للشركات العربية. يوفر النظام إدارة كاملة للمبيعات مع دعم اللغتين العربية والإنجليزية، وتكامل مع الواتساب، ونظام تنبيهات ذكي.

## الميزات الرئيسية

### 🏢 **إدارة الفروع والأقسام**
- نظام هرمي للفروع والأقسام
- صلاحيات متدرجة حسب الفرع والقسم
- تقارير منفصلة لكل فرع

### 👥 **إدارة المستخدمين**
- أدوار متعددة: مدير، مندوب مبيعات، محاسب
- صلاحيات مفصلة لكل دور
- ملفات شخصية شاملة

### 🛒 **إدارة المبيعات**
- إدارة العملاء مع فئات وتقييمات
- إدارة المنتجات مع مخزون وأسعار متدرجة
- طلبات مبيعات متكاملة مع تتبع الحالة
- أهداف مبيعات وتتبع الأداء

### 📱 **تكامل الواتساب**
- إرسال واستقبال الرسائل
- قوالب رسائل قابلة للتخصيص
- تنبيهات تلقائية للعملاء

### 🔔 **نظام التنبيهات الذكي**
- تنبيهات فورية ودورية
- قنوات متعددة: داخل النظام، بريد إلكتروني، واتساب
- تنبيهات ذكية مبنية على السلوك

### 📊 **التقارير والتحليلات**
- لوحة معلومات تفاعلية
- رسوم بيانية متقدمة
- تقارير قابلة للتخصيص والتصدير

## التقنيات المستخدمة

### Backend
- **Python 3.11** - لغة البرمجة الأساسية
- **Django 5.2.4** - إطار العمل الرئيسي
- **Django REST Framework** - لبناء APIs
- **SQLite** - قاعدة البيانات (قابلة للتغيير إلى PostgreSQL)
- **JWT** - للمصادقة والأمان

### Frontend
- **React 18** - مكتبة واجهة المستخدم
- **TypeScript** - للتطوير المتقدم
- **Vite** - أداة البناء السريعة
- **Tailwind CSS** - للتصميم
- **Recharts** - للرسوم البيانية

### الميزات التقنية
- **دعم كامل للغة العربية** مع RTL
- **تصميم متجاوب** للموبايل والديسكتوب
- **نظام ثيمات** (فاتح/داكن)
- **أمان متقدم** مع تشفير البيانات

## هيكل المشروع

```
erp_sales_module/
├── erp_backend/           # Django Backend
│   ├── settings.py        # إعدادات Django
│   ├── urls.py           # مسارات المشروع
│   └── wsgi.py           # إعدادات WSGI
├── users/                # تطبيق المستخدمين
│   ├── models.py         # نماذج المستخدمين والفروع
│   ├── views.py          # عروض API
│   ├── serializers.py    # مسلسلات البيانات
│   └── permissions.py    # الصلاحيات المخصصة
├── sales/                # تطبيق المبيعات
│   ├── models.py         # نماذج المبيعات
│   ├── views.py          # عروض المبيعات
│   └── serializers.py    # مسلسلات المبيعات
├── notifications/        # تطبيق التنبيهات
│   ├── models.py         # نماذج التنبيهات والواتساب
│   └── services.py       # خدمات الإشعارات
├── erp-frontend/         # React Frontend
│   ├── src/
│   │   ├── components/   # مكونات React
│   │   ├── contexts/     # Context للحالة العامة
│   │   └── App.jsx       # التطبيق الرئيسي
│   └── package.json      # تبعيات Node.js
├── requirements.txt      # تبعيات Python
└── README.md            # هذا الملف
```

## التثبيت والتشغيل

### متطلبات النظام
- Python 3.8+
- Node.js 18+
- Git

### خطوات التثبيت

#### 1. تحميل المشروع
```bash
git clone <repository-url>
cd erp_sales_module
```

#### 2. إعداد Backend (Django)
```bash
# إنشاء البيئة الافتراضية
python -m venv venv

# تفعيل البيئة الافتراضية
# على Linux/Mac:
source venv/bin/activate
# على Windows:
venv\Scripts\activate

# تثبيت التبعيات
pip install -r requirements.txt

# إنشاء قاعدة البيانات
python manage.py migrate

# إنشاء مستخدم إداري
python manage.py createsuperuser

# تشغيل الخادم
python manage.py runserver
```

#### 3. إعداد Frontend (React)
```bash
# الانتقال لمجلد Frontend
cd erp-frontend

# تثبيت التبعيات
npm install

# تشغيل التطبيق
npm run dev
```

### الوصول للنظام
- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000/api/
- **Django Admin**: http://localhost:8000/admin/

## حسابات تجريبية

للاختبار، يمكن استخدام الحسابات التالية:

| اسم المستخدم | كلمة المرور | الدور |
|--------------|-------------|-------|
| admin | admin123 | مدير النظام |
| sales_manager | sales123 | مدير المبيعات |
| sales_rep | rep123 | مندوب مبيعات |

## الاستخدام

### 1. تسجيل الدخول
- افتح المتصفح على http://localhost:3000
- استخدم أحد الحسابات التجريبية
- أو انقر على الحساب المطلوب لملء البيانات تلقائياً

### 2. لوحة المعلومات
- عرض إحصائيات المبيعات اليومية والشهرية
- رسوم بيانية تفاعلية للاتجاهات
- تنبيهات مهمة وإجراءات سريعة

### 3. إدارة العملاء
- إضافة وتعديل بيانات العملاء
- تصنيف العملاء (VIP، Premium، Regular)
- تتبع تاريخ التعاملات والطلبات

### 4. إدارة المنتجات
- إضافة منتجات جديدة مع الصور والأوصاف
- إدارة المخزون والأسعار
- تنبيهات المخزون المنخفض

### 5. طلبات المبيعات
- إنشاء طلبات جديدة
- تتبع حالة الطلبات
- طباعة وإرسال الطلبات

## API Documentation

### المصادقة
جميع APIs تتطلب مصادقة باستخدام JWT Token:

```bash
# تسجيل الدخول
POST /api/auth/login/
{
  "username": "admin",
  "password": "admin123"
}

# استخدام Token في الطلبات
Authorization: Bearer <your-token>
```

### نقاط النهاية الرئيسية

#### المستخدمين
- `GET /api/users/` - قائمة المستخدمين
- `POST /api/users/` - إضافة مستخدم جديد
- `GET /api/branches/` - قائمة الفروع
- `GET /api/departments/` - قائمة الأقسام

#### المبيعات
- `GET /api/customers/` - قائمة العملاء
- `POST /api/customers/` - إضافة عميل جديد
- `GET /api/products/` - قائمة المنتجات
- `GET /api/orders/` - قائمة الطلبات
- `POST /api/orders/` - إنشاء طلب جديد

#### التقارير
- `GET /api/dashboard/stats/` - إحصائيات لوحة المعلومات
- `GET /api/reports/sales/` - تقارير المبيعات

## التخصيص والتطوير

### إضافة ميزات جديدة

#### 1. إضافة نموذج جديد (Backend)
```python
# في models.py
class NewModel(models.Model):
    name = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        verbose_name = "النموذج الجديد"
        verbose_name_plural = "النماذج الجديدة"
```

#### 2. إضافة مكون جديد (Frontend)
```jsx
// في components/NewComponent.jsx
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'

export default function NewComponent() {
  return (
    <Card>
      <CardHeader>
        <CardTitle className="text-right">المكون الجديد</CardTitle>
      </CardHeader>
      <CardContent>
        {/* محتوى المكون */}
      </CardContent>
    </Card>
  )
}
```

### إعدادات الواتساب

لتفعيل تكامل الواتساب:

1. احصل على WhatsApp Business API credentials
2. أضف المعلومات في إعدادات Django:
```python
# في settings.py
WHATSAPP_API_URL = "your-whatsapp-api-url"
WHATSAPP_TOKEN = "your-whatsapp-token"
```

## الأمان

### الممارسات الأمنية المطبقة
- **تشفير كلمات المرور** باستخدام Django's built-in hashing
- **JWT Tokens** للمصادقة الآمنة
- **CORS** محدود للنطاقات المسموحة
- **صلاحيات متدرجة** لكل نوع مستخدم
- **تسجيل العمليات** لتتبع النشاط

### توصيات الأمان للإنتاج
- استخدم HTTPS في الإنتاج
- قم بتغيير SECRET_KEY في Django
- استخدم قاعدة بيانات آمنة (PostgreSQL)
- فعل النسخ الاحتياطية المنتظمة
- راقب سجلات النظام

## النشر في الإنتاج

### متطلبات الخادم
- Ubuntu 20.04+ أو CentOS 8+
- Python 3.8+
- Node.js 18+
- Nginx
- PostgreSQL 12+

### خطوات النشر

#### 1. إعداد قاعدة البيانات
```bash
# تثبيت PostgreSQL
sudo apt install postgresql postgresql-contrib

# إنشاء قاعدة البيانات
sudo -u postgres createdb erp_sales_db
sudo -u postgres createuser erp_user
```

#### 2. إعداد Django للإنتاج
```python
# في settings.py
DEBUG = False
ALLOWED_HOSTS = ['your-domain.com']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'erp_sales_db',
        'USER': 'erp_user',
        'PASSWORD': 'your-password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

#### 3. إعداد Nginx
```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location / {
        root /path/to/frontend/dist;
        try_files $uri $uri/ /index.html;
    }
}
```

## الدعم والمساعدة

### الأسئلة الشائعة

**س: كيف أضيف مستخدم جديد؟**
ج: من لوحة المعلومات، اذهب إلى "إدارة المستخدمين" ← "إضافة مستخدم جديد"

**س: كيف أغير كلمة المرور؟**
ج: من قائمة المستخدم في الشريط العلوي ← "الملف الشخصي" ← "تغيير كلمة المرور"

**س: كيف أصدر تقرير مبيعات؟**
ج: اذهب إلى "التقارير والتحليلات" ← "تقارير المبيعات" ← اختر الفترة الزمنية ← "تصدير"

### التواصل
- **البريد الإلكتروني**: support@erp-sales.com
- **الهاتف**: +966-XX-XXX-XXXX
- **الدعم الفني**: متوفر 24/7

## الترخيص

هذا المشروع مرخص تحت رخصة MIT. راجع ملف LICENSE للتفاصيل.

## المساهمة

نرحب بالمساهمات! يرجى قراءة دليل المساهمة قبل إرسال Pull Request.

---

**تم تطوير هذا النظام بواسطة فريق التطوير المتخصص في أنظمة ERP العربية**

© 2024 نظام إدارة المبيعات. جميع الحقوق محفوظة.

