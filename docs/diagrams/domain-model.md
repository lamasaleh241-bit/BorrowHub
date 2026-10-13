# BorrowHub MVP — Domain Model

## Purpose

يعرض نموذج المجال مفاهيم BorrowHub الأساسية وعلاقاتها على مستوى الأعمال، دون ربطها بتقنية تنفيذ أو تحويلها إلى مخطط قاعدة بيانات حرفي. يركز على المستخدم، الأدوار، نطاقات الجامعة، المعدات الفردية، الطلب/الاستعارة، التخصيص، الفحص، الغرامة والإشعارات.

## Source requirements

- `docs/PRD.md`: Product Overview, Target Users, User Stories, Functional Requirements FR-001–FR-030, Data Requirements, Roles & Permissions.
- `docs/user-flow.md`: تسجيل الطلب ومراجعته وتخصيص الأصل والتسليم والإرجاع والتأخير.
- `docs/ERD.md` و`docs/database.dbml`: الكيانات والعلاقات المقترحة؛ يُستخدمان مرجعًا لا مصدرًا لقواعد أعمال إضافية.

## Valid code

```mermaid
classDiagram
    class UniversityUser {
        +SSO subject
        +display name
        +affiliation student staff unknown
        +account status
    }
    class Role {
        +borrower
        +equipment manager
        +system administrator recommended
    }
    class UserRoleAssignment {
        +assigned at
    }
    class OrganizationalUnit {
        +unit name
        +scope type campus faculty department other
    }
    class ManagerScopeAssignment {
        +assigned at
        +active status
    }
    class EquipmentCategory {
        +category name
    }
    class EquipmentType {
        +type name
        +active status
    }
    class AssetUnit {
        +unique asset tag
        +operational status
    }
    class LoanRequest {
        +requested quantity
        +pickup time
        +return due time
        +request status
    }
    class AssetAssignment {
        +assigned at
        +handover time
        +return time
    }
    class ReturnInspection {
        +result good damaged incomplete or missing
        +inspection time
    }
    class Fine {
        +amount USD 50
        +status unpaid or paid
        +issued time
    }
    class Notification {
        +notification type
        +created time
        +read time
    }
    class OperatingCalendar {
        +time zone
        +active status
    }
    class WeeklyHours {
        +weekday
        +opens at
        +closes at
    }
    class Holiday {
        +date
        +name
    }

    UniversityUser "1" --> "0..*" UserRoleAssignment : has roles
    Role "1" --> "0..*" UserRoleAssignment : assigned through
    UniversityUser "1" --> "0..*" ManagerScopeAssignment : receives manager scope
    OrganizationalUnit "1" --> "0..*" ManagerScopeAssignment : assigned scope
    OrganizationalUnit "0..1" --> "0..*" OrganizationalUnit : parent child hierarchy
    OrganizationalUnit "1" --> "0..*" AssetUnit : owns inventory
    EquipmentCategory "1" --> "0..*" EquipmentType : groups
    EquipmentType "1" --> "0..*" AssetUnit : describes physical units
    UniversityUser "1" --> "0..*" LoanRequest : submits
    EquipmentType "1" --> "0..*" LoanRequest : requested type
    OrganizationalUnit "1" --> "0..*" LoanRequest : routes to scope
    LoanRequest "1" --> "0..*" AssetAssignment : allocates units
    AssetUnit "1" --> "0..*" AssetAssignment : assigned over time
    AssetAssignment "1" --> "0..1" ReturnInspection : has inspection
    LoanRequest "1" --> "0..1" Fine : may incur
    UniversityUser "1" --> "0..*" Notification : receives
    LoanRequest "1" --> "0..*" Notification : context for
    OrganizationalUnit "1" --> "0..1" OperatingCalendar : operates under
    OperatingCalendar "1" --> "0..*" WeeklyHours : defines
    OperatingCalendar "1" --> "0..*" Holiday : defines
```

## الشرح بالعربية

- **UniversityUser** يمثل هوية طالب أو موظف بعد الدخول عبر SSO. الدور والصِفة مفهومان منفصلان: affiliation قد تكون طالبًا أو موظفًا، بينما Role يحدد صلاحية Borrower أو Equipment Manager أو System Administrator الموصى به.
- **OrganizationalUnit** يمثل نطاق المخزون والإدارة، وقد يكون الحرم أو الكلية أو القسم. العلاقة الذاتية تسمح بهيكل هرمي من دون فرض أن الإطلاق يغطي الجامعة كلها.
- **EquipmentType** يمثل تعريفًا في الكتالوج، أما **AssetUnit** فيمثل كل قطعة فعلية ذات معرّف منفرد. عدد القطع وحالة توفرها يعتمدان على القطع الفردية والتخصيصات؛ لا ينبغي تكرار الإجمالي يدويًا.
- **LoanRequest** يحمل نوع المعدّة والكمية والفترة وحالة دورة الطلب. بعد الموافقة يُدار كسجل الاستعارة نفسها لتفادي تكرار البيانات.
- **AssetAssignment** يربط الطلب بالقطع الفعلية؛ هذه علاقة متعددة إلى متعددة عبر الزمن. كل تخصيص يوثق أوقات الإسناد والتسليم والإرجاع.
- **ReturnInspection** يوثق نتيجة فحص القطعة بعد الإرجاع. يقرر مسؤول المعدات حالة القطعة النهائية يدويًا.
- **Fine** اختيارية لكل استعارة، وبحد أقصى غرامة واحدة. الدفع يدوي في MVP.
- **OperatingCalendar** يربط نطاق التشغيل بساعات العمل والعطل. توقيت المنطقة الرسمية ما زال سؤالًا مفتوحًا.

المخطط يميز بين العلاقات المفاهيمية وقرارات قاعدة البيانات. لا يعني أن المستخدمين أنواع فرعية مختلفة في البرمجيات؛ الأدوار تُسنَد للمستخدمين، وقد يُمنح المستخدم أكثر من دور إذا سمحت السياسة.

## Assumptions

- كل طلب يحتوي نوع معدّة واحدًا وكمية واحدة مطلوبة، بما يتفق مع صياغة المقابلة الحالية. إذا جاز وضع عدة أنواع في طلب واحد، يلزم مفهوم Request Item منفصل.
- الجامعة الواحدة تمثل الجذر المؤسسي، وقد تكون نطاقات الحرم/الكلية/القسم مختلفة بعد قرار الإطلاق.
- الطلب والاستعارة يمثلان دورة واحدة في هذا الـMVP، ولا يُنشأ كيان قرض مكرر.
- أدوار مدير النظام وفصل الصلاحيات توصية مذكورة في PRD، وليست سياسة مؤسسية مثبتة بالكامل.

## Open questions

- هل يبدأ الإطلاق بالحرم كله أم كلية/قسم محدد؟
- كيف يُختار نطاق الاستلام إذا كان النوع نفسه متوفرًا في أكثر من وحدة؟
- هل الطلب المعلق يحجز كمية مؤقتًا؟
- متى تنتهي صفة الاستعارة النشطة: عند تسجيل الإرجاع أم بعد الفحص؟
- هل يجب أن يمكن لطلب واحد تضمين عدة أنواع معدات؟
- ما المنطقة الزمنية المعتمدة لكل نطاق؟
