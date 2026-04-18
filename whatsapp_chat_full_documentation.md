# وثيقة التصميم والتطوير الكاملة
# نظام الدردشة عبر WhatsApp — نفوذ التطوير
# WhatsApp Chat System — Full Technical Documentation

**الإصدار:** 1.0
**التاريخ:** أبريل 2026
**الشركة:** نفوذ التطوير للعقارات وإدارة الأملاك
**التقنيات:** Frappe Framework + Vue.js 3 + WhatsApp Business API

---

# الجزء الأول: نظرة عامة ومتطلبات العمل

## 1. مقدمة

### 1.1 الهدف من المشروع
بناء واجهة دردشة متكاملة داخل نظام نفوذ (Frappe/ERPNext) تتيح للموظفين الرد على رسائل WhatsApp، تتبع المحادثات، توزيعها تلقائياً، وقياس أداء الاستجابة — بديلاً عن استخدام WhatsApp Web مباشرة.

### 1.2 المشكلة الحالية
- الموظفون يردون من WhatsApp Web مباشرة خارج النظام
- لا يوجد تتبع لمن فتح أو رد على الرسالة
- لا يوجد توزيع تلقائي للرسائل على الموظفين
- لا يوجد تصعيد للرسائل المتأخرة
- لا توجد إحصائيات أو KPIs لقياس الأداء
- 99.8% من الرسائل الواردة لم يشاهدها أي موظف من داخل النظام

### 1.3 الأرقام الحالية
- 69,112 رسالة صادرة
- 4,221 رسالة واردة
- 0 رسالة مُكلَّف بها
- 9 رسائل فقط شاهدها موظف حقيقي
- حالات الرسائل الصادرة: 36% مقروءة، 18% مُسلَّمة، 18% فاشلة، 26% بدون حالة

### 1.4 الحالة المستهدفة
- واجهة شبيهة بـ WhatsApp Web / تطبيق ريل داخل النظام
- توزيع تلقائي ذكي للرسائل على الموظفين
- تتبع كامل: من فتح، من رد، وقت الاستجابة
- تصعيد تلقائي متدرج للرسائل المتأخرة
- Dashboard شامل لقياس الأداء
- إشعارات فورية (Real-time) مع صوت تنبيه

---

## 2. متطلبات العمل (Business Requirements)

### 2.1 متطلبات وظيفية

#### 2.1.1 إدارة المحادثات
- FR-01: عرض قائمة المحادثات مرتبة حسب آخر رسالة
- FR-02: فلترة المحادثات (الكل / غير مقروء / لي فقط / متأخرة)
- FR-03: البحث في المحادثات بالاسم أو الرقم
- FR-04: عرض سجل المحادثة الكامل مع العميل
- FR-05: دعم أنواع الرسائل: نص، صورة، مستند، صوت، فيديو، موقع
- FR-06: إمكانية الرد المباشر من داخل النظام
- FR-07: دعم الردود السريعة (Quick Reply Templates)
- FR-08: إمكانية إرفاق ملفات مع الرد

#### 2.1.2 التوزيع والتعيين
- FR-09: تعيين تلقائي للرسائل الواردة حسب قواعد محددة
- FR-10: تعيين يدوي / إعادة تعيين من المشرف
- FR-11: توزيع Round Robin على أعضاء الفريق
- FR-12: ربط تلقائي بالمستأجر/المالك/العقار من WhatsApp Contact

#### 2.1.3 التتبع والمراقبة
- FR-13: تسجيل أول موظف فتح الرسالة + الوقت
- FR-14: تسجيل الموظف الذي رد + الوقت
- FR-15: حساب وقت الاستجابة تلقائياً
- FR-16: تتبع حالة المحادثة (نشطة / في انتظار / تم الرد / مُصعَّدة / مغلقة)

#### 2.1.4 التصعيد
- FR-17: تصعيد تلقائي بعد 2 ساعة بدون رد → إشعار للموظف
- FR-18: تصعيد بعد 3 ساعات → إشعار لقائد الفريق
- FR-19: تصعيد بعد 4 ساعات → إشعار للمدير العام
- FR-20: أوقات التصعيد قابلة للتعديل من الإعدادات

#### 2.1.5 Dashboard والتقارير
- FR-21: عدد الرسائل التي تنتظر رد
- FR-22: متوسط وقت الاستجابة
- FR-23: نسبة الرد
- FR-24: أداء كل موظف (مستلم / رد / متوسط وقت / مفتوح / متأخر)
- FR-25: رسم بياني للرسائل بالساعة/اليوم/الشهر

### 2.2 متطلبات غير وظيفية
- NFR-01: الواجهة تعمل بسلاسة مع 73,000+ رسالة
- NFR-02: تحميل المحادثات على دفعات (Infinite Scroll)
- NFR-03: تحديث فوري بدون إعادة تحميل الصفحة (Real-time)
- NFR-04: دعم اللغة العربية (RTL) بالكامل
- NFR-05: متوافق مع Chrome/Edge/Firefox
- NFR-06: صلاحيات حسب الدور (Agent/Supervisor/Admin)
- NFR-07: أداء سريع — تحميل أقل من 3 ثوانٍ

---

# الجزء الثاني: التصميم التقني (Technical Design)

## 3. البنية التقنية (Architecture)

### 3.1 نظرة عامة على البنية

```
┌─────────────────────────────────────────────────────────┐
│                    المتصفح (Browser)                      │
│  ┌───────────────────────────────────────────────────┐  │
│  │              Vue.js 3 Application                  │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────────────┐   │  │
│  │  │Conversa- │ │  Chat    │ │  Contact Info    │   │  │
│  │  │tion List │ │  Window  │ │  Panel           │   │  │
│  │  └──────────┘ └──────────┘ └──────────────────┘   │  │
│  │  ┌──────────────────────────────────────────────┐  │  │
│  │  │           Pinia State Management              │  │  │
│  │  └──────────────────────────────────────────────┘  │  │
│  │  ┌──────────────────────────────────────────────┐  │  │
│  │  │        Frappe Bridge (API + Realtime)          │  │  │
│  │  └──────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────┘
                         │ HTTP/WebSocket
┌────────────────────────┴────────────────────────────────┐
│                  Frappe Backend (Python)                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐ │
│  │   API    │ │Assignment│ │Escalation│ │  Realtime   │ │
│  │Endpoints │ │  Engine  │ │  Engine  │ │  Publisher  │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────────┘ │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Frappe ORM / MariaDB                 │   │
│  └──────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────┐
│              WhatsApp Business API (Meta)                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                │
│  │  Send    │ │ Receive  │ │ Webhook  │                │
│  │ Messages │ │ Messages │ │ Status   │                │
│  └──────────┘ └──────────┘ └──────────┘                │
└─────────────────────────────────────────────────────────┘
```

### 3.2 مكونات النظام

| المكون | التقنية | الوصف |
|---|---|---|
| Frontend | Vue.js 3 + Vite | واجهة المستخدم التفاعلية |
| State Management | Pinia | إدارة حالة المحادثات والرسائل |
| CSS Framework | Tailwind CSS | تنسيق سريع ومتجاوب |
| Backend API | Frappe Python | نقاط النهاية للبيانات |
| Database | MariaDB (via Frappe ORM) | تخزين البيانات |
| Real-time | Frappe Realtime (Socket.io) | تحديثات فورية |
| WhatsApp API | Meta Cloud API | إرسال/استقبال الرسائل |
| Scheduled Jobs | Frappe Scheduler | التصعيد الدوري |

### 3.3 تدفق البيانات (Data Flow)

#### رسالة واردة:
```
WhatsApp User → Meta API → Webhook → Frappe Backend
    → Create WhatsApp Message (Incoming)
    → Find/Create WhatsApp Conversation
    → Apply Assignment Rules
    → Publish Realtime Event
    → Vue App receives event → Update UI instantly
    → Browser Notification + Sound
```

#### رسالة صادرة (رد):
```
Employee types reply in Vue App
    → frappe.call('send_reply', {message, conversation})
    → Create WhatsApp Message (Outgoing)
    → Meta API sends to WhatsApp User
    → Update conversation status → Replied
    → Record replied_by + replied_at
    → Calculate response_time_mins
    → Publish Realtime → Update all connected clients
```

---

## 4. هيكل قاعدة البيانات (Database Schema)

### 4.1 تعديل WhatsApp Message (Custom Fields)

DocType: WhatsApp Message (موجود — إضافة Custom Fields)

| الحقل | الاسم التقني | النوع | إلزامي | Read-Only | الوصف |
|---|---|---|---|---|---|
| معرّف المحادثة | conversation_id | Data | لا | لا | ربط الرسالة بمحادثة |
| مُعيَّن لـ | custom_assigned_to | Link → User | لا | لا | الموظف المكلف |
| وقت التعيين | custom_assigned_at | Datetime | لا | نعم | وقت التكليف التلقائي |
| أول قراءة بواسطة | custom_first_read_by | Link → User | لا | نعم | أول من فتح الرسالة |
| وقت أول قراءة | custom_first_read_at | Datetime | لا | نعم | متى فُتحت أول مرة |
| الرد بواسطة | custom_replied_by | Link → User | لا | نعم | من رد على الرسالة |
| وقت الرد | custom_replied_at | Datetime | لا | نعم | متى تم الرد |
| وقت الاستجابة | custom_response_time | Int | لا | نعم | بالدقائق (محسوب) |
| حالة المتابعة | custom_follow_status | Select | لا | لا | Pending/Read/Replied/Escalated |
| تم التصعيد | custom_escalation_sent | Check | لا | لا | هل أُرسل إشعار تصعيد |

### 4.2 WhatsApp Conversation (جديد)

DocType: WhatsApp Conversation
Naming Rule: WCONV-.YYYY.-.#####

| الحقل | الاسم التقني | النوع | إلزامي | الوصف |
|---|---|---|---|---|
| جهة الاتصال | contact | Link → WhatsApp Contact | لا | الكونتاكت المرتبط |
| رقم الهاتف | phone_number | Data | نعم | رقم العميل |
| اسم العميل | contact_name | Data | لا | الاسم (من الكونتاكت أو يدوي) |
| الموظف المسؤول | assigned_to | Link → User | لا | المُعيَّن حالياً |
| الحالة | status | Select | نعم | Active/Pending/Replied/Escalated/Closed |
| الأولوية | priority | Select | لا | Low/Normal/High/Urgent |
| وقت آخر رسالة | last_message_at | Datetime | لا | تاريخ آخر رسالة |
| معاينة آخر رسالة | last_message_preview | Small Text | لا | أول 100 حرف |
| نوع آخر رسالة | last_message_type | Select | لا | Incoming/Outgoing |
| غير مقروءة | unread_count | Int | لا | عدد الرسائل غير المقروءة |
| المستأجر | linked_tenant | Link → Tenant | لا | ربط تلقائي |
| المالك | linked_owner | Link → Property Owner | لا | ربط تلقائي |
| العقار | linked_property | Link → Property | لا | ربط تلقائي |
| وقت أول رد | first_response_time | Int | لا | بالدقائق |
| إجمالي الرسائل | total_messages | Int | لا | عدد كلي |
| تصنيف | tags | Small Text | لا | تصنيفات مخصصة |
| ملاحظات داخلية | internal_notes | Text | لا | ملاحظات الموظفين |

### 4.3 WhatsApp Team (جديد)

DocType: WhatsApp Team

| الحقل | الاسم التقني | النوع | إلزامي | الوصف |
|---|---|---|---|---|
| اسم الفريق | team_name | Data | نعم | مثل: فريق إدارة الأملاك |
| قائد الفريق | team_lead | Link → User | نعم | المسؤول عن التصعيد |
| التصعيد بعد (ساعات) | escalation_after_hrs | Float | نعم | default: 2 |
| التصعيد المرحلة 2 | escalation_stage2_hrs | Float | لا | default: 3 |
| التصعيد المرحلة 3 | escalation_stage3_hrs | Float | لا | default: 4 |
| الفريق الافتراضي | is_default | Check | لا | فريق واحد فقط |
| الأعضاء | members | Table | نعم | WhatsApp Team Member |

### 4.4 WhatsApp Team Member (جديد — Child Table)

| الحقل | الاسم التقني | النوع | الوصف |
|---|---|---|---|
| الموظف | user | Link → User | عضو الفريق |
| نشط | is_active | Check | هل يستقبل رسائل |
| الحد الأقصى | max_conversations | Int | أقصى عدد محادثات نشطة |

### 4.5 WhatsApp Assignment Rule (جديد)

DocType: WhatsApp Assignment Rule

| الحقل | الاسم التقني | النوع | الوصف |
|---|---|---|---|
| اسم القاعدة | rule_name | Data | وصف القاعدة |
| نوع الشرط | condition_type | Select | Keyword/Contact/Property/Round Robin |
| الكلمة المفتاحية | keyword | Data | للبحث في نص الرسالة |
| تعيين لموظف | assign_to_user | Link → User | موظف محدد |
| تعيين لفريق | assign_to_team | Link → WhatsApp Team | توزيع على الفريق |
| الأولوية | priority | Int | ترتيب تطبيق القواعد (1 = أولاً) |
| مفعّل | is_active | Check | تفعيل/تعطيل القاعدة |

### 4.6 WhatsApp Quick Reply (جديد)

DocType: WhatsApp Quick Reply

| الحقل | الاسم التقني | النوع | الوصف |
|---|---|---|---|
| العنوان | title | Data | اسم الرد السريع |
| الاختصار | shortcut | Data | مثل /greeting |
| نص الرسالة | message_body | Text | محتوى الرد |
| التصنيف | category | Select | تحية/متابعة/دفعات/صيانة/عام |
| اللغة | language | Select | عربي/إنجليزي |

---

# الجزء الثالث: تصميم الواجهة (UI/UX Design)

## 5. الواجهة الرئيسية

### 5.1 التخطيط العام (Layout)

الصفحة مقسمة إلى 3 أعمدة رئيسية بنمط RTL:

```
┌─────────────────────────────────────────────────────────────────────────┐
│  HEADER BAR                                                             │
│  ┌─────┐                                                               │
│  │ 📱  │  WhatsApp Inbox    [🔍]  [الكل|غير مقروء|لي|متأخرة]  [⚙]     │
│  └─────┘                                                               │
├────────────────┬────────────────────────────────┬───────────────────────┤
│                │                                │                       │
│  CONVERSATIONS │        CHAT WINDOW             │   CONTACT INFO        │
│  PANEL         │                                │   PANEL               │
│                │                                │                       │
│  Width: 300px  │     Width: flex (expand)       │   Width: 300px        │
│  Min: 250px    │     Min: 400px                 │   Min: 250px          │
│                │                                │   Collapsible         │
│                │                                │                       │
│  ┌──────────┐  │  ┌────────────────────────┐    │  ┌─────────────────┐  │
│  │ Search   │  │  │  Chat Header           │    │  │ Contact Card    │  │
│  │ Bar      │  │  │  Name + Status          │    │  │                 │  │
│  ├──────────┤  │  ├────────────────────────┤    │  ├─────────────────┤  │
│  │          │  │  │                        │    │  │ Quick Actions   │  │
│  │ Contact  │  │  │  Messages Area         │    │  │                 │  │
│  │ List     │  │  │  (Scrollable)          │    │  ├─────────────────┤  │
│  │          │  │  │                        │    │  │ Statistics      │  │
│  │ Infinite │  │  │  ┌─── Incoming ───┐    │    │  │                 │  │
│  │ Scroll   │  │  │  │ Message bubble │    │    │  ├─────────────────┤  │
│  │          │  │  │  └────────────────┘    │    │  │ Internal Notes  │  │
│  │          │  │  │                        │    │  │                 │  │
│  │          │  │  │  ┌─── Outgoing ───┐    │    │  │                 │  │
│  │          │  │  │  │ Message bubble │    │    │  │                 │  │
│  │          │  │  │  └────────────────┘    │    │  │                 │  │
│  │          │  │  │                        │    │  │                 │  │
│  │          │  │  ├────────────────────────┤    │  │                 │  │
│  │          │  │  │  Reply Box             │    │  │                 │  │
│  │          │  │  │  [📎] [📷] [/] [Send]  │    │  │                 │  │
│  └──────────┘  │  └────────────────────────┘    │  └─────────────────┘  │
└────────────────┴────────────────────────────────┴───────────────────────┘
```

### 5.2 قائمة المحادثات (Conversations Panel)

#### عنصر المحادثة الواحدة:
```
┌─────────────────────────────────────────┐
│                                         │
│  🟢  أحمد محمد الفهد            10:30  │
│                                         │
│      مرحبا أريد الاستفسار عن...   [3]  │
│                                         │
│      👤 سارة  │  🏢 N4900  │  ⚡ عالي  │
│                                         │
└─────────────────────────────────────────┘
```

#### عناصر العرض:
- دائرة الحالة: 🟢 تم الرد / 🟡 في انتظار / 🔴 متأخر / ⚪ مغلق
- اسم العميل: من WhatsApp Contact أو الرقم مباشرة
- معاينة الرسالة: أول 40 حرف من آخر رسالة
- الوقت: وقت آخر رسالة (نسبي: الآن، 5 د، أمس)
- Badge: عدد غير المقروءة [3]
- معلومات إضافية: الموظف المعيّن، العقار، الأولوية

#### ألوان الحالة:
| اللون | الحالة | الوصف |
|---|---|---|
| 🟢 أخضر | Replied | تم الرد — آخر رسالة صادرة |
| 🟡 أصفر | Pending/Active | في انتظار رد — آخر رسالة واردة |
| 🔴 أحمر | Escalated | متأخرة — تجاوزت وقت التصعيد |
| ⚪ رمادي | Closed | مغلقة |
| 🔵 أزرق | New | محادثة جديدة لم تُفتح |

#### فلاتر التبويب:
- الكل: جميع المحادثات غير المغلقة
- غير مقروء: unread_count > 0
- لي فقط: assigned_to = current_user
- متأخرة: status = Escalated
- مغلقة: status = Closed (مخفي افتراضياً)

### 5.3 نافذة المحادثة (Chat Window)

#### الهيدر:
```
┌───────────────────────────────────────────────────────────┐
│  ← رجوع   أحمد محمد الفهد                               │
│            +966501234567                                   │
│            مُعيَّن: سارة │ ● Active │ ⏱ 15 دقيقة          │
└───────────────────────────────────────────────────────────┘
```

#### فقاعات الرسائل:

رسالة واردة (يمين — خلفية #dcf8c6):
```
                              ┌──────────────────────────┐
                              │                          │
                              │  مرحبا أريد الاستفسار   │
                              │  عن إيجار الوحدة في     │
                              │  عمارة الأمير تركي       │
                              │                          │
                              │              10:30 AM    │
                              └──────────────────────────┘
```

رسالة صادرة (يسار — خلفية #ffffff):
```
┌──────────────────────────┐
│                          │
│  حسناً سأتواصل معك      │
│  خلال ساعة لتحديد        │
│  موعد المعاينة           │
│                          │
│  10:45 AM  ✓✓  سارة      │
└──────────────────────────┘
```

#### أنواع المحتوى المدعومة:
| النوع | العرض | الإجراء |
|---|---|---|
| نص | عرض مباشر مع دعم الروابط | نسخ النص |
| صورة | Thumbnail 200x200 | تكبير (Lightbox) |
| فيديو | Thumbnail مع أيقونة تشغيل | تشغيل (Modal) |
| مستند | أيقونة PDF/DOC + اسم الملف | تحميل |
| صوت | مشغّل صوت مضمّن | تشغيل/إيقاف |
| موقع | خريطة مصغرة | فتح Google Maps |
| جهة اتصال | اسم + رقم | إضافة للكونتاكت |

#### فاصل التاريخ:
```
─────────── 18 أبريل 2026 ───────────
```

### 5.4 صندوق الرد (Reply Box)

```
┌──────────────────────────────────────────────────────────┐
│  ┌────────────────────────────────────────────────────┐  │
│  │                                                    │  │
│  │  اكتب رسالة... (اكتب / للردود السريعة)            │  │
│  │                                                    │  │
│  │                                                    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  [📎 مرفق]  [📷 صورة]  [📋 قالب]          [إرسال 📤]    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

#### الردود السريعة (عند كتابة /):
```
┌──────────────────────────────────────────────────────────┐
│  📋 الردود السريعة                              [✕]     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  /greeting   مرحباً بك في نفوذ التطوير، كيف يمكننا     │
│              مساعدتك؟                                    │
│                                                          │
│  /payment    يمكنكم السداد عبر منصة إيجار من خلال       │
│              رابط الدفع الالكتروني أو عبر تطبيق البنك   │
│                                                          │
│  /hours      أوقات العمل من السبت إلى الخميس من         │
│              الساعة 9:00 صباحاً إلى 10:00 مساءً         │
│                                                          │
│  /location   موقع مكتبنا: [رابط Google Maps]             │
│                                                          │
│  /thanks     شكراً لتواصلكم مع نفوذ التطوير             │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

#### اختصارات لوحة المفاتيح:
| الاختصار | الإجراء |
|---|---|
| Ctrl + Enter | إرسال الرسالة |
| / | فتح الردود السريعة |
| Esc | إغلاق الردود السريعة |
| ↑ / ↓ | التنقل في قائمة الردود |
| Enter | اختيار الرد السريع |

### 5.5 لوحة معلومات العميل (Contact Info Panel)

```
┌─────────────────────────────────┐
│  👤 بيانات جهة الاتصال          │
├─────────────────────────────────┤
│                                 │
│  الاسم:    أحمد محمد الفهد      │
│  الهاتف:   +966501234567        │
│  النوع:    مستأجر               │
│  العقار:   N4900 الأمير تركي    │
│  الوحدة:   شقة 5                │
│  المالك:   عبدالملك آل الشيخ    │
│  مسؤول العقار: أحمد بكر         │
│                                 │
├─────────────────────────────────┤
│  ⚡ إجراءات سريعة               │
│                                 │
│  ┌───────────────────────────┐  │
│  │ تعيين لموظف          ▼   │  │
│  └───────────────────────────┘  │
│  ┌───────────────────────────┐  │
│  │ تغيير الأولوية       ▼   │  │
│  └───────────────────────────┘  │
│                                 │
│  [🏢 ربط بعقار]                │
│  [👤 فتح سجل العميل]           │
│  [📋 فتح طلب صيانة]            │
│  [📄 فتح طلب عميل CRM]         │
│  [🔒 إغلاق المحادثة]           │
│                                 │
├─────────────────────────────────┤
│  📊 إحصائيات المحادثة           │
│                                 │
│  وقت الاستجابة:  15 دقيقة      │
│  الرسائل:        8 واردة       │
│                   5 صادرة       │
│  المحادثة منذ:    2 ساعة        │
│  محادثات سابقة:   3             │
│                                 │
├─────────────────────────────────┤
│  🏷️ التصنيفات                   │
│                                 │
│  [يحتاج متابعة] [مستأجر]       │
│  [+ إضافة تصنيف]               │
│                                 │
├─────────────────────────────────┤
│  📝 ملاحظات داخلية              │
│                                 │
│  ┌───────────────────────────┐  │
│  │ أضف ملاحظة...            │  │
│  └───────────────────────────┘  │
│                                 │
│  • سارة (10:45):               │
│    العميل يسأل عن تجديد العقد  │
│                                 │
│  • محمد (أمس):                  │
│    تم إرسال عرض السعر           │
│                                 │
└─────────────────────────────────┘
```

---

## 6. Dashboard

### 6.1 البطاقات العلوية (KPIs)

```
┌────────────┬────────────┬────────────┬────────────┬────────────┐
│            │            │            │            │            │
│  📩 47     │  ⏱ 23 د   │  ✅ 89%    │  🔴 8      │  📊 156    │
│  تنتظر    │  متوسط     │  نسبة     │  متأخرة   │  اليوم     │
│  رد        │  استجابة   │  الرد     │ (escalated)│  (إجمالي)  │
│            │            │            │            │            │
└────────────┴────────────┴────────────┴────────────┴────────────┘
```

### 6.2 رسم بياني: الرسائل بالساعة

```
عدد الرسائل
50 │
   │         ██
40 │         ██  ██
   │    ██   ██  ██
30 │    ██   ██  ██  ██
   │    ██   ██  ██  ██
20 │ ██ ██   ██  ██  ██  ██
   │ ██ ██   ██  ██  ██  ██  ██
10 │ ██ ██   ██  ██  ██  ██  ██  ██
   │ ██ ██   ██  ██  ██  ██  ██  ██
 0 └────────────────────────────────
    8   9   10  11  12  13  14  15
    
    ■ واردة (أخضر)    ■ صادرة (أزرق)
```

### 6.3 جدول أداء الموظفين

```
┌──────────────┬────────┬────────┬─────────────┬────────┬──────────┐
│   الموظف     │ مستلم  │  رد    │ متوسط وقت  │ مفتوح  │  متأخر  │
├──────────────┼────────┼────────┼─────────────┼────────┼──────────┤
│ 🟢 سارة     │   45   │   40   │    18 د     │   5    │    0     │
│ 🟡 محمد     │   30   │   25   │    35 د     │   5    │    2     │
│ 🟢 أحمد     │   22   │   20   │    12 د     │   2    │    0     │
│ 🔴 فاطمة    │   18   │   15   │    45 د     │   3    │    1     │
│ 🟢 خالد     │   15   │   14   │    20 د     │   1    │    0     │
└──────────────┴────────┴────────┴─────────────┴────────┴──────────┘
```

### 6.4 توزيع حسب الحالة

```
Active    ████████████████████     45%
Pending   ██████████                25%
Replied   ██████                    15%
Escalated ████                      10%
Closed    ██                         5%
```

---

# الجزء الرابع: التطوير التقني (Technical Implementation)

## 7. هيكل المشروع (Project Structure)

### 7.1 ملفات المشروع الكاملة

```
whatsapp_inbox/
│
├── frontend/                           ← Vue.js 3 Application
│   ├── src/
│   │   ├── App.vue                     ← المكون الرئيسي
│   │   ├── main.js                     ← نقطة الدخول
│   │   │
│   │   ├── components/                 ← مكونات Vue
│   │   │   ├── layout/
│   │   │   │   ├── AppHeader.vue       ← الشريط العلوي
│   │   │   │   └── AppLayout.vue       ← التخطيط الثلاثي
│   │   │   │
│   │   │   ├── conversations/
│   │   │   │   ├── ConversationList.vue    ← قائمة المحادثات
│   │   │   │   ├── ConversationItem.vue    ← عنصر محادثة واحد
│   │   │   │   ├── ConversationSearch.vue  ← البحث
│   │   │   │   └── ConversationFilters.vue ← الفلاتر
│   │   │   │
│   │   │   ├── chat/
│   │   │   │   ├── ChatWindow.vue          ← نافذة المحادثة
│   │   │   │   ├── ChatHeader.vue          ← هيدر المحادثة
│   │   │   │   ├── MessageList.vue         ← قائمة الرسائل
│   │   │   │   ├── MessageBubble.vue       ← فقاعة رسالة
│   │   │   │   ├── MediaMessage.vue        ← رسالة وسائط
│   │   │   │   ├── DateDivider.vue         ← فاصل التاريخ
│   │   │   │   ├── ReplyBox.vue            ← صندوق الرد
│   │   │   │   ├── QuickReplyPicker.vue    ← اختيار الردود السريعة
│   │   │   │   └── FileUploader.vue        ← رفع الملفات
│   │   │   │
│   │   │   ├── contact/
│   │   │   │   ├── ContactInfoPanel.vue    ← لوحة معلومات العميل
│   │   │   │   ├── ContactCard.vue         ← بطاقة العميل
│   │   │   │   ├── QuickActions.vue        ← إجراءات سريعة
│   │   │   │   ├── ConversationStats.vue   ← إحصائيات
│   │   │   │   ├── TagsManager.vue         ← إدارة التصنيفات
│   │   │   │   └── InternalNotes.vue       ← ملاحظات داخلية
│   │   │   │
│   │   │   ├── dashboard/
│   │   │   │   ├── DashboardView.vue       ← صفحة Dashboard
│   │   │   │   ├── KPICards.vue            ← بطاقات KPI
│   │   │   │   ├── HourlyChart.vue         ← رسم الرسائل بالساعة
│   │   │   │   ├── EmployeeTable.vue       ← جدول أداء الموظفين
│   │   │   │   └── StatusPieChart.vue      ← توزيع الحالات
│   │   │   │
│   │   │   └── common/
│   │   │       ├── LoadingSpinner.vue      ← مؤشر تحميل
│   │   │       ├── EmptyState.vue          ← حالة فارغة
│   │   │       ├── Avatar.vue              ← صورة المستخدم
│   │   │       ├── Badge.vue               ← شارة العدد
│   │   │       └── TimeAgo.vue             ← الوقت النسبي
│   │   │
│   │   ├── stores/                     ← Pinia State Management
│   │   │   ├── conversations.js        ← حالة المحادثات
│   │   │   ├── messages.js             ← حالة الرسائل
│   │   │   ├── user.js                 ← حالة المستخدم الحالي
│   │   │   └── dashboard.js            ← حالة Dashboard
│   │   │
│   │   ├── composables/                ← Vue Composables
│   │   │   ├── useFrappe.js            ← Frappe API wrapper
│   │   │   ├── useRealtime.js          ← Frappe Realtime wrapper
│   │   │   ├── useNotification.js      ← Browser notifications
│   │   │   ├── useSound.js             ← أصوات التنبيه
│   │   │   └── useInfiniteScroll.js    ← التمرير اللانهائي
│   │   │
│   │   ├── utils/
│   │   │   ├── formatters.js           ← تنسيق التاريخ والأرقام
│   │   │   ├── constants.js            ← الثوابت
│   │   │   └── helpers.js              ← دوال مساعدة
│   │   │
│   │   └── assets/
│   │       ├── styles/
│   │       │   ├── main.css            ← الأنماط الرئيسية
│   │       │   ├── chat.css            ← أنماط الشات
│   │       │   ├── rtl.css             ← دعم RTL
│   │       │   └── variables.css       ← المتغيرات
│   │       └── sounds/
│   │           └── notification.mp3    ← صوت التنبيه
│   │
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── postcss.config.js
│
├── whatsapp_inbox/                     ← Frappe Backend
│   ├── doctype/
│   │   ├── whatsapp_conversation/
│   │   │   ├── whatsapp_conversation.py
│   │   │   ├── whatsapp_conversation.json
│   │   │   └── whatsapp_conversation.js
│   │   ├── whatsapp_team/
│   │   │   ├── whatsapp_team.py
│   │   │   └── whatsapp_team.json
│   │   ├── whatsapp_team_member/
│   │   │   └── whatsapp_team_member.json
│   │   ├── whatsapp_assignment_rule/
│   │   │   ├── whatsapp_assignment_rule.py
│   │   │   └── whatsapp_assignment_rule.json
│   │   └── whatsapp_quick_reply/
│   │       └── whatsapp_quick_reply.json
│   │
│   ├── page/
│   │   └── whatsapp_inbox/
│   │       ├── whatsapp_inbox.py
│   │       ├── whatsapp_inbox.js       ← loads Vue bundle
│   │       └── whatsapp_inbox.html
│   │
│   ├── api/
│   │   ├── __init__.py
│   │   ├── conversations.py            ← CRUD محادثات
│   │   ├── messages.py                 ← إرسال/استقبال
│   │   ├── assignment.py               ← التعيين التلقائي
│   │   ├── contacts.py                 ← ربط جهات الاتصال
│   │   └── dashboard.py                ← بيانات Dashboard
│   │
│   ├── services/
│   │   ├── __init__.py
│   │   ├── assignment_engine.py        ← محرك التوزيع
│   │   ├── escalation_engine.py        ← محرك التصعيد
│   │   └── contact_resolver.py         ← ربط الأرقام بالعملاء
│   │
│   ├── hooks.py
│   ├── tasks.py                        ← Scheduled Jobs
│   └── __init__.py
│
├── public/
│   └── dist/                           ← Vue build output
│       ├── whatsapp_inbox.js
│       ├── whatsapp_inbox.css
│       └── assets/
│
├── setup.py
└── README.md
```

---

## 8. Backend API Endpoints

### 8.1 Conversations API

| Method | Endpoint | الوصف |
|---|---|---|
| GET | get_conversations | جلب قائمة المحادثات مع فلاتر |
| GET | get_conversation | جلب محادثة واحدة بالتفاصيل |
| POST | create_conversation | إنشاء محادثة جديدة |
| PUT | update_conversation | تحديث حالة/تعيين المحادثة |
| POST | close_conversation | إغلاق المحادثة |
| POST | assign_conversation | تعيين لموظف |
| POST | add_note | إضافة ملاحظة داخلية |
| POST | add_tag | إضافة تصنيف |

### 8.2 Messages API

| Method | Endpoint | الوصف |
|---|---|---|
| GET | get_messages | جلب رسائل محادثة (paginated) |
| POST | send_reply | إرسال رد نصي |
| POST | send_media | إرسال ملف/صورة |
| POST | send_template | إرسال قالب WhatsApp |
| POST | mark_as_read | تحديد كمقروءة |

### 8.3 Dashboard API

| Method | Endpoint | الوصف |
|---|---|---|
| GET | get_kpis | بطاقات KPI |
| GET | get_hourly_stats | إحصائيات بالساعة |
| GET | get_employee_stats | أداء الموظفين |
| GET | get_status_distribution | توزيع الحالات |

### 8.4 Quick Reply API

| Method | Endpoint | الوصف |
|---|---|---|
| GET | get_quick_replies | جلب الردود السريعة |
| GET | search_quick_replies | بحث بالاختصار |

---

## 9. نظام التوزيع التلقائي (Assignment Engine)

### 9.1 مسار التوزيع (Flowchart)

```
رسالة واردة جديدة
        │
        ▼
┌──────────────────────┐
│ 1. هل توجد محادثة    │─── نعم ──→ أضف للمحادثة الحالية
│    مفتوحة لهذا الرقم؟│              (أبقِ نفس المُعيَّن)
└──────────┬───────────┘              + زيادة unread_count
           │ لا
           ▼
┌──────────────────────┐
│ 2. أنشئ محادثة جديدة │
│    WhatsApp           │
│    Conversation       │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│ 3. ابحث عن العميل    │─── وُجد ──→ اربط بـ Tenant/Owner/Property
│    في WhatsApp Contact│              عيّن لمسؤول العقار المرتبط
└──────────┬───────────┘
           │ لم يُوجد
           ▼
┌──────────────────────┐
│ 4. فحص قواعد التعيين │
│    بالكلمات المفتاحية │─── مطابق ──→ عيّن حسب القاعدة
│    (حسب الأولوية)     │
└──────────┬───────────┘
           │ لا مطابق
           ▼
┌──────────────────────┐
│ 5. Round Robin على   │──→ عيّن للموظف الأقل تحميلاً
│    الفريق الافتراضي  │    (أقل محادثات Active)
└──────────────────────┘
           │
           ▼
┌──────────────────────┐
│ 6. إشعار فوري        │
│    للموظف المُعيَّن    │
│    (Realtime + Sound) │
└──────────────────────┘
```

### 9.2 قواعد التعيين المقترحة

| الأولوية | الكلمة المفتاحية | التعيين لـ |
|---|---|---|
| 1 | صيانة، تسريب، كهرباء، سباكة | فريق الصيانة |
| 2 | إيجار، عقد، تجديد، دفعة | فريق إدارة الأملاك |
| 3 | فاتورة، سداد، تحويل، دفع | فريق المالية |
| 4 | قضية، محامي، إخلاء، مطالبة | فريق القضايا |
| 5 | استفسار، معلومات، عقار | فريق خدمة العملاء (افتراضي) |

---

## 10. نظام التصعيد (Escalation Engine)

### 10.1 مراحل التصعيد

```
رسالة واردة بدون رد
        │
        │ بعد 2 ساعة (قابل للتعديل)
        ▼
┌──────────────────────┐
│ المرحلة 1:            │
│ ⚠️ إشعار للموظف      │
│ المُعيَّن              │
│                       │
│ → تنبيه صوتي          │
│ → إشعار متصفح         │
│ → تنبيه داخل النظام   │
└──────────┬───────────┘
           │ بعد ساعة إضافية (3 ساعات إجمالي)
           ▼
┌──────────────────────┐
│ المرحلة 2:            │
│ 🔴 إشعار لقائد       │
│ الفريق               │
│                       │
│ → بريد إلكتروني       │
│ → رسالة WhatsApp      │
│ → تغيير الحالة:       │
│   Escalated           │
└──────────┬───────────┘
           │ بعد ساعة إضافية (4 ساعات إجمالي)
           ▼
┌──────────────────────┐
│ المرحلة 3:            │
│ 🚨 إشعار للمدير      │
│ العام                 │
│                       │
│ → بريد إلكتروني       │
│ → رسالة WhatsApp      │
│ → تغيير الأولوية:     │
│   Urgent              │
└──────────────────────┘
```

### 10.2 Scheduled Job

- يعمل كل 15 دقيقة
- يفحص المحادثات بحالة Active أو Pending
- يحسب الوقت منذ آخر رسالة واردة بدون رد صادر
- يطبق مراحل التصعيد بالترتيب
- يسجل التصعيد في custom_escalation_sent لمنع التكرار

---

## 11. Real-time Updates

### 11.1 الأحداث (Events)

| الحدث | البيانات | من يستقبل | التأثير على الواجهة |
|---|---|---|---|
| whatsapp_new_message | conversation, message, phone | المُعيَّن + المشرفين | تحديث القائمة + فقاعة جديدة + صوت |
| whatsapp_message_status | message_id, status | كل المتصلين | تحديث ✓ / ✓✓ |
| whatsapp_assigned | conversation, assigned_to | الموظف الجديد | إضافة للقائمة + إشعار |
| whatsapp_escalation | conversation, stage | قائد الفريق/المدير | تنبيه أحمر |
| whatsapp_conversation_closed | conversation | كل المتصلين | إزالة من القائمة |
| whatsapp_unread_update | counts | كل المتصلين | تحديث Badge في القائمة |

### 11.2 إشعارات المتصفح

```
┌────────────────────────────────┐
│ 📱 رسالة WhatsApp جديدة       │
│                                │
│ أحمد محمد: مرحبا أريد         │
│ الاستفسار عن...                │
│                                │
│ [عرض]              [تجاهل]    │
└────────────────────────────────┘
```

- تظهر فقط إذا النافذة غير نشطة (document.hidden)
- مع صوت تنبيه notification.mp3
- الضغط على "عرض" يفتح المحادثة مباشرة

---

## 12. الأمان والصلاحيات

### 12.1 الأدوار (Roles)

| الدور | الصلاحيات |
|---|---|
| WhatsApp Agent | عرض المحادثات المُعيَّنة له فقط، الرد، إضافة ملاحظات |
| WhatsApp Supervisor | كل صلاحيات Agent + عرض كل محادثات الفريق + إعادة تعيين |
| WhatsApp Admin | كل الصلاحيات + إدارة الفرق + القواعد + الإعدادات + Dashboard |

### 12.2 قواعد الوصول (Permission Rules)

- WhatsApp Conversation: Agent يرى فقط assigned_to = self
- WhatsApp Conversation: Supervisor يرى كل المحادثات في فريقه
- WhatsApp Conversation: Admin يرى كل شيء
- WhatsApp Message: نفس قواعد المحادثة المرتبطة
- WhatsApp Team: Admin فقط
- WhatsApp Assignment Rule: Admin فقط
- WhatsApp Quick Reply: الكل يقرأ، Admin يعدّل

### 12.3 API Security

- كل API endpoint يستخدم @frappe.whitelist()
- التحقق من الصلاحيات في كل request
- لا يمكن الوصول لمحادثات خارج النطاق المسموح
- Rate limiting لمنع الإرسال المكثف

---

# الجزء الخامس: التكامل والنشر

## 13. التكامل مع النظام الحالي

### 13.1 ربط جهات الاتصال التلقائي

عند وصول رسالة من رقم جديد:

```
الرقم: 966501234567
    │
    ├──→ بحث في WhatsApp Contact → وُجد
    │       │
    │       ├──→ مرتبط بـ Tenant → ربط + جلب العقار/الوحدة/المسؤول
    │       ├──→ مرتبط بـ Property Owner → ربط + جلب العقارات
    │       └──→ مرتبط بـ Employee → ربط كموظف
    │
    └──→ لم يُوجد → إنشاء WhatsApp Contact جديد (اسم = الرقم)
```

### 13.2 إجراءات سريعة من المحادثة

| الإجراء | ماذا يحدث |
|---|---|
| فتح طلب صيانة | إنشاء Prevention Maintenance Ticket مع ربط العقار/المستأجر |
| فتح طلب عميل CRM | إنشاء Customer Request مع ربط الرقم |
| عرض عقد الإيجار | فتح Rent Contract المرتبط بالمستأجر |
| عرض سندات القبض | عرض قائمة Tenant Payment Receipt للمستأجر |
| فتح ملف المستأجر | فتح Tenant DocType |
| فتح ملف المالك | فتح Property Owner DocType |

### 13.3 التكامل مع frappe_whatsapp

- الإرسال يستخدم نفس API الموجود في WhatsApp Message.send_message()
- الاستقبال عبر نفس Webhook الحالي
- إضافة hook بعد إنشاء WhatsApp Message لتحديث Conversation

---

## 14. خطة التنفيذ

### المرحلة 1: الأساس (أسبوعان)

الأسبوع 1:
- إنشاء Doctypes: WhatsApp Conversation, Team, Quick Reply, Assignment Rule
- إضافة Custom Fields على WhatsApp Message
- إعداد مشروع Vue.js مع Vite + Tailwind
- بناء Frappe Page wrapper

الأسبوع 2:
- مكونات Vue: ConversationList, ChatWindow, MessageBubble
- ReplyBox مع إرسال الرسائل
- Backend APIs: get_conversations, get_messages, send_reply
- Frappe Bridge (useFrappe composable)

### المرحلة 2: التوزيع والتتبع (أسبوع)

- Assignment Engine: التعيين التلقائي
- Contact Resolver: ربط الأرقام بالعملاء
- تسجيل first_read + replied_by تلقائياً
- حساب response_time
- ContactInfoPanel component

### المرحلة 3: التصعيد والإشعارات (أسبوع)

- Escalation Engine (Scheduled Job)
- frappe.realtime integration
- Browser Notifications + Sound
- Quick Reply Templates
- File upload support

### المرحلة 4: Dashboard والتقارير (أسبوع)

- Dashboard Page (KPIs + Charts)
- Employee Performance Table
- Status Distribution Chart
- Hourly/Daily messages chart
- Script Report للتقارير التفصيلية

### المرحلة 5: التحسين والاختبار (أسبوع)

- تحسين الأداء (Virtual Scrolling, Lazy Loading)
- اختبار مع بيانات حقيقية
- إصلاح الأخطاء
- توثيق المستخدم
- نشر على Production

---

## 15. متطلبات تقنية

### 15.1 البيئة

| المتطلب | الإصدار |
|---|---|
| Frappe Framework | v14+ |
| Node.js | 18+ |
| Vue.js | 3.3+ |
| Vite | 5+ |
| Pinia | 2+ |
| Tailwind CSS | 3+ |
| frappe_whatsapp | (الموجود حالياً) |

### 15.2 المتصفحات المدعومة

| المتصفح | الإصدار |
|---|---|
| Google Chrome | آخر نسختين |
| Microsoft Edge | آخر نسختين |
| Mozilla Firefox | آخر نسختين |
| Safari | 16+ |

### 15.3 المتطلبات الخارجية

- WhatsApp Business API (Cloud API) — موجود ويعمل
- Phone Number ID: 691056547415554
- Access Token صالح
- Webhook endpoint مُعد

---

## 16. مخاطر وتحديات

| المخاطر | الاحتمال | التأثير | الحل |
|---|---|---|---|
| أداء بطيء مع 73K+ رسالة | متوسط | عالي | Virtual Scrolling + Pagination |
| WebSocket disconnection | متوسط | متوسط | Auto-reconnect + Polling fallback |
| WhatsApp 24-hour window | عالي | عالي | تنبيه المستخدم قبل انتهاء النافذة |
| مقاومة الموظفين للتغيير | عالي | متوسط | تدريب + واجهة سهلة |
| فشل Meta API | منخفض | عالي | Retry mechanism + Error logging |

---

## 17. مؤشرات النجاح (Success Metrics)

| المؤشر | الهدف | القياس |
|---|---|---|
| نسبة الرد من النظام | > 80% خلال 3 أشهر | responded_from_system / total_incoming |
| متوسط وقت الاستجابة | < 30 دقيقة | AVG(response_time_mins) |
| نسبة التصعيد | < 10% | escalated / total_incoming |
| رضا الموظفين | > 4/5 | استبيان بعد شهر |
| تقليل استخدام WhatsApp Web | > 70% | مقارنة قبل/بعد |
