
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
| `doc_events/whatsapp_message.py` | 27 سطر — ربط WhatsApp message events |
| `doc_events/sms_log.py` | 13 سطر — SMS tracking |
| `hooks.py` | 12 سطر — تسجيل doc_events |
| `print_format/contract_termination_notice_pf/` | Print Format |

