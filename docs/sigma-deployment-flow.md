# Sigma — Deployment Flow (พร้อมตัวอย่างจริง)

> อัปเดต: 2026-04-03

---

## ภาพรวม Flow ทั้งหมด

```
┌─────────────────────────────────────────────────────────────────┐
│                        SIGMA PLATFORM                           │
│                                                                 │
│  👤 User                    🔐 Admin            ⚙️  SIGMA       │
│  ─────────────────          ──────────          ──────────────  │
│                                                                 │
│  1. สร้าง Project           │                  │               │
│     └─ ตั้งชื่อ, สมาชิก    │                  │               │
│          │                  │                  │               │
│  2. สร้าง Service           │                  │               │
│     └─ เลือก tech stack     │                  │               │
│          │                  │                  │               │
│  3. ขอ Integration  ──────► 4. Admin Review    │               │
│     └─ เลือก template       │   ├─ ✅ Approve ─┼──► 5. Deploy  │
│                             │   └─ ❌ Reject   │    │          │
│                             │       │           │    ▼          │
│                             │  แจ้ง User        │   Plan        │
│                             │                  │    │          │
│                             │                  │    ▼          │
│  6. ดู Status ◄─────────────┼──────────────────┼── Apply       │
│     └─ Logs, Outputs        │                  │    │          │
│                             │                  │    ▼          │
│                             │                  │  Resource     │
│                             │                  │  Created ✅   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Flow แยกตาม Step

### Step 1-2: Project & Service

```
User
 │
 ├─► สร้าง Project
 │     name: "my-project"
 │     → User กลายเป็น ProjectOwner อัตโนมัติ
 │
 └─► สร้าง Service ใน Project
       name: "storage-service"
       tech stack: Azure
       status: pending ⏳
```

---

### Step 3: ขอ Integration (Resource Request)

```
User กดขอ Integration
 │
 ├─ เลือก Integration Type: "Azure Storage Account"
 ├─ กรอก Config:
 │     storage_account_name: "mystg2024"
 │     account_tier: "Standard"
 │     replication_type: "LRS"
 │     access_tier: "Hot"
 │
 └─► ระบบสร้าง ResourceRequest
       status: waiting ⏳
       → snapshot plan ไว้ให้ admin ดู
```

---

### Step 4: Admin Review

```
Admin เปิด /admin/resource-requests
 │
 ├─ ดู plan details ของ request
 │
 ├─► ✅ Approve
 │     ResourceRequest status: approved
 │     → ส่งต่อให้ SIGMA ทำงาน
 │
 └─► ❌ Reject
       ResourceRequest status: rejected
       reason: "ชื่อซ้ำกับ storage ที่มีอยู่แล้ว"
       → แจ้ง User ผ่าน MS Teams
```

---

### Step 5: SIGMA Deploy (Terraform)

```
SIGMA ได้รับ approved config
 │
 ├─► Phase 1: PLANNING
 │     สร้าง Terraform config จาก template
 │     run: terraform plan
 │     Deployment status: planning → pending
 │     บันทึก planOutput ไว้ใน DB
 │
 └─► Phase 2: APPLY
       run: terraform apply
       Deployment status: queued → running → completed ✅
       │
       ├─ สร้าง Resource record ใน DB
       └─ บันทึก outputs (endpoint, keys, ...)
```

---

### Step 6: ดูผล

```
User เปิด Service Dashboard
 │
 ├─ Deployment status: completed ✅
 ├─ Logs: terraform apply output
 └─ Resource outputs:
       endpoint: "mystg2024.blob.core.windows.net"
       resource_group: "my-prod-rg"
       location: "eastus"
```

---

## ตัวอย่างจริง: Deploy Azure Storage Account

### สิ่งที่ต้องมีก่อน (Dependencies)

```
Azure Subscription (Provider)
  └─ ต้องมีก่อน ─► Azure Resource Group
                       └─ ต้องมีก่อน ─► Azure Storage Account ← เป้าหมาย
```

> ระบบ deploy ตาม dependency order อัตโนมัติ

---

### Step A: Deploy Azure Resource Group (ก่อน)

**Template:** `azure-resource-group`

```
Inputs ที่ต้องกรอก:
  provider_azurerm:        → [เลือก Azure Provider ที่ตั้งค่าไว้]
  resource_group_name:     "my-prod-rg"
  resource_group_location: "eastus"

Outputs ที่ได้:
  ✓ resource_group_name:     "my-prod-rg"
  ✓ resource_group_location: "eastus"
  ✓ endpoint:                [Azure resource group endpoint]
```

---

### Step B: Deploy Azure Storage Account (หลัก)

**Template:** `azure-storage-account`

```
Inputs ที่ต้องกรอก:
  provider_azurerm:               → [Azure Provider]
  resource_azure-resource-group:  → [my-prod-rg จาก Step A] ← dependency
  storage_account_name:           "mystg2024"
  account_tier:                   "Standard"   (Standard | Premium)
  replication_type:               "LRS"        (LRS | GRS | RAGRS | ZRS)
  access_tier:                    "Hot"        (Hot | Cool)

  [auto จาก dependency]
  rg_name:     "my-prod-rg"   ← ดึงจาก Resource Group output
  rg_location: "eastus"       ← ดึงจาก Resource Group output

Outputs ที่ได้:
  ✓ storage_account_name:              "mystg2024"
  ✓ storage_account_id:                "/subscriptions/.../mystg2024"
  ✓ storage_account_resource_group:    "my-prod-rg"
  ✓ storage_account_location:          "eastus"
  ✓ storage_account_tier:              "Standard"
  ✓ storage_account_replication_type:  "LRS"
  ✓ storage_account_primary_access_key: [🔐 encrypted]
```

---

### Deployment Status Timeline

```
00:00  ResourceRequest สร้าง        status: waiting ⏳
00:01  Admin approve                status: approved ✅
00:02  SIGMA เริ่มทำงาน
00:03  Resource Group deploy        status: waiting → running
00:05  Resource Group สำเร็จ        status: completed ✅
00:06  Storage Account deploy       status: waiting → planning
00:07  terraform plan เสร็จ         status: pending
00:08  terraform apply เริ่ม        status: running
00:12  Storage Account สำเร็จ       status: completed ✅
       → Resource record สร้างใน DB
       → Outputs บันทึกเรียบร้อย
```

---

### สิ่งที่เกิดขึ้นใน Database

```
Deployment record:
  id:       "clp4k8x2i..."
  name:     "azure-storage-account-mystg2024"
  status:   "completed"
  templateId: "azure-storage-account"
  config:   { storage_account_name: "mystg2024", ... }
  logs:     { terraform_plan: "...", terraform_apply: "..." }

Resource record:
  uuid:  "resource-uuid-789"
  name:  "mystg2024"
  type:  "azure-storage-account"
  config: {
    endpoint:           "mystg2024.blob.core.windows.net"
    resource_group:     "my-prod-rg"
    location:           "eastus"
    account_tier:       "Standard"
    replication_type:   "LRS"
  }

UniqueConstraint:
  key:   "storage_account_name"
  value: "mystg2024"
  scope: "resource"    ← ป้องกันชื่อซ้ำ
```

---

## Deployment Status ทั้งหมด

```
waiting ──► planning ──► pending ──► queued ──► running ──► completed ✅
                                                         └──► failed ❌
                                                         └──► invalid ⚠️
```

| Status | ความหมาย |
|---|---|
| `waiting` | รออยู่ใน queue |
| `planning` | กำลัง terraform plan |
| `pending` | plan เสร็จ รอ apply |
| `queued` | อยู่ใน execution queue |
| `running` | กำลัง terraform apply |
| `completed` | สำเร็จ ✅ |
| `failed` | ล้มเหลว ❌ |
| `invalid` | config ผิดพลาด ⚠️ |

---

## Provider ที่รองรับ

| Provider | ตัวอย่าง Resources |
|---|---|
| Azure | Resource Group, Storage Account, Container Registry, AKS, ... |
| Proxmox | Virtual Machine, Container |
| Docker | Container, Volume, Network |
| Nginx | Virtual Host, SSL |
| Terraform | Generic Terraform module |
| Nutanix | VM, Cluster |
| Kubernetes | Deployment, Secret, Service |
