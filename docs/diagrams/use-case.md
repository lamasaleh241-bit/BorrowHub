# BorrowHub MVP — Use Case Diagram

## Purpose

يحدد مخطط حالات الاستخدام من يتفاعل مع BorrowHub وما الوظائف الداخلة في MVP. يشمل مسار المستعير ومسؤول المعدات ودور مدير النظام الموصى به، إضافة إلى مزود SSO بوصفه نظامًا خارجيًا مساعدًا.

## Source requirements

- `docs/PRD.md`: Users/Roles, MVP Scope, FR-001–FR-030, NFR-001–NFR-002.
- `docs/user-flow.md`: المسار الرئيسي من تسجيل الدخول إلى الإرجاع والغرامة.
- `docs/ERD.md` و`docs/database.dbml`: الكيانات، النطاقات، الإشعارات، والتقويم.

## Valid code

```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle

actor "Borrower\n(Student or Staff)" as Borrower
actor "Equipment Manager" as Manager
actor "System Administrator\n(recommended role)" as Admin
actor "University SSO" as SSO <<external system>>

rectangle "BorrowHub MVP" {
  usecase "Sign in with\nuniversity account" as UC_SignIn
  usecase "Browse equipment\nand view availability" as UC_Browse
  usecase "Submit borrowing request" as UC_Request
  usecase "Validate eligibility, dates,\nwork calendar and availability" as UC_Validate
  usecase "View own request history\nand notifications" as UC_History
  usecase "Cancel request\nbefore approval" as UC_Cancel
  usecase "Review request" as UC_Review
  usecase "Approve or reject request" as UC_Decide
  usecase "Recheck availability\nand assign asset units" as UC_Assign
  usecase "Record physical handover" as UC_Handover
  usecase "Record return and\ninspection result" as UC_Return
  usecase "Record fine status\nmanually" as UC_Fine
  usecase "Manage inventory\nand asset statuses" as UC_Inventory
  usecase "Manage manager roles\nand scopes" as UC_Access
  usecase "Configure working hours,\ndays and holidays" as UC_Calendar
  usecase "Review audit events" as UC_Audit
}

Borrower --> UC_SignIn
Borrower --> UC_Browse
Borrower --> UC_Request
Borrower --> UC_History
Borrower --> UC_Cancel
Borrower --> UC_Handover
Borrower --> UC_Return

Manager --> UC_SignIn
Manager --> UC_Review
Manager --> UC_Decide
Manager --> UC_Assign
Manager --> UC_Handover
Manager --> UC_Return
Manager --> UC_Fine
Manager --> UC_Inventory
Manager --> UC_Audit

Admin --> UC_SignIn
Admin --> UC_Access
Admin --> UC_Calendar
Admin --> UC_Audit

SSO --> UC_SignIn

UC_Request ..> UC_Validate : <<include>>
UC_Decide ..> UC_Assign : <<include>>
UC_History ..> UC_SignIn : <<include>>
@enduml
```

## الشرح بالعربية

- **الفاعل الأساسي الأول: Borrower (طالب أو موظف)**: يستعرض التوفر، يقدم طلبًا، يرى السجل والإشعارات، ويلغي الطلب قبل الموافقة. صفتي الطالب والموظف لهما التدفق نفسه بحسب PRD.
- **الفاعل الأساسي الثاني: Equipment Manager**: يراجع الطلبات ضمن نطاقه، يقرر الموافقة أو الرفض، يعيد التحقق من التوفر ويعين القطع، ويسجل التسليم والإرجاع والفحص ويحدث السداد يدويًا.
- **الفاعل الإداري: System Administrator**: دور موصى به في PRD لإدارة نطاقات المسؤولين وتقويم العمل؛ الصلاحيات النهائية تحتاج اعتمادًا مؤسسيًا.
- **الفاعل الثانوي: University SSO**: خدمة خارجية تتحقق من هوية الجامعة. مزود SSO وبروتوكوله غير محددين.
- حالة الاستخدام “Submit borrowing request” تتضمن التحقق من حد الطلبات النشطة، وجود تأخير/غرامة غير مدفوعة، صحة فترة العشرة أيام، ساعات العمل، والتوفر.
- عند قرار الموافقة يعاد فحص التوفر وتُربط القطع الفردية بالطلب. لا توجد حالة استخدام دفع إلكتروني، بريد/SMS أو QR لأنها مستبعدة من MVP.

العلاقة `include` بين القرار وتخصيص القطع تصف أن التخصيص مطلوب ضمن مسار الاعتماد/التسليم، لكنها لا تحسم نقطة التوقيت الدقيقة؛ هذه نقطة مفتوحة في PRD.

## Assumptions

- مستعير الطالب والموظف فاعل واحد لأن السياسة والرحلة لا تختلفان في الوثائق.
- SSO ممثل كفاعل ثانوي خارجي؛ التكامل الفعلي لمزود الهوية لم يُحدد.
- مدير النظام مفصول عن مسؤول المعدات وفق توصية PRD، وليس دورًا مؤسسيًا نهائيًا مؤكدًا.
- تسجيل الغرامة المدفوعة مسؤولية موظف مخول؛ لا تتم معالجة دفع مالي داخل النظام.

## Open questions

- هل يستطيع مسؤول المعدات أيضًا استخدام النظام كمستعير؟
- هل مدير النظام يرى التدقيق لكل الجامعة أم ضمن سياسة محددة؟
- ما النطاق الأولي للحرم/الكلية/القسم؟
- هل طلب الاستعارة يشمل نوعًا واحدًا أم عدة أنواع؟
- ما التوقيت المحدد لتعيين القطع بالنسبة للموافقة والتسليم؟
- من هو مزود SSO وما السمات التي ترجعها الهوية؟
