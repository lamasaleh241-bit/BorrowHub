# BorrowHub — Relational ERD

## الغرض والنطاق

هذا المخطط يمثل قاعدة البيانات العلائقية المقترحة لـ BorrowHub MVP وفق `docs/PRD.md` و`docs/user-flow.md`. صيغ Mermaid أدناه قابلة للنسخ إلى Mermaid Live. التصميم منطقي ومتوافق مع أنواع PostgreSQL الشائعة، لكن PRD لم يحدد محرك قاعدة بيانات نهائيًا.

## Mermaid ER Diagram

```mermaid
erDiagram
    USERS {
        uuid user_id PK
        varchar sso_subject UK
        varchar display_name
        enum affiliation
        enum status
        timestamptz created_at
        timestamptz updated_at
    }
    ROLES {
        smallint role_id PK
        enum code UK
        varchar description
    }
    USER_ROLES {
        uuid user_id PK, FK
        smallint role_id PK, FK
        timestamptz assigned_at
        uuid assigned_by_user_id FK
    }
    ORGANIZATIONAL_UNITS {
        uuid unit_id PK
        uuid parent_unit_id FK
        enum unit_type
        varchar unit_name
        boolean is_active
    }
    MANAGER_SCOPE_ASSIGNMENTS {
        uuid manager_user_id PK, FK
        uuid unit_id PK, FK
        uuid assigned_by_user_id FK
        timestamptz assigned_at
        boolean is_active
    }
    EQUIPMENT_CATEGORIES {
        uuid category_id PK
        varchar category_name UK
        timestamptz created_at
    }
    EQUIPMENT_TYPES {
        uuid equipment_type_id PK
        uuid category_id FK
        varchar type_name UK
        boolean is_active
    }
    ASSET_UNITS {
        uuid asset_unit_id PK
        uuid equipment_type_id FK
        uuid unit_id FK
        varchar asset_tag UK
        enum operational_status
    }
    OPERATING_CALENDARS {
        uuid calendar_id PK
        uuid unit_id FK, UK
        varchar timezone_name
        boolean is_active
        uuid updated_by_user_id FK
    }
    CALENDAR_WEEKLY_HOURS {
        uuid weekly_hours_id PK
        uuid calendar_id FK
        smallint weekday
        time opens_at
        time closes_at
    }
    CALENDAR_HOLIDAYS {
        uuid holiday_id PK
        uuid calendar_id FK
        date holiday_date
        varchar holiday_name
    }
    LOAN_REQUESTS {
        uuid loan_request_id PK
        uuid borrower_user_id FK
        uuid equipment_type_id FK
        uuid unit_id FK
        smallint requested_quantity
        timestamptz pickup_at
        timestamptz due_at
        enum status
        timestamptz submitted_at
        uuid decision_by_user_id FK
        timestamptz decision_at
    }
    ASSET_ASSIGNMENTS {
        uuid asset_assignment_id PK
        uuid loan_request_id FK
        uuid asset_unit_id FK
        uuid assigned_by_user_id FK
        timestamptz assigned_at
        timestamptz handover_at
        uuid handed_over_by_user_id FK
        timestamptz returned_at
        uuid returned_by_user_id FK
    }
    RETURN_INSPECTIONS {
        uuid return_inspection_id PK
        uuid asset_assignment_id FK, UK
        enum result
        uuid inspected_by_user_id FK
        timestamptz inspected_at
        text notes
    }
    FINES {
        uuid fine_id PK
        uuid loan_request_id FK, UK
        numeric amount
        char currency
        enum status
        timestamptz issued_at
        timestamptz paid_at
        uuid recorded_paid_by_user_id FK
    }
    NOTIFICATIONS {
        uuid notification_id PK
        uuid recipient_user_id FK
        uuid loan_request_id FK
        uuid fine_id FK
        enum notification_type
        timestamptz created_at
        timestamptz read_at
    }
    AUDIT_EVENTS {
        uuid audit_event_id PK
        enum actor_kind
        uuid actor_user_id FK
        varchar action_code
        enum entity_type
        uuid entity_id
        jsonb old_values
        jsonb new_values
        timestamptz occurred_at
    }

    USERS ||--o{ USER_ROLES : has
    ROLES ||--o{ USER_ROLES : assigned_as
    USERS o|--o{ USER_ROLES : assigns

    ORGANIZATIONAL_UNITS o|--o{ ORGANIZATIONAL_UNITS : parent_of
    USERS ||--o{ MANAGER_SCOPE_ASSIGNMENTS : receives_scope
    ORGANIZATIONAL_UNITS ||--o{ MANAGER_SCOPE_ASSIGNMENTS : scoped_to
    USERS ||--o{ MANAGER_SCOPE_ASSIGNMENTS : assigns_scope

    EQUIPMENT_CATEGORIES ||--o{ EQUIPMENT_TYPES : categorizes
    EQUIPMENT_TYPES ||--o{ ASSET_UNITS : has_physical_units
    ORGANIZATIONAL_UNITS ||--o{ ASSET_UNITS : owns_inventory

    ORGANIZATIONAL_UNITS ||--o| OPERATING_CALENDARS : uses_calendar
    USERS ||--o{ OPERATING_CALENDARS : updates
    OPERATING_CALENDARS ||--o{ CALENDAR_WEEKLY_HOURS : defines_hours
    OPERATING_CALENDARS ||--o{ CALENDAR_HOLIDAYS : defines_holidays

    USERS ||--o{ LOAN_REQUESTS : submits
    EQUIPMENT_TYPES ||--o{ LOAN_REQUESTS : requested_type
    ORGANIZATIONAL_UNITS ||--o{ LOAN_REQUESTS : routes_to_scope
    USERS o|--o{ LOAN_REQUESTS : decides

    LOAN_REQUESTS ||--o{ ASSET_ASSIGNMENTS : allocates
    ASSET_UNITS ||--o{ ASSET_ASSIGNMENTS : assigned_over_time
    USERS ||--o{ ASSET_ASSIGNMENTS : assigns_asset
    USERS o|--o{ ASSET_ASSIGNMENTS : records_handover
    USERS o|--o{ ASSET_ASSIGNMENTS : records_return
    ASSET_ASSIGNMENTS ||--o| RETURN_INSPECTIONS : inspected_as
    USERS ||--o{ RETURN_INSPECTIONS : performs_inspection

    LOAN_REQUESTS ||--o| FINES : may_generate
    USERS ||--o{ FINES : records_payment
    USERS ||--o{ NOTIFICATIONS : receives
    LOAN_REQUESTS ||--o{ NOTIFICATIONS : contextualizes
    FINES o|--o{ NOTIFICATIONS : may_contextualize
    USERS o|--o{ AUDIT_EVENTS : performs
```

> `AUDIT_EVENTS.entity_id` هو مرجع متعدد الأنواع (polymorphic) ولا يظهر كعلاقة FK في الرسم؛ تفاصيل المفاضلة موضحة في `docs/database-review.md`.

## قراءة الكيانات والعلاقات

- `USERS` يمثل هوية SSO الجامعية. الصفة طالب/موظف لا تخلط مع صلاحية الدور.
- `ROLES` و`USER_ROLES` يفصلان المستخدم عن صلاحياته ويدعمان إسناد أكثر من دور عند اعتماد ذلك.
- `ORGANIZATIONAL_UNITS` يمثل نطاقات الجامعة الهرمية المحتملة. `MANAGER_SCOPE_ASSIGNMENTS` هو جدول الربط بين المسؤولين والنطاقات.
- `EQUIPMENT_CATEGORIES` و`EQUIPMENT_TYPES` يمثلان تصنيف الكتالوج، بينما `ASSET_UNITS` يسجل كل قطعة فعلية بمعرّف مستقل.
- `OPERATING_CALENDARS` له نطاق واحد، وله عدة فترات أسبوعية وعطلات. يمكن تمثيل أكثر من فترة عمل في اليوم.
- `LOAN_REQUESTS` هو سجل الطلب الذي يستمر ليصبح سجل الاستعارة عند الموافقة؛ لذلك لا توجد نسختان منفصلتان متكررتان للطلب والقرض.
- `ASSET_ASSIGNMENTS` يربط طلب الاستعارة بالقطع الفعلية. العلاقة بين الطلب والقطع هي N:M عبر الزمن، ممثلة بعلاقة 1:N من كل من الطلب والقطعة إلى سجل التخصيص.
- `RETURN_INSPECTIONS` يسجل نتيجة فحص واحدة لكل قطعة مخصصة. الحالة التشغيلية الحالية تحفظ على `ASSET_UNITS`، بينما نتيجة الفحص تاريخية.
- `FINES` اختياري 1:1 مع طلب/استعارة؛ الفهرس الفريد على `loan_request_id` يمنع أكثر من غرامة واحدة لكل استعارة.
- `NOTIFICATIONS` تتبع مستلم الإشعار وطلبه، ويمكن ربط إشعار الغرامة بسجل الغرامة.
- `AUDIT_EVENTS` يحفظ سجل الإجراءات ومنفذها عند توفر مستخدم، أو يحدد أن الفاعل النظام.

## علاقة النموذج بتدفق المستخدم

يبدأ التدفق بتحميل `USERS` عبر معرف SSO ثم التحقق من `USER_ROLES`. يقرأ الكتالوج من `EQUIPMENT_TYPES` و`EQUIPMENT_CATEGORIES` ويحسب الكمية المتاحة من `ASSET_UNITS` والتخصيصات الفعالة. يحفظ الطلب في `LOAN_REQUESTS` ويرسل إشعارات في `NOTIFICATIONS`. يراجع مسؤول المعدات ضمن نطاقه، ثم تسجل القطع المخصصة في `ASSET_ASSIGNMENTS`. التسليم والإرجاع يوثقان على مستوى كل قطعة، ويضاف فحص في `RETURN_INSPECTIONS`. عند التأخير، ينشأ `FINES` مرة واحدة وتُسجل التنبيهات، ويُحفظ الدفع اليدوي وسائر التغييرات المهمة في `AUDIT_EVENTS`.

## ملاحظات على cardinality

| العلاقة | الكاردينالية | سببها |
|---|---|---|
| User ↔ Role | M:N عبر `USER_ROLES` | المستخدم قد يحمل أكثر من دور، والدور يشمل مستخدمين كثيرين |
| Manager ↔ Organizational unit | M:N عبر `MANAGER_SCOPE_ASSIGNMENTS` | المسؤول قد يدير نطاقات متعددة، والنطاق قد يخدمه أكثر من مسؤول |
| Category → Equipment type | 1:N | الفئة تضم أنواعًا متعددة، والنوع يتبع فئة واحدة |
| Equipment type → Asset unit | 1:N | النوع له قطع مادية كثيرة، والقطعة تتبع نوعًا واحدًا |
| User → Loan request | 1:N | المستخدم يستطيع تقديم طلبات عديدة عبر الزمن |
| Loan request ↔ Asset unit | M:N عبر `ASSET_ASSIGNMENTS` | الطلب قد يحتاج عدة قطع، والقطعة يعاد تخصيصها لطلبات متعاقبة؛ يمنع تداخل الحيازة |
| Assignment → Return inspection | 1:0..1 | قد لا يكون الإرجاع قد حصل بعد؛ بعد الإرجاع يسجل فحص واحد للقطعة |
| Loan request → Fine | 1:0..1 | ليست كل استعارة متأخرة؛ غرامة واحدة كحد أقصى لكل استعارة |
| Operating calendar → weekly hours/holidays | 1:N | لكل تقويم عدة أيام/فترات عمل وعطلات |
