# Sigma Platform — โฟล์วการทำงานแยก Module

> อัปเดต: 2026-04-03

---

## ภาพรวม

Sigma เป็นแพลตฟอร์มจัดการ infrastructure บน Azure สำหรับทีม DevOps/Infra  
โฟล์วหลักคือ **Project → Service → Integration → Deploy** โดยมี approval workflow สำหรับ resource requests

### Project-Service-Resource Hierarchy

```
Project (owner, members via ACL)
  ├─ Environment
  │   └─ Service
  │       ├─ ServiceIntegration
  │       ├─ Deployment
  │       │   ├─ Resource (VMs, containers, ...)
  │       │   │   ├─ ResourceDependency
  │       │   │   ├─ UniqueConstraint
  │       │   │   └─ Secret (encrypted)
  │       │   └─ ResourceRequest (admin approval)
  │       └─ Quota
  └─ Provider (infrastructure backends)
```

---

## 1. Project Management

### User Actions
- สร้าง project → user กลายเป็น `ProjectOwner` อัตโนมัติ
- จัดการสมาชิก: add/update/remove ผ่าน ACL
  - POST/PUT/DELETE `/api/projects/[projectUUID]/access-control`
  - Roles ที่ assign ได้: `ProjectContributor`, `ProjectViewer`
  - ไม่สามารถ remove หรือเปลี่ยน role ของ `ProjectOwner`

### Project Roles

| Role | สิทธิ์ |
|---|---|
| `ProjectOwner` | Full control, จัดการ access ได้ทั้งหมด |
| `ProjectContributor` | สร้าง/แก้ไข resources, ไม่มีสิทธิ์จัดการ access |
| `ProjectViewer` | Read-only |

### Quota Management
- Admin ตั้งค่า quota ผ่าน PATCH `/api/projects/[projectUUID]/quota`
- ประเภท quota: **CPU** (vCPUs), **Memory** (GB), **Storage** (GB)
- ข้อจำกัด: ลด quota ไม่ได้ถ้า new quota < current usage
- เก็บแยกกัน: `quotas` (limits) vs `usage` (ค่าปัจจุบัน)

### Data Models
- `Project`: uuid, name (unique per owner), description, ownerId
- `ProjectResourceQuota`: projectId, quotas (JSON), usage (JSON)
- `ACL`: userId, projectId, roleId

---

## 2. Service Management

### 2 แบบของ Service

#### แบบ Project-Scoped
1. User เลือก serviceType และ integrations
2. Service สร้างด้วย status `pending`
3. Admin review → approve/reject
4. เมื่อ approve: status → `active`, SIGMA orchestrate deployments

#### แบบ Environment-Scoped
1. User ตั้งชื่อ service (lowercase, alphanumeric, hyphens, max 100 chars)
2. เลือกผ่าน **Package** (pre-configured) หรือ **ServiceType** (custom)
3. เลือก tech stacks + quota
4. Service สร้างด้วย status `active` (Package) หรือ `pending` (ServiceType)

### Service Status

| Status | ความหมาย |
|---|---|
| `pending` | รอ admin approve หรือ configure |
| `active` | deployments กำลังทำงาน |
| `rejected` | admin ปฏิเสธ (มี rejectDescription) |
| `failed` | deployment ล้มเหลว |
| `destroyed` | cleanup แล้ว |

### ServiceIntegration Status

| Status | ความหมาย |
|---|---|
| `waiting` | สร้างแล้ว ยังไม่ deploy |
| `pending` | กำลัง deploy |
| `completed` | deploy สำเร็จ |
| `failed` | deploy ล้มเหลว |

### Data Models
- `Service`: id, name, type, status, serviceTypeId, environmentId, technologyStackId, quotaId, rejectDescription
- `ServiceIntegration`: serviceId, integrationId, status, destroy flag
- `Package`: name + PackageIntegration + PackageTechStack + PackageQuota

---

## 3. Integration & Deployment Workflow

### ขั้นตอนหลัก (Request → Approve → Deploy)

```
1. User ส่ง Resource Request
   POST /api/projects/[projectUUID]/services/[serviceId]/integrations/[integrationId]/resource-request
   → สร้าง ResourceRequest (status: waiting)
   → สร้าง deployment plan snapshot

2. Admin review ที่ /admin/resource-requests
   → ดู pending requests

3A. Approve:
    POST /api/admin/resource-requests/[requestId] { action: "approve" }
    → validate config
    → ResourceRequest status → approved
    → SIGMA.run() orchestrate deployments

3B. Reject:
    POST /api/admin/resource-requests/[requestId] { action: "reject", reason: "..." }
    → ResourceRequest status → rejected
    → reason บันทึกไว้ (1-500 chars)
```

### ResourceRequest Status

| Status | ความหมาย |
|---|---|
| `waiting` | รอ admin review |
| `approved` | อนุมัติแล้ว SIGMA เริ่ม orchestrate |
| `rejected` | ปฏิเสธพร้อม reason |

### Data Models
- `ResourceRequest`: serviceId, userId, type ("ADD_RESOURCE"), status, config (JSON), planDetails, reason, approvedById, approvedAt
- `Integration`: name, type, subtype, dependencies
- `IntegrationDependency`: dependentId ↔ dependencyId (dependency graph)

---

## 4. Deployment Lifecycle

### Deployment Operations

| Action | Endpoint | Permission |
|---|---|---|
| Start | POST `/api/projects/[projectUUID]/deployments/[id]` | `PROJECT_RESOURCE_CREATE` |
| Get details | GET `/api/projects/[projectUUID]/deployments/[id]` | `PROJECT_RESOURCE_READ` |
| Delete | DELETE `/api/projects/[projectUUID]/deployments/[id]` | Admin |
| Logs | GET `/api/projects/[projectUUID]/deployments/[id]/logs` | - |
| Undo | POST `/api/projects/[projectUUID]/deployments/[id]/undo` | - |

### Deployment Status

| Status | ความหมาย |
|---|---|
| `waiting` | สร้างแล้ว รออยู่ใน queue |
| `planning` | Terraform planning phase |
| `pending` | Plan เสร็จ รอ apply |
| `queued` | อยู่ใน execution queue |
| `running` | กำลัง execute |
| `completed` | สำเร็จ |
| `failed` | ล้มเหลว |
| `invalid` | config ไม่ถูกต้อง |

### Deployment Flags

| Flag | ความหมาย |
|---|---|
| `plan` | plan-only mode (ไม่ apply จริง) |
| `destroy` | กำลัง destroy resources |
| `unregister` | กำลัง unregister |
| `update` | กำลัง update existing resources |
| `import` | กำลัง import existing resources |
| `active` | false = inactive/archived |

### Data Models
- `Deployment`: id, type, name, status, logs (JSON), config (JSON), planOutput, userId, projectId, templateId, providerId, serviceIntegrationId
- `DeploymentQueue`: type (Plan|Apply|Refresh|Destroy), deploymentId, projectId, timestamp

---

## 5. Resource & Quota Management

### Quota Enforcement
- ตั้งค่าระดับ project โดย admin
- ลด quota ไม่ได้ถ้า new quota < current usage
- Track แยก: limits vs actual usage

### Resource Lifecycle
1. สร้างโดย SIGMA orchestrator ตอน deployment
2. เก็บใน `Resource` model พร้อม deploymentId
3. มี dependencies และ constraints
4. ลบเมื่อ deployment ถูก destroy

### UniqueConstraint Scopes

| Scope | ความหมาย |
|---|---|
| `global` | Unique ทั้ง platform |
| `project` | Unique ภายใน project |
| `resource` | Unique ภายใน resource |

### Data Models
- `ProjectResourceQuota`: projectId, quotas (JSON), usage (JSON)
- `ResourceQuotaUsage`: resourceId, usage (JSON)
- `Resource`: uuid, name, type, deploymentId, details, config, tags
- `UniqueConstraint`: key, value, scope
- `ResourceDependency`: dependentId ↔ dependencyId

---

## 6. Access Control / RBAC

### System Roles

| Role | สิทธิ์ |
|---|---|
| `SUPER_ADMIN` | ทุกอย่างทั้ง platform |
| `ADMIN` | เกือบทุกอย่าง ยกเว้น super-admin exclusive |
| `PROJECT_MANAGER` | สร้าง/จัดการ projects |
| `USER` | Base user, read-only |

### Resource-Level Roles

**Project Roles** (priority 1 = สูงสุด):

| Role | Priority |
|---|---|
| `ProjectOwner` | 1 |
| `ProjectContributor` | 2 |
| `ProjectViewer` | 3 |

**Provider Roles**:

| Role | Priority |
|---|---|
| `ProviderOwner` | 1 |
| `ProviderEditor` | 2 |
| `ProviderViewer` | 3 |

### Permissions หลัก

```
ADMIN_USER_MANAGEMENT
PROJECT_CREATE / READ / UPDATE / DELETE
PROJECT_QUOTA_MANAGEMENT
PROJECT_ACCESS_CONTROL
PROJECT_RESOURCE_CREATE / READ / UPDATE / DELETE
PROJECT_RESOURCE_SECRET_READ
PROJECT_SERVICE_CREATE / READ / EDIT / DELETE
PROJECT_ENVIRONMENT_READ / EDIT
PROJECT_ENVIRONMENT_SERVICE_CREATE
PROJECT_PROVIDER_READ
PROJECT_VERSION_CONTROL
PROVIDER_READ / EDIT / ACCESS_CONTROL
TEMPLATE_READ / CREATE / UPDATE / DELETE
```

### User Status

| Status | ความหมาย |
|---|---|
| `PENDING` | รอ verification |
| `VERIFIED` | Active |

### Data Models
- `User`: id, email, name, role, status, password, salt, tokenVersion
- `Role`: id, name, RolePermission[]
- `Permission`: id, name
- `RolePermission`: roleId, permissionId
- `ACL`: userId, projectId/providerId, roleId
- `Group`: id, name, userGroups, ACLs

---

## 7. Providers & Templates

### Provider Workflow
1. Admin สร้าง provider ผ่าน POST `/api/providers/[providerName]`
   - ผูกกับ `TerraformProvider` (terraform source/version)
   - เก็บ config (JSON) พร้อม input field definitions
2. Admin เพิ่ม provider เข้า project → ใช้ใน deployments ได้
3. Secret fields ถูก encrypt ด้วย `SecurityManager.encryptData()`

**Provider Input Types**: String, Number, Boolean, List, File, Map, Providers, Resource, Regex, KubernetesB64

**ลบ Provider ไม่ได้ถ้า**:
- ถูกใช้งานอยู่ใน active deployments
- ยังผูกกับ projects
- มี resource provider role

### Template Workflow
1. Template นิยาม inputs, outputs, resources, actions
2. ผูกกับ TerraformProvider ผ่าน `TemplateProvider`
3. เมื่อสร้าง integration → plan endpoint return deployments ต่อ template
4. Admin approve → SIGMA สร้าง Deployment records ผูกกับ template
5. Deployment execution ใช้ template drive terraform

### Template Plan Flow
```
getIntegrationNodes()
  → DeploymentGraph.getExecutionOrder()
    → mapDeployments()
      → For each template:
          - Fetch template details
          - Extract imports
          - Map to DeploymentTemplate
          - Return with deployment nodes
```

### Data Models
- `Provider`: uuid, name, description, config (JSON), terraformProviderId, resourceId
- `TerraformProvider`: name, details (JSON), type
- `Template`: name, type, description, details (JSON), inputs, outputs, actions, imports
- `TemplateProvider`: templateId ↔ TerraformProviderId
- `ProjectTemplate`: projectUUID ↔ templateId

---

## Service Lifecycle — Full Flow

```
User สร้าง Service
        ↓
Service status: pending (Project) / active (Environment+Package)
ServiceIntegration status: waiting
        ↓
[ถ้า pending] Admin review → Approve / Reject
        ↓ (Approve)
Service status: active
        ↓
User ส่ง Resource Request
ResourceRequest status: waiting
        ↓
Admin approve ResourceRequest
ResourceRequest status: approved
        ↓
SIGMA.run() ด้วย customConfig
        ↓
ServiceIntegration status: pending → completed / failed
Deployment status: waiting → running → completed / failed
```

---

## Notifications

- ใช้ **MS Teams** notification ผ่าน `MSTeamNotification`
- ส่งเมื่อ: service request ถูก approve หรือ reject
