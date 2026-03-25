# 🗺️ Tersa ERP — خريطة الطريق الكاملة للبناء الحقيقي

## Tech Stack المحدد

| الطبقة | التقنية | السبب |
|--------|---------|-------|
| Frontend | Next.js 14 + Tailwind CSS | SSR + سرعة عالية |
| Backend | Node.js + Express | مرونة + سرعة |
| Database | PostgreSQL + Prisma ORM | موثوقية + سهولة |
| Auth | JWT + bcrypt | أمان + سهولة |
| Hosting | Railway (DB) + Vercel (App) | مجاني في البداية |
| AI | Claude API (Anthropic) | أفضل تحليل عربي |

---

## 📁 هيكل المشروع الكامل

```
tersa-erp/
├── frontend/                    # Next.js App
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx       # Sidebar + Topbar
│   │   │   ├── page.tsx         # Dashboard
│   │   │   ├── finance/page.tsx
│   │   │   ├── employees/page.tsx
│   │   │   ├── inventory/page.tsx
│   │   │   ├── analytics/page.tsx
│   │   │   ├── reports/page.tsx
│   │   │   ├── shopify/page.tsx
│   │   │   └── settings/page.tsx
│   │   └── api/
│   │       └── ai/route.ts      # AI proxy
│   ├── components/
│   │   ├── ui/                  # Buttons, Inputs, Cards
│   │   ├── charts/              # Chart components
│   │   ├── tables/              # Data tables
│   │   └── modals/              # Modal components
│   └── lib/
│       ├── api.ts               # API client
│       └── auth.ts              # Auth helpers
│
├── backend/                     # Express API
│   ├── src/
│   │   ├── routes/
│   │   │   ├── auth.routes.ts
│   │   │   ├── transactions.routes.ts
│   │   │   ├── employees.routes.ts
│   │   │   ├── inventory.routes.ts
│   │   │   ├── shopify.routes.ts
│   │   │   └── reports.routes.ts
│   │   ├── middleware/
│   │   │   ├── auth.middleware.ts    # JWT verify
│   │   │   ├── rbac.middleware.ts    # Role check
│   │   │   └── audit.middleware.ts  # Log actions
│   │   ├── services/
│   │   │   ├── shopify.service.ts
│   │   │   ├── bosta.service.ts
│   │   │   ├── ai.service.ts
│   │   │   └── reports.service.ts
│   │   └── webhooks/
│   │       └── shopify.webhook.ts
│   └── prisma/
│       └── schema.prisma
```

---

## 🗄️ Database Schema (PostgreSQL)

```prisma
// prisma/schema.prisma

model Company {
  id          String   @id @default(cuid())
  name        String
  currency    String   @default("EGP")
  shopifyUrl  String?
  shopifyKey  String?
  bostaKey    String?
  createdAt   DateTime @default(now())
  
  users         User[]
  transactions  Transaction[]
  employees     Employee[]
  products      Product[]
  orders        Order[]
}

model User {
  id        String   @id @default(cuid())
  name      String
  email     String   @unique
  password  String
  role      Role     @default(EMPLOYEE)
  companyId String
  company   Company  @relation(fields:[companyId], references:[id])
  logs      AuditLog[]
  createdAt DateTime @default(now())
}

enum Role {
  ADMIN
  MANAGER
  ACCOUNTANT
  EMPLOYEE
}

model Transaction {
  id          String   @id @default(cuid())
  description String
  category    String
  amount      Float
  type        TxType
  date        DateTime
  companyId   String
  company     Company  @relation(fields:[companyId], references:[id])
  createdAt   DateTime @default(now())
}

enum TxType {
  REVENUE
  EXPENSE
}

model Employee {
  id          String   @id @default(cuid())
  name        String
  role        String
  salary      Float
  startDate   DateTime
  attendance  Int      @default(100)
  companyId   String
  company     Company  @relation(fields:[companyId], references:[id])
}

model Product {
  id          String   @id @default(cuid())
  name        String
  category    String
  stock       Int
  costPrice   Float
  sellPrice   Float
  supplier    String?
  minStock    Int      @default(20)
  companyId   String
  company     Company  @relation(fields:[companyId], references:[id])
}

model Order {
  id          String      @id @default(cuid())
  shopifyId   String?
  customer    String
  amount      Float
  status      OrderStatus @default(PENDING)
  bostaId     String?
  companyId   String
  company     Company     @relation(fields:[companyId], references:[id])
  createdAt   DateTime    @default(now())
}

enum OrderStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  RETURNED
}

model AuditLog {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields:[userId], references:[id])
  action    String
  details   String?
  createdAt DateTime @default(now())
}
```

---

## 🔌 API Endpoints الكاملة

### Auth
```
POST   /api/auth/register     → إنشاء حساب
POST   /api/auth/login        → تسجيل دخول → JWT
GET    /api/auth/me           → بيانات المستخدم الحالي
POST   /api/auth/logout       → تسجيل خروج
```

### Transactions
```
GET    /api/transactions              → كل المعاملات
POST   /api/transactions              → إضافة معاملة
PUT    /api/transactions/:id          → تعديل
DELETE /api/transactions/:id          → حذف
GET    /api/transactions/export/csv   → تصدير CSV
GET    /api/transactions/export/pdf   → تصدير PDF
GET    /api/transactions/summary      → ملخص (إيرادات/مصروفات/ربح)
```

### Employees
```
GET    /api/employees          → كل الموظفين
POST   /api/employees          → إضافة موظف
PUT    /api/employees/:id      → تعديل
DELETE /api/employees/:id      → حذف
GET    /api/employees/payroll  → كشف الرواتب
```

### Inventory
```
GET    /api/inventory              → كل المنتجات
POST   /api/inventory              → إضافة منتج
PUT    /api/inventory/:id          → تعديل
DELETE /api/inventory/:id          → حذف
GET    /api/inventory/low-stock    → المنتجات الأقل من الحد
PUT    /api/inventory/:id/stock    → تحديث المخزون
```

### Shopify Integration
```
POST   /api/shopify/connect         → ربط Shopify
GET    /api/shopify/orders          → جلب الأوردرات
POST   /api/shopify/sync            → مزامنة يدوية
POST   /api/shopify/webhook         → webhook من Shopify
GET    /api/shopify/products        → جلب المنتجات
```

### Bosta Integration
```
POST   /api/bosta/create-shipment   → إنشاء شحنة
GET    /api/bosta/track/:id         → تتبع شحنة
GET    /api/bosta/shipments         → كل الشحنات
```

### Reports
```
GET    /api/reports/monthly    → تقرير شهري
GET    /api/reports/annual     → تقرير سنوي
GET    /api/reports/profit     → قائمة أرباح وخسائر
GET    /api/reports/cashflow   → تدفق نقدي
POST   /api/reports/export     → تصدير PDF
```

### AI
```
POST   /api/ai/chat            → محادثة مع AI
POST   /api/ai/insights        → تحليل تلقائي
POST   /api/ai/forecast        → توقع الإيرادات
```

---

## 🔗 Shopify + Bosta Integration (الكود الحقيقي)

```typescript
// backend/src/services/shopify.service.ts
import Shopify from '@shopify/shopify-api';

export async function getShopifyOrders(company: Company) {
  const client = new Shopify.Clients.Rest(company.shopifyUrl!, company.shopifyKey!);
  const orders = await client.get({ path: 'orders', query: { status: 'any', limit: 250 } });
  return orders.body.orders;
}

export async function handleNewOrder(order: any, companyId: string) {
  // 1. حفظ الأوردر في الداتابيز
  await prisma.order.create({
    data: {
      shopifyId: order.id.toString(),
      customer: order.customer?.first_name + ' ' + order.customer?.last_name,
      amount: parseFloat(order.total_price),
      status: 'PROCESSING',
      companyId
    }
  });

  // 2. إضافة إيراد تلقائي
  await prisma.transaction.create({
    data: {
      description: `أوردر Shopify #${order.order_number}`,
      category: 'مبيعات',
      amount: parseFloat(order.total_price),
      type: 'REVENUE',
      date: new Date(),
      companyId
    }
  });

  // 3. إنشاء شحنة على Bosta تلقائياً
  await createBostaShipment(order, companyId);
}
```

```typescript
// backend/src/services/bosta.service.ts
export async function createBostaShipment(order: any, companyId: string) {
  const company = await prisma.company.findUnique({ where: { id: companyId } });
  
  const response = await fetch('https://app.bosta.co/api/v2/deliveries', {
    method: 'POST',
    headers: {
      'Authorization': company!.bostaKey!,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      type: 10, // توصيل
      specs: { packageType: 'Parcel', size: 'SMALL' },
      receiver: {
        firstName: order.shipping_address.first_name,
        lastName: order.shipping_address.last_name,
        phone: order.phone,
        address: {
          city: order.shipping_address.city,
          firstLine: order.shipping_address.address1
        }
      },
      cod: parseFloat(order.total_price)
    })
  });
  
  const shipment = await response.json();
  
  // حفظ Bosta ID في الأوردر
  await prisma.order.update({
    where: { shopifyId: order.id.toString() },
    data: { bostaId: shipment._id }
  });
}
```

---

## 🚀 خطوات النشر (Step by Step)

### الخطوة 1: إعداد البيئة
```bash
# 1. تثبيت Node.js 20+ من nodejs.org

# 2. إنشاء المشروع
mkdir tersa-erp && cd tersa-erp
npm init -y

# 3. Backend packages
npm install express prisma @prisma/client bcryptjs jsonwebtoken
npm install @shopify/shopify-api dotenv cors helmet
npm install -D typescript ts-node @types/node

# 4. Frontend
npx create-next-app@latest frontend --typescript --tailwind
```

### الخطوة 2: إعداد PostgreSQL
```bash
# على Railway.app (مجاناً):
# 1. روح railway.app
# 2. New Project → Database → PostgreSQL
# 3. انسخ DATABASE_URL

# تشغيل Prisma
npx prisma init
npx prisma migrate dev --name init
npx prisma generate
```

### الخطوة 3: متغيرات البيئة
```env
# .env
DATABASE_URL="postgresql://user:pass@host:5432/tersa_erp"
JWT_SECRET="your-super-secret-key-here"
SHOPIFY_API_KEY="your-shopify-key"
SHOPIFY_API_SECRET="your-shopify-secret"
BOSTA_API_KEY="your-bosta-key"
ANTHROPIC_API_KEY="your-claude-key"
PORT=3001
```

### الخطوة 4: النشر
```bash
# Backend على Railway
railway login
railway up

# Frontend على Vercel
npx vercel --prod
```

---

## 💰 خطة التكاليف

| الخدمة | السعر |
|--------|-------|
| Railway (DB + Backend) | مجاني حتى $5/شهر |
| Vercel (Frontend) | مجاني |
| Shopify Partner (اختبار) | مجاني |
| Claude API | ~$5-20/شهر حسب الاستخدام |
| **الإجمالي في البداية** | **$0-25/شهر** |

---

## 📅 خطة التطوير (12 أسبوع)

| الأسبوع | المهام |
|---------|--------|
| 1-2 | إعداد المشروع + Auth + Database |
| 3-4 | Financial Management (CRUD) |
| 5-6 | Employee + Inventory Management |
| 7-8 | Shopify + Bosta Integration |
| 9-10 | Analytics + Reports + AI |
| 11 | Notifications + Automations |
| 12 | Testing + Deployment + Launch |

---

## 🎯 الأولويات الذكية

**ابدأ بده:**
1. Auth + Database Schema
2. Financial Management (الأهم)
3. Shopify Integration (ده بيوفر وقت)

**اتركه للآخر:**
- AI Insights (بعد ما عندك بيانات حقيقية)
- Multi-company (بعد ما النظام الأساسي شغال)
- PWA Mobile

---

*Tersa ERP Roadmap v1.0 — Built for Youssef Elsmeen*
