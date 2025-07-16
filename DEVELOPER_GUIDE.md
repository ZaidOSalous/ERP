# دليل المطور التقني - ERP Sales Module

## نظرة عامة على الهندسة

نظام إدارة المبيعات مبني على هندسة منفصلة (Decoupled Architecture) مع:
- **Backend**: Django REST API
- **Frontend**: React SPA
- **Database**: SQLite (قابل للترقية إلى PostgreSQL)
- **Authentication**: JWT-based

## هيكل قاعدة البيانات

### نماذج المستخدمين (users/models.py)

#### Branch (الفروع)
```python
class Branch(models.Model):
    name = models.CharField(max_length=100)           # اسم الفرع
    code = models.CharField(max_length=20, unique=True)  # كود الفرع
    address = models.TextField()                      # العنوان
    phone = models.CharField(max_length=20)           # الهاتف
    email = models.EmailField()                       # البريد الإلكتروني
    manager = models.ForeignKey(User)                 # مدير الفرع
    is_main = models.BooleanField(default=False)      # فرع رئيسي؟
    is_active = models.BooleanField(default=True)     # نشط؟
```

#### Department (الأقسام)
```python
class Department(models.Model):
    name = models.CharField(max_length=100)           # اسم القسم
    code = models.CharField(max_length=20)            # كود القسم
    branch = models.ForeignKey(Branch)                # الفرع التابع له
    parent = models.ForeignKey('self', null=True)     # القسم الأب
    manager = models.ForeignKey(User)                 # مدير القسم
    description = models.TextField()                  # وصف القسم
```

#### UserProfile (ملف المستخدم)
```python
class UserProfile(models.Model):
    user = models.OneToOneField(User)                 # المستخدم
    employee_id = models.CharField(max_length=20)     # رقم الموظف
    branch = models.ForeignKey(Branch)                # الفرع
    department = models.ForeignKey(Department)        # القسم
    role = models.CharField(max_length=50)            # الدور
    phone = models.CharField(max_length=20)           # الهاتف
    avatar = models.ImageField()                      # الصورة الشخصية
    hire_date = models.DateField()                    # تاريخ التوظيف
```

### نماذج المبيعات (sales/models.py)

#### Customer (العملاء)
```python
class Customer(models.Model):
    name = models.CharField(max_length=200)           # اسم العميل
    email = models.EmailField()                       # البريد الإلكتروني
    phone = models.CharField(max_length=20)           # الهاتف
    whatsapp = models.CharField(max_length=20)        # واتساب
    address = models.TextField()                      # العنوان
    category = models.CharField(max_length=20)        # فئة العميل
    credit_limit = models.DecimalField()              # حد الائتمان
    branch = models.ForeignKey(Branch)                # الفرع
    sales_rep = models.ForeignKey(User)               # مندوب المبيعات
```

#### Product (المنتجات)
```python
class Product(models.Model):
    name = models.CharField(max_length=200)           # اسم المنتج
    code = models.CharField(max_length=50)            # كود المنتج
    description = models.TextField()                  # الوصف
    category = models.CharField(max_length=100)       # الفئة
    unit = models.CharField(max_length=20)            # الوحدة
    cost_price = models.DecimalField()                # سعر التكلفة
    selling_price = models.DecimalField()             # سعر البيع
    stock_quantity = models.IntegerField()            # الكمية المتوفرة
    min_stock_level = models.IntegerField()           # الحد الأدنى للمخزون
```

#### SalesOrder (طلبات المبيعات)
```python
class SalesOrder(models.Model):
    order_number = models.CharField(max_length=50)    # رقم الطلب
    customer = models.ForeignKey(Customer)            # العميل
    sales_rep = models.ForeignKey(User)               # مندوب المبيعات
    order_date = models.DateTimeField()               # تاريخ الطلب
    delivery_date = models.DateField()                # تاريخ التسليم
    status = models.CharField(max_length=20)          # حالة الطلب
    total_amount = models.DecimalField()              # إجمالي المبلغ
    discount = models.DecimalField()                  # الخصم
    tax = models.DecimalField()                       # الضريبة
    notes = models.TextField()                        # ملاحظات
```

### نماذج التنبيهات (notifications/models.py)

#### WhatsAppConfig (إعدادات الواتساب)
```python
class WhatsAppConfig(models.Model):
    branch = models.OneToOneField(Branch)             # الفرع
    api_url = models.URLField()                       # رابط API
    access_token = models.CharField(max_length=500)   # رمز الوصول
    phone_number = models.CharField(max_length=20)    # رقم الهاتف
    is_active = models.BooleanField()                 # نشط؟
```

#### MessageTemplate (قوالب الرسائل)
```python
class MessageTemplate(models.Model):
    name = models.CharField(max_length=100)           # اسم القالب
    template_type = models.CharField(max_length=50)   # نوع القالب
    content_ar = models.TextField()                   # المحتوى بالعربية
    content_en = models.TextField()                   # المحتوى بالإنجليزية
    variables = models.JSONField()                    # المتغيرات
```

## هندسة Backend (Django)

### بنية المشروع
```
erp_backend/
├── settings.py          # الإعدادات الرئيسية
├── urls.py             # مسارات المشروع
└── wsgi.py             # إعدادات WSGI

users/                  # تطبيق المستخدمين
├── models.py           # نماذج البيانات
├── serializers.py      # مسلسلات API
├── views.py            # عروض API
├── permissions.py      # الصلاحيات المخصصة
└── urls.py             # مسارات التطبيق

sales/                  # تطبيق المبيعات
├── models.py           # نماذج المبيعات
├── serializers.py      # مسلسلات المبيعات
├── views.py            # عروض المبيعات
└── urls.py             # مسارات المبيعات

notifications/          # تطبيق التنبيهات
├── models.py           # نماذج التنبيهات
├── services.py         # خدمات الإشعارات
└── tasks.py            # مهام الخلفية
```

### نظام الصلاحيات

#### صلاحيات مخصصة (users/permissions.py)
```python
class BranchPermission(BasePermission):
    """صلاحية الوصول للفرع"""
    def has_permission(self, request, view):
        user_branch = request.user.userprofile.branch
        return user_branch.is_active

class DepartmentPermission(BasePermission):
    """صلاحية الوصول للقسم"""
    def has_object_permission(self, request, view, obj):
        user_dept = request.user.userprofile.department
        return obj.department == user_dept or user_dept.parent == obj.department
```

#### استخدام الصلاحيات في Views
```python
class CustomerViewSet(ModelViewSet):
    permission_classes = [IsAuthenticated, BranchPermission]
    
    def get_queryset(self):
        user_branch = self.request.user.userprofile.branch
        return Customer.objects.filter(branch=user_branch)
```

### نظام المصادقة JWT

#### إعدادات JWT (settings.py)
```python
from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(hours=24),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
}
```

#### استخدام JWT في Frontend
```javascript
// تخزين Token
localStorage.setItem('access_token', response.data.access);

// استخدام Token في الطلبات
const config = {
  headers: {
    'Authorization': `Bearer ${localStorage.getItem('access_token')}`
  }
};
```

### خدمات الواتساب

#### خدمة إرسال الرسائل (notifications/services.py)
```python
class WhatsAppService:
    def __init__(self, branch):
        self.config = WhatsAppConfig.objects.get(branch=branch)
    
    def send_message(self, phone_number, message):
        """إرسال رسالة واتساب"""
        url = f"{self.config.api_url}/messages"
        headers = {
            'Authorization': f'Bearer {self.config.access_token}',
            'Content-Type': 'application/json'
        }
        data = {
            'messaging_product': 'whatsapp',
            'to': phone_number,
            'text': {'body': message}
        }
        response = requests.post(url, headers=headers, json=data)
        return response.json()
    
    def send_template_message(self, phone_number, template_name, variables):
        """إرسال رسالة من قالب"""
        template = MessageTemplate.objects.get(name=template_name)
        message = template.content_ar.format(**variables)
        return self.send_message(phone_number, message)
```

## هندسة Frontend (React)

### بنية المشروع
```
src/
├── components/         # المكونات
│   ├── auth/          # مكونات المصادقة
│   ├── customers/     # مكونات العملاء
│   ├── products/      # مكونات المنتجات
│   ├── orders/        # مكونات الطلبات
│   └── ui/            # مكونات واجهة المستخدم
├── contexts/          # Context للحالة العامة
│   ├── AuthContext.jsx
│   └── ThemeContext.jsx
├── hooks/             # Custom Hooks
├── services/          # خدمات API
├── utils/             # أدوات مساعدة
└── App.jsx            # التطبيق الرئيسي
```

### إدارة الحالة مع Context

#### AuthContext (contexts/AuthContext.jsx)
```jsx
const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  const login = async (credentials) => {
    try {
      const response = await authService.login(credentials);
      setUser(response.data.user);
      localStorage.setItem('access_token', response.data.access);
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('access_token');
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
};
```

### خدمات API

#### خدمة العملاء (services/customerService.js)
```javascript
class CustomerService {
  constructor() {
    this.baseURL = 'http://localhost:8000/api';
  }

  async getCustomers(params = {}) {
    const response = await axios.get(`${this.baseURL}/customers/`, {
      params,
      headers: this.getAuthHeaders()
    });
    return response.data;
  }

  async createCustomer(customerData) {
    const response = await axios.post(`${this.baseURL}/customers/`, customerData, {
      headers: this.getAuthHeaders()
    });
    return response.data;
  }

  getAuthHeaders() {
    const token = localStorage.getItem('access_token');
    return token ? { Authorization: `Bearer ${token}` } : {};
  }
}

export default new CustomerService();
```

### دعم اللغة العربية

#### إعدادات RTL
```css
/* في App.css */
.rtl {
  direction: rtl;
  text-align: right;
}

.rtl .sidebar {
  right: 0;
  left: auto;
}

.rtl .main-content {
  margin-right: 250px;
  margin-left: 0;
}
```

#### مكون متعدد اللغات
```jsx
const LanguageProvider = ({ children }) => {
  const [language, setLanguage] = useState('ar');
  
  const t = (key) => {
    const translations = {
      ar: {
        'dashboard': 'لوحة المعلومات',
        'customers': 'العملاء',
        'products': 'المنتجات'
      },
      en: {
        'dashboard': 'Dashboard',
        'customers': 'Customers',
        'products': 'Products'
      }
    };
    return translations[language][key] || key;
  };

  return (
    <LanguageContext.Provider value={{ language, setLanguage, t }}>
      <div className={language === 'ar' ? 'rtl' : 'ltr'}>
        {children}
      </div>
    </LanguageContext.Provider>
  );
};
```

## أفضل الممارسات

### Backend (Django)

#### 1. تنظيم النماذج
```python
class BaseModel(models.Model):
    """نموذج أساسي لجميع النماذج"""
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
    
    class Meta:
        abstract = True
```

#### 2. استخدام Managers مخصصة
```python
class ActiveManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_active=True)

class Customer(BaseModel):
    # ... fields
    objects = models.Manager()  # المدير الافتراضي
    active = ActiveManager()    # المدير المخصص
```

#### 3. التحقق من البيانات
```python
class CustomerSerializer(serializers.ModelSerializer):
    def validate_phone(self, value):
        if not value.startswith('+966'):
            raise serializers.ValidationError('رقم الهاتف يجب أن يبدأ بـ +966')
        return value
    
    def validate(self, data):
        if data['credit_limit'] < 0:
            raise serializers.ValidationError('حد الائتمان لا يمكن أن يكون سالباً')
        return data
```

### Frontend (React)

#### 1. مكونات قابلة لإعادة الاستخدام
```jsx
const DataTable = ({ data, columns, onEdit, onDelete }) => {
  return (
    <div className="overflow-x-auto">
      <table className="min-w-full table-auto">
        <thead>
          <tr>
            {columns.map(col => (
              <th key={col.key} className="px-4 py-2 text-right">
                {col.title}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {data.map(row => (
            <tr key={row.id}>
              {columns.map(col => (
                <td key={col.key} className="px-4 py-2">
                  {row[col.key]}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};
```

#### 2. Custom Hooks
```jsx
const useApi = (url) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await axios.get(url);
        setData(response.data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
};
```

#### 3. معالجة الأخطاء
```jsx
const ErrorBoundary = ({ children }) => {
  const [hasError, setHasError] = useState(false);

  useEffect(() => {
    const handleError = (error) => {
      console.error('خطأ في التطبيق:', error);
      setHasError(true);
    };

    window.addEventListener('error', handleError);
    return () => window.removeEventListener('error', handleError);
  }, []);

  if (hasError) {
    return (
      <div className="text-center p-8">
        <h2 className="text-xl font-bold text-red-600">حدث خطأ غير متوقع</h2>
        <button 
          onClick={() => setHasError(false)}
          className="mt-4 px-4 py-2 bg-blue-500 text-white rounded"
        >
          إعادة المحاولة
        </button>
      </div>
    );
  }

  return children;
};
```

## الاختبارات

### اختبارات Backend (Django)

#### اختبار النماذج
```python
from django.test import TestCase
from users.models import Branch, Department

class BranchModelTest(TestCase):
    def setUp(self):
        self.branch = Branch.objects.create(
            name='الفرع الرئيسي',
            code='MAIN',
            address='الرياض',
            phone='+966501234567',
            email='main@company.com'
        )

    def test_branch_creation(self):
        self.assertEqual(self.branch.name, 'الفرع الرئيسي')
        self.assertEqual(self.branch.code, 'MAIN')
        self.assertTrue(self.branch.is_active)
```

#### اختبار APIs
```python
from rest_framework.test import APITestCase
from django.contrib.auth.models import User

class CustomerAPITest(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.client.force_authenticate(user=self.user)

    def test_create_customer(self):
        data = {
            'name': 'عميل تجريبي',
            'email': 'test@example.com',
            'phone': '+966501234567'
        }
        response = self.client.post('/api/customers/', data)
        self.assertEqual(response.status_code, 201)
```

### اختبارات Frontend (React)

#### اختبار المكونات
```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Login from '../components/auth/Login';

describe('Login Component', () => {
  test('renders login form', () => {
    render(<Login />);
    expect(screen.getByText('تسجيل الدخول')).toBeInTheDocument();
    expect(screen.getByPlaceholderText('اسم المستخدم')).toBeInTheDocument();
  });

  test('handles form submission', () => {
    const mockLogin = jest.fn();
    render(<Login onLogin={mockLogin} />);
    
    fireEvent.change(screen.getByPlaceholderText('اسم المستخدم'), {
      target: { value: 'testuser' }
    });
    fireEvent.click(screen.getByText('دخول'));
    
    expect(mockLogin).toHaveBeenCalled();
  });
});
```

## الأداء والتحسين

### تحسين Backend

#### 1. استخدام select_related و prefetch_related
```python
# بدلاً من
customers = Customer.objects.all()
for customer in customers:
    print(customer.branch.name)  # N+1 queries

# استخدم
customers = Customer.objects.select_related('branch').all()
for customer in customers:
    print(customer.branch.name)  # 1 query
```

#### 2. التخزين المؤقت
```python
from django.core.cache import cache

def get_dashboard_stats(branch_id):
    cache_key = f'dashboard_stats_{branch_id}'
    stats = cache.get(cache_key)
    
    if not stats:
        stats = calculate_dashboard_stats(branch_id)
        cache.set(cache_key, stats, 300)  # 5 دقائق
    
    return stats
```

### تحسين Frontend

#### 1. تحميل البيانات بشكل تدريجي
```jsx
const CustomersList = () => {
  const [customers, setCustomers] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);

  const loadMore = useCallback(async () => {
    setLoading(true);
    const newCustomers = await customerService.getCustomers({ page });
    setCustomers(prev => [...prev, ...newCustomers]);
    setPage(prev => prev + 1);
    setLoading(false);
  }, [page]);

  return (
    <InfiniteScroll
      dataLength={customers.length}
      next={loadMore}
      hasMore={true}
      loader={<div>جاري التحميل...</div>}
    >
      {customers.map(customer => (
        <CustomerCard key={customer.id} customer={customer} />
      ))}
    </InfiniteScroll>
  );
};
```

#### 2. تحسين الرسوم البيانية
```jsx
const SalesChart = memo(({ data }) => {
  const chartData = useMemo(() => {
    return data.map(item => ({
      name: item.month,
      sales: item.total_sales,
      target: item.target
    }));
  }, [data]);

  return (
    <ResponsiveContainer width="100%" height={300}>
      <LineChart data={chartData}>
        <XAxis dataKey="name" />
        <YAxis />
        <CartesianGrid strokeDasharray="3 3" />
        <Tooltip />
        <Line type="monotone" dataKey="sales" stroke="#8884d8" />
        <Line type="monotone" dataKey="target" stroke="#82ca9d" />
      </LineChart>
    </ResponsiveContainer>
  );
});
```

## الأمان المتقدم

### حماية CSRF
```python
# في settings.py
CSRF_TRUSTED_ORIGINS = [
    'https://your-domain.com',
    'http://localhost:3000'
]

# في views.py
from django.views.decorators.csrf import csrf_exempt
from django.utils.decorators import method_decorator

@method_decorator(csrf_exempt, name='dispatch')
class WebhookView(View):
    def post(self, request):
        # معالجة webhook
        pass
```

### تشفير البيانات الحساسة
```python
from cryptography.fernet import Fernet
from django.conf import settings

class EncryptedField(models.TextField):
    def __init__(self, *args, **kwargs):
        self.cipher = Fernet(settings.ENCRYPTION_KEY)
        super().__init__(*args, **kwargs)
    
    def from_db_value(self, value, expression, connection):
        if value is None:
            return value
        return self.cipher.decrypt(value.encode()).decode()
    
    def to_python(self, value):
        if isinstance(value, str):
            return value
        return self.cipher.decrypt(value.encode()).decode()
    
    def get_prep_value(self, value):
        return self.cipher.encrypt(value.encode()).decode()
```

## التوثيق التلقائي

### Swagger/OpenAPI
```python
# في settings.py
INSTALLED_APPS = [
    'drf_yasg',
]

# في urls.py
from drf_yasg.views import get_schema_view
from drf_yasg import openapi

schema_view = get_schema_view(
    openapi.Info(
        title="ERP Sales API",
        default_version='v1',
        description="واجهة برمجة تطبيقات نظام إدارة المبيعات",
    ),
    public=True,
)

urlpatterns = [
    path('swagger/', schema_view.with_ui('swagger', cache_timeout=0)),
    path('redoc/', schema_view.with_ui('redoc', cache_timeout=0)),
]
```

## المراقبة والسجلات

### إعداد السجلات
```python
# في settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': 'erp_sales.log',
        },
    },
    'loggers': {
        'sales': {
            'handlers': ['file'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}

# في views.py
import logging
logger = logging.getLogger('sales')

class CustomerViewSet(ModelViewSet):
    def create(self, request):
        logger.info(f'إنشاء عميل جديد بواسطة {request.user.username}')
        return super().create(request)
```

---

هذا الدليل يوفر فهماً شاملاً لهندسة النظام وأفضل الممارسات للتطوير والصيانة.

