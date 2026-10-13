# BorrowHub MVP — Design-Level Class Diagram

## Purpose

يعرض مخطط الفئات التصميمي بنية الكيانات وحقولها الأساسية وروابطها وكارديناليتها، مستندًا إلى نموذج DBML. هو وصف بنيوي تحليلي فقط؛ لا يتضمن كود تطبيق أو تفاصيل إطار عمل أو بنية نشر.

## Source requirements

- `docs/PRD.md`: Data Requirements, Roles & Permissions, FR-001–FR-030.
- `docs/user-flow.md`: تدفق الطلب والموافقة والتسليم والإرجاع.
- `docs/ERD.md` و`docs/database.dbml`: الجداول وPK/FK والعلاقات.

## Valid code

```mermaid
classDiagram
    class User {
        +UUID userId
        +String ssoSubject
        +String displayName
        +Affiliation affiliation
        +UserStatus status
        +authenticateWithSSO()
    }
    class Role {
        +Integer roleId
        +RoleCode code
    }
    class UserRoleAssignment {
        +DateTime assignedAt
    }
    class OrganizationalUnit {
        +UUID unitId
        +String unitName
        +ScopeType unitType
        +Boolean isActive
    }
    class ManagerScopeAssignment {
        +DateTime assignedAt
        +Boolean isActive
    }
    class EquipmentCategory {
        +UUID categoryId
        +String categoryName
    }
    class EquipmentType {
        +UUID equipmentTypeId
        +String typeName
        +Boolean isActive
        +getCatalogCount()
        +getAvailability(period)
    }
    class AssetUnit {
        +UUID assetUnitId
        +String assetTag
        +AssetStatus operationalStatus
        +setStatusAfterInspection(status)
    }
    class LoanRequest {
        +UUID loanRequestId
        +Integer requestedQuantity
        +DateTime pickupAt
        +DateTime dueAt
        +LoanRequestStatus status
        +submit()
        +cancelBeforeApproval()
        +approveAfterAvailabilityCheck()
        +reject()
    }
    class AssetAssignment {
        +UUID assignmentId
        +DateTime assignedAt
        +DateTime handoverAt
        +DateTime returnedAt
        +recordHandover()
        +recordReturn()
    }
    class ReturnInspection {
        +UUID inspectionId
        +InspectionResult result
        +DateTime inspectedAt
        +recordResult()
    }
    class Fine {
        +UUID fineId
        +Decimal amount
        +String currency
        +FineStatus status
        +DateTime issuedAt
        +DateTime paidAt
        +recordManualPayment()
    }
    class Notification {
        +UUID notificationId
        +NotificationType type
        +DateTime createdAt
        +DateTime readAt
    }
    class OperatingCalendar {
        +UUID calendarId
        +String timeZoneName
        +isValidPickupOrReturn(time)
    }
    class WeeklyHours {
        +Integer weekday
        +Time opensAt
        +Time closesAt
    }
    class Holiday {
        +Date holidayDate
        +String holidayName
    }
    class AuditEvent {
        +UUID auditEventId
        +String actionCode
        +String entityType
        +UUID entityId
        +DateTime occurredAt
    }

    User "1" --> "0..*" UserRoleAssignment : has
    Role "1" --> "0..*" UserRoleAssignment : assigned through
    User "1" --> "0..*" ManagerScopeAssignment : receives
    OrganizationalUnit "1" --> "0..*" ManagerScopeAssignment : scope
    OrganizationalUnit "0..1" --> "0..*" OrganizationalUnit : parent

    EquipmentCategory "1" --> "0..*" EquipmentType : category
    EquipmentType "1" --> "0..*" AssetUnit : physical units
    OrganizationalUnit "1" --> "0..*" AssetUnit : inventory scope
    User "1" --> "0..*" LoanRequest : borrower
    EquipmentType "1" --> "0..*" LoanRequest : requested type
    OrganizationalUnit "1" --> "0..*" LoanRequest : request scope
    User "0..1" --> "0..*" LoanRequest : decision maker

    LoanRequest "1" --> "0..*" AssetAssignment : allocations
    AssetUnit "1" --> "0..*" AssetAssignment : allocation history
    User "1" --> "0..*" AssetAssignment : assigned by
    AssetAssignment "1" --> "0..1" ReturnInspection : inspection
    User "1" --> "0..*" ReturnInspection : inspected by
    LoanRequest "1" --> "0..1" Fine : fine
    User "0..1" --> "0..*" Fine : records payment

    User "1" --> "0..*" Notification : recipient
    LoanRequest "1" --> "0..*" Notification : request context
    OrganizationalUnit "1" --> "0..1" OperatingCalendar : calendar
    OperatingCalendar "1" --> "0..*" WeeklyHours : hours
    OperatingCalendar "1" --> "0..*" Holiday : holidays
    User "0..1" --> "0..*" AuditEvent : actor
```

## الشرح بالعربية

المخطط يترجم النموذج المفاهيمي إلى فئات ذات خصائص وأنواع بيانات مفهومة، مع عمليات نطاقية مختصرة مثل إلغاء طلب قبل الموافقة وتسجيل التسليم أو الدفع. هذه أسماء سلوكية تحليلية وليست توقيعًا نهائيًا لواجهات برمجية.

- `User`, `Role`, و`UserRoleAssignment` يمثلون الهوية والصلاحيات دون اعتبار الطالب والموظف صنفين برمجيين منفصلين.
- `EquipmentType` يمثل عنصر الكتالوج، و`AssetUnit` كل قطعة فعلية. `getCatalogCount()` و`getAvailability(period)` قيم مشتقة؛ لا تخزن كنسخ مكررة.
- `LoanRequest` يدير دورة الطلب/الاستعارة. `AssetAssignment` يربط الطلب بالقطع الفعلية وسجلات الحيازة.
- `ReturnInspection` و`Fine` منفصلان عن الطلب لأنهما سجلات اختيارية/تابعة وتملك بيانات دورة خاصة.
- `OperatingCalendar`, `WeeklyHours`, و`Holiday` تدعم التحقق من المواعيد.
- `AuditEvent` يربط الإجراء بفاعل وهدف، مع ملاحظة أن مرجع الهدف من نوع متعدد لا يمثل FK تقليديًا.

## Assumptions

- الخصائص المعروضة حد أدنى مفاهيمي من DBML؛ لم يحدد المشروع قاعدة البيانات أو طريقة إنشاء UUID.
- `LoanRequest` يجمع الطلب والقرض لتجنب ازدواج السجلات في MVP.
- عمليات الفئات تصف سلوكًا مشتقًا من المتطلبات فقط؛ لا تستلزم تصميم تطبيق أو شيفرة.
- نوع الطلب واحد لكل سجل؛ الطلبات متعددة المعدات تتطلب `LoanRequestItem` إضافيًا.

## Open questions

- ما نطاق جدول المؤسسة والوحدات عند إطلاق المشروع؟
- هل تُمنح الأدوار متعددة للمستخدم، أم تُفرض حصريتها؟
- هل يستطيع الموظف المسؤول أن يكون مستعيرًا أيضًا؟
- ما توقيت تعيين القطع ودورة إغلاق الاستعارة؟
- ما هو مصدر الحقيقة للعدد والحالة الكلية المعروضة للنوع؟ التوصية اشتقاقهما من الأصول الفردية.
- ما المنطقة الزمنية وقواعد التعامل مع تغير التقويم بعد تقديم الطلب؟
