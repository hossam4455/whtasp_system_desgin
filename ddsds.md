# 📋 تقرير شامل — إجراءات عقود الإيجار (Rent Contract Actions)

> توثيق كامل لكل إجراء في قائمة "اجراءات العقد" — ماذا اتعمل، ليه، وإيه الفايلات المُعدّلة.

---

## 🗺️ خريطة الإجراءات

| # | الإجراء | الـ DocType | الحالات المتاحة |
|---|---------|-------------|-----------------|
| 1 | 🏛️ رفع قضية | `Case` | كل الحالات |
| 2 | 💸 مطالبة مالية | `Financial Claim` | `نشط`, `منتهي` (بدون المؤرشف) |
| 3 | 💰 رفع قيمة الإيجار | `Rent Raise Request` | `نشط` |
| 4 | 🚫 عدم الرغبة بالتجديد | `Non Renewal Request` | `نشط`, `مسجل` |
| 5 | 🔄 تجديد تلقائي | `Auto Renewal Request` | `نشط` |
| 6 | ⚖️ تسوية العقد | `Rent Contract Cancellation Request` | `نشط` |
| 7 | 📝 إشعار فسخ عقد | `Contract Termination Notice` | `نشط`, `منتهي`, `مسجل` |
| 8 | 💵 إنشاء سند قبض | `Tenant Payment Receipt` | `نشط` |
| 9 | 🔑 استلام الوحدة | `Unit Receive Request` | نهاية العقد |
| 10 | 🚪 تسليم الوحدة | `Unit Deliver Request` | قبل التفعيل (تلقائي) |

---

## 1️⃣ رفع قضية (File Case) 🏛️

### الغرض
إنشاء قضية قانونية على مستأجر مخالف لشروط العقد. الإجراء ينتقل من مسؤول العقار → مشرف العقار → محامي للمتابعة حتى الإنتهاء.

### ماذا تم تنفيذه
- **Dialog** مخصص `create_case_dialog(frm)` داخل `rent_contract.js` يفتح نافذة لاختيار نوع القضية من قائمة ديناميكية.
- **تسجيل الخطوات** — الـ Case DocType مربوط بـ workflow (مسوّدة → بانتظار الاعتماد → معتمد → مكتمل).
- **workflow_state** منفصل للـ lawyer assignment واعتماد المشرف.

### الفايلات المُعدّلة

| الملف | التغيير |
|------|---------|
| `nofouth_property_managment/doctype/rent_contract/rent_contract.js` | إضافة زر + `create_case_dialog` |
| `nofouth_property_managment/doctype/case/case.js` | validation + lawyer assignment |
| `nofouth_property_managment/doctype/case/case.json` | حقول: case_type, description, lawyer, status |
| `nofouth_property_managment/doctype/case/case.py` | after_insert → ربط مع العقد |
| `fixtures/workflow.json` | workflow state machine لـ Case |

### الـ Commits المرتبطة

| Commit | التاريخ | الوصف |
|--------|---------|-------|
| `52ed670` | 2026-04-01 | new updates cases, property owner — إضافة types + ربط بالمالك |
| `34d67b3` | 2026-02-24 | updates in cases and fix flexible contracts payment commission |
| `d4a4f4d` | 2026-03-16 | new updates from nofouth dev 15-03 |
| `221b2cd` | 2026-04-05 | update sales invoice — ربط فاتورة مع القضية |

---

## 2️⃣ مطالبة مالية (Financial Claim) 💸

### الغرض
إشعار المستأجر بوجود مبلغ مستحق غير مسدد. الإجراء متاح من:
- داخل عقد الإيجار
- جداول التحصيل والمتأخرين

### ماذا تم تنفيذه

#### الـ UI (Dialog Box)
- اختيار **رقم الدفعة** (قائمة منسدلة تعرض الدفعات بحالات `غير مستحقة`, `استحقت`, `تم الدفع - جزئي`).
- **تاريخ الخطاب** (تلقائي = اليوم، قابل للتعديل).
- **أنواع الإشعارات** — 4 checkboxes:
  - رسالة واتساب
  - SMS
  - بريد إلكتروني
  - إرفاق الخطاب PDF
- **عرض تفاصيل الدفعة** (read-only): الفترة من/إلى، قيمة الإيجار، المصروفات، الضريبة، المبلغ المدفوع، المتبقي.

#### شاشة Financial Claim (بعد الإنشاء)
- **بيانات العقد والمستأجر** — رقم العقد، نوع، بداية/نهاية، اسم المستأجر، هوية، هاتف.
- **بيانات الوحدة والعقار** — العقار، الوحدات، المالك، مسؤول العقار.
- **جدول الدفعات** — قابل للتعديل.
- **خيارات الإشعار** — قابلة للتعديل.
- **قالب الإشعار** — Jinja يُنتج نص المطالبة.
- **مخرجات** — PDF / Word (checkbox).
- **حفظ PDF** تلقائي في مجلد العقد.

### قيود هامة مُنفَّذة
1. **`Financial Claim` ليس لها workflow** — الإنشاء النهائي يتم بعد الاعتماد من شاشة المراجعة.
2. **منع إنشاء مطالبة لعقار مؤرشف** — فحص `workflow_state === 'مؤرشف'` قبل عرض الزر.
3. **قناة الإرسال الافتراضية** — WhatsApp + SMS.

### الفايلات المُعدّلة

| الملف | التغيير |
|------|---------|
| `doctype/rent_contract/rent_contract.js:578+` | الزر + Dialog كامل + payment filter |
| `doctype/financial_claim/financial_claim.js` | UI logic (83 سطر مضاف) |
| `doctype/financial_claim/financial_claim.json` | 343 إضافة — fields + sections |
| `doctype/financial_claim/financial_claim.py` | 291 إضافة — create + attach + send |
| `doctype/financial_claim_payments/` | child table للدفعات |
| `print_format/financial_claim_letter/` | Print Format Jinja |
| `controllers/property_transaction.py` | تسجيل GL لو ربط مع فاتورة |
| `apis/notifications_api.py` | إرسال قنوات الإشعار |
| `apis/letters_api.py` | توليد نص الخطاب |

### الـ Commits المرتبطة

| Commit | التاريخ | الوصف |
|--------|---------|-------|
| `80ba98d` | 2026-03-26 | **contract termination notice + financial claim updates** — الأساس |
| `b184aea` | 2026-03-31 | تحسينات الـ payment selection + ربط بالـ overdue reports |
| `66def7f` | — | schedule in financial claim |
| `fa8a3e5` | — | Apply finance changes |

---

## 3️⃣ رفع قيمة الإيجار (Rent Raise) 💰

### الغرض
إشعار المستأجر بزيادة الإيجار قبل نهاية العقد بـ 30 يوم — لو رغب في التجديد، يُطبَّق السعر الجديد.

### ماذا تم تنفيذه

#### المدخلات
- **قيمة الإيجار الجديدة** (validation: أكبر من الحالية).
- **جدول رفع المصروفات** — child table:
  - نوع المصروف (Link)
  - قيمة جديدة (validation: أكبر من الحالية)
  - لا تكرار لنفس النوع
- **إرسال خطاب قبل 30 يوم** — Yes/No.

#### Validations الحرجة
```python
# Server-side في rent_raise_request.py
if new_rent_amount < contract.annual_rent:
    throw("قيمة الإيجار الجديدة أقل من قيمة إيجار الحالية في العقد")

for expense in new_expenses:
    if expense.amount < contract_expense.amount:
        throw(f"قيمة المصروف الجديدة من النوع {expense.type} أقل من الحالية")
```

#### 3-State Workflow
```
مسودة (مسؤول العقار)
      ↓
بانتظار اعتماد مشرف العقار
      ↓
معتمد
```

### تأثير الاعتماد
- **عند تجديد العقد** → Server-side validation يمنع تجديد بقيمة أقل من الـ rent raise المعتمد.

### الفايلات المُعدّلة

| الملف | التغيير |
|------|---------|
| `doctype/rent_contract/rent_contract.js:511` | إضافة الزر |
| `doctype/rent_contract/rent_contract.py` | `create_rent_raise_request` method |
| `doctype/rent_raise_request/rent_raise_request.json` | كل الحقول |
| `doctype/rent_raise_request/rent_raise_request.py` | validation + save + send |
| `doctype/rent_raise_request/rent_raise_request.js` | UI + notification options |
| `doctype/raise_rent_request_expense/` | child table للمصروفات |
| `doctype/rent_raise_units/` | child table للوحدات |
| `doctype/letter_of_rent_raise/` | DocType مصاحب للخطاب |
| `print_format/letter_of_rent_raise_pf/` | Print Format |
| `fixtures/workflow.json` | 3-state workflow |

### الـ Commits

| Commit | التاريخ | الوصف |
|--------|---------|-------|
| `463749f` | 2026-03-26 | updates of exit admin + workflow |
| `5202984` | — | Update Phase 1 — تحسينات UX |
| `4000668` | — | updates with fix |

---

## 4️⃣ عدم الرغبة بالتجديد (Non-Renewal) 🚫

### الغرض
إشعار المستأجر بعدم تجديد العقد عند انتهائه، مع طلب تسليم الوحدة.

### ماذا تم تنفيذه
- **تاريخ الخطاب** (تلقائي + قابل للتعديل).
- **ملاحظات** نصية.
- **جدول الوحدات** — جلب تلقائي من العقد.
- **خيارات الإشعار** — WhatsApp, SMS, Email, PDF.
- **3-State Workflow** (نفس الـ rent raise).

### Validations بعد الاعتماد
```python
# في rent_raise_request.py و auto_renewal_request.py
if has_approved_non_renewal(contract):
    throw("لا يمكن عمل رفع إيجار/تجديد تلقائي بعد اعتماد عدم الرغبة بالتجديد")
```

### الفايلات المُعدّلة

| الملف | التغيير |
|------|---------|
| `doctype/rent_contract/rent_contract.js:557` | الزر |
| `doctype/rent_contract/rent_contract.py` | `create_non_renewal_request` method |
| `doctype/non_renewal_request/non_renewal_request.json` | الحقول |
| `doctype/non_renewal_request/non_renewal_request.py` | logic + email/whatsapp |
| `doctype/non_renewal_request/non_renewal_request.js` | UI |
| `print_format/non_renwal/` | Print Format |

### الـ Commits

| Commit | التاريخ | الوصف |
|--------|---------|-------|
| `463749f` | 2026-03-26 | updates of exit admin |
| `5202984` | — | Update Phase 1 |
| `4000668` | — | updates with fix |
| `1aa58c8` | — | unit handling |
| `946d6b7` | — | Updating |

---

## 5️⃣ تجديد تلقائي (Auto Renewal) 🔄

### الغرض
إنشاء طلب تجديد تلقائي للعقد بنفس الشروط الحالية.

### ماذا تم تنفيذه
- **Mapped Doc** — الـ frappe.model.open_mapped_doc ينسخ بيانات العقد.
- **Validations**:
  - حالة العقد = `نشط` فقط.
  - لا يوجد `non_renewal_request` معتمد.
  - لا يوجد `rent_raise_request` معتمد (لأن القيم الجديدة لازم تُطبق).

### الفايلات المُعدّلة

| الملف | التغيير |
|------|---------|
| `doctype/rent_contract/rent_contract.js:519` | الزر |
| `doctype/rent_contract/rent_contract.py` | `create_auto_renewal_request` method |
| `doctype/auto_renewal_request/auto_renewal_request.json` | الحقول |
| `doctype/auto_renewal_request/auto_renewal_request.py` | logic |
| `doctype/auto_renewal_request/auto_renewal_request.js` | UI |
| `print_format/auto_renewal_request_pf/` | Print Format |

### الـ Commits

| Commit | التاريخ | الوصف |
|--------|---------|-------|
| `079d806` | 2026-03-27 | print format |
| `463749f` | 2026-03-26 | updates of exit admin |
| `5202984` | — | Update Phase 1 |

---

## 6️⃣ تسوية العقد (Contract Settlement) ⚖️

### الغرض
إنشاء طلب تسوية عقد نشط — حساب الدفعات المستحقة وتعديل حالتها.

### ماذا تم تنفيذه

#### الـ DocType Structure
- **`Rent Contract Cancellation Request`** — الـ master
- **`Rent Contract Cancel Request Unit`** — child للوحدات
- **`Rent Contract Canellation Payment`** — child للدفعات

#### الـ Flow
1. جلب جميع الدفعات من العقد.
2. حساب الـ refund/receivable لكل دفعة.
3. الموظف يعتمد → يتم إنشاء Journal Entry للتسوية.
4. حالة العقد → `ملغى`.

### Validations
- لا يمكن تسوية عقد فيه مطالبة مالية مفتوحة.
- لا يمكن تسوية عقد بعد حدث فسخ.

### الفايلات المُعدّلة

| الملف | التغيير |
|------|---------|
| `doctype/rent_contract/rent_contract.js:528` | الزر |
| `doctype/rent_contract/rent_contract.py` | `create_cancel_request` method |
| `doctype/rent_contract_cancellation_request/` | master doctype |
| `doctype/rent_contract_cancel_request_unit/` | child للوحدات |
| `doctype/rent_contract_canellation_payment/` | child للدفعات |
| `doctype/rent_contract_reconcile_payment/` | reconciliation logic |
| `print_format/cancelation_request/` | Print Format |

### الـ Commits

| Commit | التاريخ | الوصف |
|--------|---------|-------|
| `4cf641d` | 2026-04-05 | reconsilation — الـ cancel reconcile logic |
| `2c44601` | — | updates — الـ cancellation |

---

## 7️⃣ إشعار فسخ عقد (Contract Termination Notice) 📝

### الغرض
إشعار رسمي للمستأجر بفسخ العقد بسبب مخالفة.

### ماذا تم تنفيذه

#### الـ Inputs
- **رقم العقد** (تلقائي Read-only).
- **تاريخ الخطاب** (تلقائي قابل للتعديل).
- **نوع المخالفة** (نص).
- **التوضيح** (نص).

#### الـ Contract Data (Read-only)
رقم العقد، المستأجر، المالك، العقار، الوحدات، الإيجار السنوي، الخدمات السنوية، التواريخ، مسؤول العقار.

#### Channels
WhatsApp, SMS, Email + إرفاق PDF.

#### Workflow 3-State
```
مسودة (مسؤول العقار)
      ↓
بانتظار اعتماد مشرف العقار
      ↓
معتمد
```

### تأثير الاعتماد
```python
# في auto_renewal_request.py و rent_contract.py
if has_approved_termination_notice(contract):
    throw("لا يمكن تجديد العقد في ظل وجود إشعار فسخ معتمد")
```

### الفايلات المُعدّلة — التفصيل الكامل

| الملف | التغيير |
|------|---------|
| `doctype/rent_contract/rent_contract.js:502` | الزر |
| `doctype/rent_contract/rent_contract.py` | `create_contract_termination_notice` |
| `doctype/contract_termination_notice/__init__.py` | إنشاء الـ module |
| `doctype/contract_termination_notice/contract_termination_notice.json` | 356 سطر — الحقول كاملة |
| `doctype/contract_termination_notice/contract_termination_notice.py` | 306 سطر — Python logic |
| `doctype/contract_termination_notice/contract_termination_notice.js` | 54 سطر UI |
| `doctype/contract_termination_notice/test_contract_termination_notice.py` | tests |
| `controllers/property_transaction.py` | 65 سطر — ربط GL |
| `doc_events/whatsapp_message.py` | 27 سطر — ربط WhatsApp message events |
| `doc_events/sms_log.py` | 13 سطر — SMS tracking |
| `hooks.py` | 12 سطر — تسجيل doc_events |
| `print_format/contract_termination_notice_pf/` | Print Format |

### الـ Commit الرئيسي
**`80ba98d`** (2026-03-26): `contract termination notice + financial claim updates`
- 14 ملف معدّل
- +1,500 سطر إضافي
- هذا هو الأساس الكامل للفيتشر.

---

## 8️⃣ إنشاء سند قبض (Create Receipt Voucher) 💵

### الغرض
إنشاء `Tenant Payment Receipt` مربوط بالعقد تلقائياً.

### ماذا تم تنفيذه

#### Flow (بدون Dialog)
1. الضغط على الزر → `check_duplicate_tpr` API.
2. إذا وُجد TPR مسوّدة/مفتوح بنفس الدفعة → يمنع الإنشاء.
3. يوجه لـ `Tenant Payment Receipt` form بـ pre-filled values:
   - `rent_contract` = العقد الحالي
   - `tenant` = المستأجر

### Validation الحرجة
```python
# rent_contract.py:check_duplicate_tpr
def check_duplicate_tpr(rent_contract):
    open_tpr = frappe.db.exists('Tenant Payment Receipt', {
        'rent_contract': rent_contract,
        'docstatus': ['!=', 2],  # مش ملغي
        'workflow_state': ['in', ['مسوّدة', 'بانتظار الاعتماد']]
    })
    if open_tpr:
        frappe.throw(f"يوجد سند قبض مفتوح ({open_tpr})")
```

### الفايلات المُعدّلة

| الملف | التغيير |
|------|---------|
| `doctype/rent_contract/rent_contract.js:535` | الزر + duplicate check |
| `doctype/rent_contract/rent_contract.py` | `check_duplicate_tpr` method |
| `doctype/tenant_payment_receipt/` | الـ TPR doctype (تحسينات عديدة) |
| `doctype/tenant_payment_receipt_unit/` | child table |
| `controllers/property_transaction.py` | GL posting |
| `print_format/tenant_payment_receipt/` | Print Format |

### الـ Commits

| Commit | التاريخ | الوصف |
|--------|---------|-------|
| `57ccaa1` | 2026-04-18 | debugging TPR Submitting Process |
| `edd1ffa` | 2026-04-18 | edits |
| `00b155d` | 2026-04-06 | commission on services from property not contract in TPR |
| `bd0f5e4` | 2026-04-06 | fix |
| `dc409e3` | 2026-04-02 | opd handling logic |
| `87b4ad0` | 2026-02-25 | tenant payment receipt |

---

## 9️⃣ استلام الوحدة (Unit Receive) 🔑

### الغرض
استلام الوحدة من المستأجر بعد نهاية العقد.

### الـ Inputs
- تاريخ الاستلام
- ملاحظات الاستلام
- تقييمات الاستلام (multi-rating)
- مرفقات صور الاستلام (child table بالصور)

### تأثير الاعتماد
- حالة العقد → `مؤرشف`.
- **لا يؤثر على حالة الدفعات** (المستحقات تظل كما هي).

### الفايلات المُعدّلة

| الملف | التغيير |
|------|---------|
| `doctype/rent_contract/rent_contract.js:487` | الزر |
| `doctype/unit_receive_request/unit_receive_request.json` | الحقول |
| `doctype/unit_receive_request/unit_receive_request.py` | archive on approval |
| `print_format/` | Print Format |

---

## 🔟 تسليم الوحدة (Unit Deliver) 🚪

### الغرض
تسليم الوحدة للمستأجر — **ينشأ تلقائياً** عند إنشاء العقد ويعتمد تلقائياً مع اعتماد العقد.

### الـ Inputs
- تاريخ التسليم
- ملاحظات التسليم
- تقييمات التسليم
- مرفقات صور التسليم

### التكامل
- `after_insert` على Rent Contract → ينشئ Unit Deliver Request تلقائياً.
- Status change الأول للعقد لـ "نشط" → يعتمد الـ deliver request.

- **تاريخ الخطاب** (تلقائي قابل للتعديل).
- **نوع المخالفة** (نص).
- **التوضيح** (نص).

#### الـ Contract Data (Read-only)
رقم العقد، المستأجر، المالك، العقار، الوحدات، الإيجار السنوي، الخدمات السنوية، التواريخ، مسؤول العقار.

#### Channels
WhatsApp, SMS, Email + إرفاق PDF.
### الفايلات المُعدّلة

| الملف | التغيير |
|------|---------|
| `doctype/rent_contract/rent_contract.py` | auto-create in `after_insert` |
| `doctype/rent_contract/rent_contract.js:852` | الزر (للحالات الخاصة) |
| `doctype/unit_deliver_request/unit_deliver_request.json` | 14 سطر تحديث |
| `doctype/unit_deliver_request/unit_deliver_request.js` | 20 سطر logic |
| `doctype/unit_deliver_request/unit_deliver_request.py` | validation |
| `api.py` | `create_unit_deliver_request_api` |

### الـ Commits

| Commit | التاريخ | الوصف |
|--------|---------|-------|
| `463749f` | 2026-03-26 | updates of exit admin |
| `ce1b2b1` | 2026-03-11 | fix |
| `2ca9dbe` | — | excluding validation of partners |
| `9e08b5d` | — | Fix |
| `cc30d66` | — | Enhancing Code |

---

## 🔗 العلاقات المتشابكة بين الإجراءات

```
                    ┌─────────────────────────┐
                    │    Rent Contract        │
                    │    (نشط)                │
                    └────────────┬────────────┘
                                 │
            ┌──────────┬──────────┼──────────┬──────────┬──────────┐
            │          │          │          │          │          │
        رفع قضية   مطالبة    رفع إيجار  عدم التجديد  تجديد   إشعار فسخ
                    مالية                            تلقائي
                                 │
                                 ↓ (عند الاعتماد)
            ┌─────────────────────────────────────────┐
            │ ⛔ يمنع: تجديد تلقائي + رفع إيجار       │
            └─────────────────────────────────────────┘
                                 │
                                 ↓ (عند التسليم/الاستلام)
                       ┌─────────────────┐
                       │   مؤرشف         │
                       └─────────────────┘
```

---

## 📅 Timeline شامل للتطوير

### Q1 2026 (يناير - مارس)

| التاريخ | الإجراء | التفاصيل |
|---------|--------|----------|
| 2026-01-20 | Reports, procedures, taxes, fixes | `be3bd57` — بداية revamp |
| 2026-01-26 | update validation of commission | `65e1f7b` |
| 2026-02-03 | Updates in workflow | `960dee8` — تحديد workflow states |
| 2026-02-07 | Taxes NEW VERSION | `88019f5` + `bf1d8d5` — New doctypes related to property |
| 2026-02-11 | fix invoice cleared validation | `d77d1c3` |
| 2026-02-24 | updates in cases | `34d67b3` — إصلاح عمولة العقود المرنة |
| 2026-02-25 | tenant payment receipt | `87b4ad0` + `4ba4cfd` |
| 2026-03-16 | major updates from dev branch | `d4a4f4d` + `110dabb` + `415c8ca` |
| **2026-03-26** | **🌟 Contract Termination + Financial Claim** | **`80ba98d` — الأساس الكامل** |
| 2026-03-26 | updates of exit admin | `463749f` — workflow enhancements |
| 2026-03-27 | print format | `079d806` — Auto Renewal |
| 2026-03-31 | new updates in reports and financial claim | `b184aea` — تحسينات |

### Q2 2026 (أبريل - إلى الآن)

| التاريخ | الإجراء | التفاصيل |
|---------|--------|----------|
| 2026-04-01 | new updates cases, property owner | `52ed670` — case types |
| 2026-04-05 | reconsilation | `4cf641d` — تسوية العقد |
| 2026-04-06 | commission on services from property | `00b155d` |
| 2026-04-18 | debugging TPR | `57ccaa1` |
| 2026-04-20 | fix contract fees GL reverse | `007e777` |

---

## 🐛 أبرز الـ Bugs اللي اتحلت

### 1. **Financial Claim على عقار مؤرشف**
- **المشكلة**: كان يمكن إنشاء مطالبة على عقار مؤرشف.
- **الحل**: `rent_contract.js:568-577` — فحص `workflow_state === 'مؤرشف'` وإزالة الزر.
- **Commit**: `b184aea`.

### 2. **TPR Duplicate**
- **المشكلة**: إنشاء أكتر من سند قبض مفتوح لنفس العقد.
- **الحل**: `check_duplicate_tpr` API.
- **Commit**: `edd1ffa`, `57ccaa1`.

### 3. **Contract Fees GL Reversed**
- **المشكلة**: قيود GL للرسوم مقلوبة (credit/debit).
- **الحل**: تعديل accounting logic في `property_transaction.py`.
- **Commit**: `007e777`, `a8bf134`, `bbaed1c`.

### 4. **Flexible Contracts Commission**
- **المشكلة**: العقود المرنة بتحسب عمولة غلط.
- **الحل**: revamp لـ commission calculation.
- **Commit**: `34d67b3`.

### 5. **Invoice Cleared Validation**
- **المشكلة**: validation بتفشل لو الـ invoice كان cleared.
- **Commit**: `d77d1c3`.

### 6. **Tax Percent Auto-Calculation**
- **المشكلة**: نسبة الضريبة ما كانتش تتحسب تلقائياً.
- **Commit**: `f3078d4`, `88019f5`.

### 7. **Validation of Partners**
- **المشكلة**: شركاء العقار كانوا يسببوا blocking validations.
- **الحل**: استثنائهم من الـ unit deliver validation.
- **Commit**: `2ca9dbe`.

### 8. **Cancellation Reconciliation**
- **المشكلة**: تسوية العقد كانت بتخلي GL entries معلقة.
- **الحل**: `rent_contract_reconcile_payment` doctype.
- **Commit**: `4cf641d`.

---

## 🗂️ الفايلات الأساسية — Map كامل

### الـ Frontend
```
nofouth_property_managment/doctype/rent_contract/
├── rent_contract.js          ← كل الـ buttons (~900+ سطر)
└── rent_contract_list.js     ← list view
```

### الـ Backend
```
nofouth_property_managment/doctype/rent_contract/
└── rent_contract.py          ← methods:
                                - create_contract_termination_notice
                                - create_rent_raise_request
                                - create_auto_renewal_request
                                - create_cancel_request
                                - create_non_renewal_request
                                - create_unit_deliver_request
                                - check_duplicate_tpr
                                - create_case_dialog logic
```

### الـ DocTypes الفرعية (10 جديدة)
```
nofouth_property_managment/doctype/
├── case/                           ← رفع قضية
├── financial_claim/                ← مطالبة مالية (+ financial_claim_payments)
├── rent_raise_request/             ← رفع إيجار (+ raise_rent_request_expense, rent_raise_units)
├── non_renewal_request/            ← عدم التجديد
├── auto_renewal_request/           ← تجديد تلقائي
├── rent_contract_cancellation_request/  ← تسوية (+ cancel_request_unit, canellation_payment)
├── contract_termination_notice/    ← فسخ
├── tenant_payment_receipt/         ← سند قبض (+ tenant_payment_receipt_unit)
├── unit_receive_request/           ← استلام
└── unit_deliver_request/           ← تسليم
```

### الـ Print Formats
```
nofouth_property_managment/print_format/
├── contract_termination_notice_pf/
├── non_renwal/
├── financial_claim_letter/
├── cancelation_request/
├── tenant_payment_receipt/
├── letter_of_rent_raise_pf/
└── auto_renewal_request_pf/
```

### الـ APIs + Helpers
```
nofouth_property_management/
├── api.py                         ← create_unit_deliver_request_api
├── apis/
│   ├── notifications_api.py       ← channel routing (WhatsApp/SMS/Email)
│   ├── letters_api.py             ← توليد نصوص الإشعارات
│   └── reports_api.py             ← overdue tenants, undue reports
├── controllers/
│   └── property_transaction.py    ← GL posting للإجراءات
└── doc_events/
    ├── sales_invoice.py
    ├── sms_log.py                 ← tracking
    └── whatsapp_message.py        ← tracking
```

### الـ Workflows + Fixtures
```
nofouth_property_management/fixtures/
├── workflow.json                  ← 3-state workflows
├── custom_field.json              ← الحقول المضافة
└── property_setter.json           ← UI customizations
```

---

## 📊 إحصائيات شاملة

| البيان | القيمة |
|--------|--------|
| **عدد الإجراءات المُنفَّذة** | 10 |
| **الـ DocTypes الجديدة** | 10 رئيسي + ~7 child tables |
| **الـ Whitelisted APIs** | 15+ |
| **الـ Print Formats** | 7 |
| **عدد الـ Commits (12 شهر)** | 80+ |
| **الـ Files المعدّلة (تقديرياً)** | 50+ ملف |

---

## 🎯 الخلاصة

تم بناء منظومة كاملة لإدارة "اجراءات العقد" بتشمل:
1. ✅ **10 إجراءات مستقلة** لكل واحد DocType خاص بيه
2. ✅ **Workflows 3-state** (مسودة → بانتظار → معتمد) للإجراءات المهمة
3. ✅ **قنوات إشعار متعددة** (WhatsApp/SMS/Email/PDF) لكل إجراء
4. ✅ **Cross-validations** بين الإجراءات (مثلاً: منع التجديد بعد الفسخ)
5. ✅ **Print Formats** احترافية بـ Jinja templates
6. ✅ **GL integration** لكل إجراء مالي
7. ✅ **Tracking** كامل للـ notifications (delivered/read/failed)

---

## 📌 أوامر للمراجعة السريعة

```bash
# لرؤية كل التغييرات على إجراء معين (مثلاً Financial Claim):
cd /home/frappe/frappe-bench/apps/nofouth_property_management
git log --oneline --all -- "nofouth_property_management/nofouth_property_managment/doctype/financial_claim/"

# لرؤية commit معين بالكامل:
git show 80ba98d

# للبحث عن وظيفة معينة (متى أُضيفت):
git log --all -S "create_contract_termination_notice" --oneline
```
