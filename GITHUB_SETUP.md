# دليل نشر المشروع على GitHub

## خطوات النشر على GitHub

### 1. إنشاء Repository جديد على GitHub

1. اذهب إلى [GitHub.com](https://github.com)
2. انقر على "New repository" أو الزر الأخضر "New"
3. املأ البيانات التالية:
   - **Repository name**: `erp-sales-module`
   - **Description**: `نظام إدارة المبيعات المتكامل مع دعم الواتساب والتنبيهات الذكية`
   - **Visibility**: اختر Public أو Private حسب رغبتك
   - **لا تختر** "Add a README file" (لأن لدينا ملفات جاهزة)
   - **لا تختر** "Add .gitignore" (لأن لدينا ملف جاهز)
   - **اختر License**: MIT License
4. انقر على "Create repository"

### 2. ربط المشروع المحلي بـ GitHub

بعد إنشاء Repository، ستظهر لك صفحة بتعليمات. استخدم الأوامر التالية:

```bash
# إضافة remote origin
git remote add origin https://github.com/YOUR_USERNAME/erp-sales-module.git

# تغيير اسم الفرع إلى main (اختياري)
git branch -M main

# رفع الكود لأول مرة
git push -u origin main
```

**ملاحظة**: استبدل `YOUR_USERNAME` باسم المستخدم الخاص بك على GitHub.

### 3. إعداد GitHub Pages (اختياري)

لعرض التوثيق على الإنترنت:

1. اذهب إلى إعدادات Repository
2. انقر على "Pages" من القائمة الجانبية
3. في "Source" اختر "Deploy from a branch"
4. اختر "main" branch و "/ (root)" folder
5. انقر على "Save"

### 4. إعداد GitHub Actions للـ CI/CD (اختياري)

إنشاء ملف `.github/workflows/ci.yml`:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test-backend:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    
    - name: Run tests
      run: |
        python manage.py test
      env:
        DATABASE_URL: postgres://postgres:postgres@localhost:5432/test_db

  test-frontend:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: erp-frontend/package-lock.json
    
    - name: Install dependencies
      run: |
        cd erp-frontend
        npm ci
    
    - name: Build project
      run: |
        cd erp-frontend
        npm run build
    
    - name: Run tests
      run: |
        cd erp-frontend
        npm test
```

## إعداد Repository للمساهمين

### 1. إنشاء ملف CONTRIBUTING.md

```markdown
# المساهمة في المشروع

## كيفية المساهمة

1. Fork المشروع
2. إنشاء فرع جديد (`git checkout -b feature/amazing-feature`)
3. Commit التغييرات (`git commit -m 'Add amazing feature'`)
4. Push للفرع (`git push origin feature/amazing-feature`)
5. فتح Pull Request

## معايير الكود

- اتبع PEP 8 للـ Python
- استخدم ESLint للـ JavaScript
- أضف تعليقات باللغة العربية
- اكتب اختبارات للميزات الجديدة

## الإبلاغ عن الأخطاء

استخدم GitHub Issues لإبلاغ عن الأخطاء مع تضمين:
- وصف مفصل للمشكلة
- خطوات إعادة الإنتاج
- البيئة المستخدمة
- لقطات شاشة إن أمكن
```

### 2. إنشاء Issue Templates

إنشاء مجلد `.github/ISSUE_TEMPLATE/` مع الملفات التالية:

**bug_report.md**:
```markdown
---
name: تقرير خطأ
about: أبلغ عن خطأ لمساعدتنا في التحسين
title: '[BUG] '
labels: bug
assignees: ''
---

## وصف الخطأ
وصف واضح ومختصر للخطأ.

## خطوات إعادة الإنتاج
1. اذهب إلى '...'
2. انقر على '....'
3. مرر لأسفل إلى '....'
4. شاهد الخطأ

## السلوك المتوقع
وصف واضح ومختصر لما كنت تتوقع حدوثه.

## لقطات الشاشة
إذا كان ذلك مناسباً، أضف لقطات شاشة لمساعدتنا في شرح مشكلتك.

## معلومات البيئة:
- نظام التشغيل: [مثل iOS]
- المتصفح: [مثل chrome, safari]
- الإصدار: [مثل 22]

## سياق إضافي
أضف أي سياق آخر حول المشكلة هنا.
```

**feature_request.md**:
```markdown
---
name: طلب ميزة
about: اقترح فكرة لهذا المشروع
title: '[FEATURE] '
labels: enhancement
assignees: ''
---

## هل طلب الميزة مرتبط بمشكلة؟
وصف واضح ومختصر للمشكلة. مثال: أشعر بالإحباط عندما [...]

## وصف الحل المطلوب
وصف واضح ومختصر لما تريده أن يحدث.

## وصف البدائل المدروسة
وصف واضح ومختصر لأي حلول أو ميزات بديلة فكرت فيها.

## سياق إضافي
أضف أي سياق أو لقطات شاشة أخرى حول طلب الميزة هنا.
```

### 3. إعداد Pull Request Template

إنشاء ملف `.github/pull_request_template.md`:

```markdown
## وصف التغييرات

وصف مختصر للتغييرات المقترحة.

## نوع التغيير

- [ ] إصلاح خطأ (تغيير لا يكسر الوظائف الموجودة ويحل مشكلة)
- [ ] ميزة جديدة (تغيير لا يكسر الوظائف الموجودة ويضيف وظيفة)
- [ ] تغيير كاسر (إصلاح أو ميزة تتسبب في عدم عمل الوظائف الموجودة كما هو متوقع)
- [ ] تحديث التوثيق

## كيف تم اختبار هذا؟

وصف الاختبارات التي أجريتها للتحقق من تغييراتك.

- [ ] اختبار A
- [ ] اختبار B

## قائمة التحقق:

- [ ] الكود يتبع إرشادات الأسلوب لهذا المشروع
- [ ] لقد أجريت مراجعة ذاتية للكود الخاص بي
- [ ] لقد علقت على الكود الخاص بي، خاصة في المناطق صعبة الفهم
- [ ] لقد أجريت التغييرات المقابلة على التوثيق
- [ ] تغييراتي لا تولد تحذيرات جديدة
- [ ] لقد أضفت اختبارات تثبت أن إصلاحي فعال أو أن ميزتي تعمل
- [ ] الاختبارات الجديدة والموجودة تمر محلياً مع تغييراتي
```

## إعداد الحماية والأمان

### 1. إعداد Branch Protection Rules

1. اذهب إلى Settings > Branches
2. انقر على "Add rule"
3. في "Branch name pattern" اكتب `main`
4. فعّل:
   - "Require a pull request before merging"
   - "Require status checks to pass before merging"
   - "Require branches to be up to date before merging"
   - "Include administrators"

### 2. إعداد Secrets للـ CI/CD

1. اذهب إلى Settings > Secrets and variables > Actions
2. أضف المتغيرات التالية:
   - `DATABASE_URL`
   - `SECRET_KEY`
   - `WHATSAPP_ACCESS_TOKEN`

## إعداد Releases

### 1. إنشاء أول Release

```bash
# إنشاء tag للإصدار
git tag -a v1.0.0 -m "الإصدار الأول من نظام إدارة المبيعات"

# رفع التاغ
git push origin v1.0.0
```

### 2. إنشاء Release على GitHub

1. اذهب إلى "Releases" في صفحة Repository
2. انقر على "Create a new release"
3. اختر التاغ `v1.0.0`
4. أضف عنوان: "الإصدار الأول - نظام إدارة المبيعات v1.0.0"
5. أضف وصف للإصدار:

```markdown
## الميزات الجديدة 🚀

- نظام إدارة المبيعات المتكامل
- دعم الواتساب للتنبيهات
- واجهة مستخدم باللغة العربية
- لوحة معلومات تفاعلية
- إدارة العملاء والمنتجات والطلبات
- نظام صلاحيات متقدم
- تقارير وإحصائيات شاملة

## التقنيات المستخدمة 🛠️

- Backend: Django 5.2.4 + DRF
- Frontend: React + Vite
- Database: PostgreSQL/SQLite
- Authentication: JWT
- UI: Material-UI مع دعم RTL

## التثبيت 📦

راجع ملف `INSTALLATION.md` للحصول على تعليمات التثبيت المفصلة.

## التوثيق 📚

- [دليل المطور](DEVELOPER_GUIDE.md)
- [دليل المستخدم](USER_MANUAL.md)
- [دليل التثبيت](INSTALLATION.md)
```

## نصائح للنشر الناجح

### 1. تحسين README.md

تأكد من أن ملف README يحتوي على:
- وصف واضح للمشروع
- لقطات شاشة
- تعليمات التثبيت السريع
- روابط للتوثيق
- معلومات الترخيص
- طرق المساهمة

### 2. إضافة Badges

أضف badges في أعلى README:

```markdown
![GitHub release](https://img.shields.io/github/release/YOUR_USERNAME/erp-sales-module.svg)
![GitHub license](https://img.shields.io/github/license/YOUR_USERNAME/erp-sales-module.svg)
![GitHub issues](https://img.shields.io/github/issues/YOUR_USERNAME/erp-sales-module.svg)
![GitHub stars](https://img.shields.io/github/stars/YOUR_USERNAME/erp-sales-module.svg)
```

### 3. إعداد GitHub Pages للتوثيق

إنشاء مجلد `docs/` مع ملفات HTML للتوثيق التفاعلي.

### 4. إضافة Demo

إذا أمكن، أضف رابط لنسخة تجريبية من النظام.

## الصيانة المستمرة

### 1. تحديث التبعيات

```bash
# تحديث Python packages
pip list --outdated
pip install --upgrade package_name

# تحديث Node packages
npm outdated
npm update
```

### 2. مراقبة الأمان

- فعّل GitHub Security Alerts
- استخدم Dependabot للتحديثات التلقائية
- راجع Security Advisories بانتظام

### 3. إدارة Issues و Pull Requests

- رد على Issues خلال 48 ساعة
- راجع Pull Requests بانتظام
- استخدم Labels لتنظيم المشاكل
- أغلق Issues المحلولة

---

**تهانينا! مشروعك جاهز للنشر على GitHub** 🎉

تذكر أن النشر هو البداية فقط. الصيانة المستمرة والتفاعل مع المجتمع هما مفتاح نجاح المشروع.

