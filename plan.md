# แผนงาน Final Project — T05 เสนอหัวข้อโครงงาน

> ENGSE203 · Section 2 · ทีม T-05 (คู่) · repo: `engse203-final-2-T-05`
> จัดทำ: 2026-10-15 — ใช้เป็นแผนร่วมของทั้งทีม · แก้ไขได้ผ่าน PR

---

## 1. สถานะและเวลาที่เหลือ

| milestone | กำหนด (Sec 2) | สถานะ |
|---|---|---|
| M0 TEAM.md | 8 ต.ค. 23:59 | ✅ ส่งแล้ว |
| **M1** ตั้งต้นจาก LAB 13 + schema 2 ตาราง + `v0.1.0` | **11 ต.ค. 23:59** | ❌ **เลยกำหนด** |
| **M2** API ของ proposals + login + deploy ครั้งแรก + `v0.5.0` | **16 ต.ค. 23:59** | ⏳ กำลังดำเนินการ |
| หยุดเพิ่มฟีเจอร์ แก้ bug | 20 ต.ค. | — |
| **M3** `v1.0.0` + URL · ส่งใน LMS | **22 ต.ค. ก่อนเวลาสอบ** | 7 วัน |

**กลยุทธ์เวลา:** ไล่ M1 + M2 ให้ทันภายในวันที่ 16 ต.ค. (รวมเป็นงานก้อนเดียว) · 17–20 ต.ค. ทำ test ให้ครบ + เอกสาร + ⭐ · 21 ต.ค. Release Checklist + tag `v1.0.0`

---

## 2. โจทย์ T05 — เสนอหัวข้อโครงงาน

ระบบให้นักศึกษา**ยื่นเสนอหัวข้อโครงงาน** (Web App · Mobile App · AI / Data · IoT ฯลฯ) พร้อมเลือกอาจารย์ที่ปรึกษา · เจ้าหน้าที่หลักสูตร/กรรมการ**ตรวจสอบ · พิจารณา · อนุมัติ · หรือไม่อนุมัติ**

### บทบาทผู้ใช้

| ใคร | ทำอะไรได้ |
|---|---|
| นักศึกษา / คนทั่วไป (ไม่ login) | ดูรายการหัวข้อโครงงาน · กรองตามสถานะ · ดูรายละเอียด · ยื่นเสนอหัวข้อโครงงานใหม่ · ดู dashboard |
| เจ้าหน้าที่หลักสูตร (login) | ทุกอย่างข้างบน + เปลี่ยนสถานะ (submitted → reviewing → approved/rejected) + ลบรายการ |

### สถานะของข้อเสนอโครงงาน

```
submitted ──รับพิจารณา──▶ reviewing ──ผ่านการอนุมัติ──▶ approved
                              │
                              └──ไม่ผ่านเกณฑ์──▶ rejected
```

### กฎเฉพาะของ T05 (ต้องมี test ยืนยัน)

1. **จำนวนสมาชิกในกลุ่ม (`teamSize`) ต้องเป็นจำนวนเต็ม 1–4 คน** และ **ชื่อโครงงาน (`title`) ต้องยาว 10–200 ตัวอักษร**
2. **อาจารย์ที่ปรึกษาที่เลือกต้องมีโควตาคงเหลือ (`availableQuota > 0`)** = `advisors.max_quota − COUNT(*)` ของ proposals ที่สถานะ `submitted` / `reviewing` / `approved` สำหรับอาจารย์ท่านนั้น (ไม่นับ `rejected`)

### การตีความที่ทีมเลือก (เขียนไว้ใน README)

- โครงงานที่ถูกเปลี่ยนสถานะเป็น `rejected` จะ**คืนโควตา**ให้อาจารย์ที่ปรึกษาอัตโนมัติ (สามารถรับกลุ่มใหม่แทนได้)
- การเปลี่ยนสถานะ (`reviewing`, `approved`, `rejected`) **เจ้าหน้าที่หลักสูตรกดเปลี่ยนเอง** ด้วย PUT

---

## 3. จุดเริ่มต้น — ไม่เริ่มจากศูนย์

เริ่มจาก `~/class/port/labs/week-13/source/` ของจักรพงศ์ (LAB 13 Quality & Security) ซึ่งมีครบแล้ว:

- โครง React + Vite + Express + SQLite + JWT
- `requests` resource ของ Campus Service (ต้องแปลงเป็น `proposals`)
- auth middleware · scrypt + salt · `authenticate` · `requireRole`
- CI workflow · validator pure function · test infrastructure (vitest · supertest)

**งานของ M1** คือคัดลอกเข้ามา + วางไฟล์ kit + สร้าง schema ตารางของ T05 + ตั้งค่า config

### คำสั่งคัดลอก (ตามคู่มือ)

```bash
cd ~/engse203-final-2-T-05 && git pull

# ① คัดลอกโค้ด LAB 13 (ไม่เอา node_modules · .db · .env)
rsync -a --exclude node_modules --exclude '*.db' --exclude .env \
  ~/class/port/labs/week-13/source/ ./

# ② วาง kit (-n = ไม่ทับ TEAM.md ที่กรอกแล้ว)
cp -rn ~/class/engse203-lab/labs/final-term-project/kit/. ./

# ③ ลบไฟล์ของ LAB เดิมที่ไม่ใช้
rm -f BUG_REPORTS.md DEMO.md DATABASE_CHOICES.md TEST_CASES.md check-week1*.mjs
rm -rf api/scripts/check-project.mjs api/data/initialRequests.json \
       frontend/public/data frontend/src/services/requestStorage.js

# ④ ไฟล์ลับของเครื่องตัวเอง
cp api/.env.example api/.env

# ⑤ แทน README ที่ GitHub ให้
cp PROJECT_README_TEMPLATE.md README.md

# ⑥ แก้ package.json (root) · บรรทัด "check"
#    → "node --disable-warning=ExperimentalWarning check-project.mjs"

# ⑦ project.config.json และ TEAM_CONTRACT.md — กรอกจาก TEAM.md
```

---

## 4. สถาปัตยกรรม

### 4.1 ตาราง (`api/data/schema.sql`)

```sql
-- users (เก็บ staff + นักศึกษาผู้ยื่นโครงงาน)
CREATE TABLE users (
  id            INTEGER PRIMARY KEY AUTOINCREMENT,
  name          TEXT NOT NULL,
  department    TEXT NOT NULL,
  email         TEXT NOT NULL UNIQUE,
  role          TEXT NOT NULL DEFAULT 'student'
                CHECK (role IN ('student', 'staff')),
  password_hash TEXT
);

-- advisors — อาจารย์ที่ปรึกษาโครงงาน
CREATE TABLE advisors (
  id         TEXT PRIMARY KEY,              -- ADV-001
  name       TEXT NOT NULL,
  max_quota  INTEGER NOT NULL CHECK (max_quota > 0),
  expertise  TEXT NOT NULL,                 -- ความเชี่ยวชาญ
  department TEXT                           -- สาขาวิชา
);

-- proposals — ข้อเสนอหัวข้อโครงงาน (ตารางหลัก — มี ≥ 6 ฟิลด์ รวม status)
CREATE TABLE proposals (
  id            TEXT PRIMARY KEY,           -- PR-001
  student_id    INTEGER NOT NULL,           -- อ้างอิงนักศึกษาผู้ยื่น
  advisor_id    TEXT NOT NULL,              -- อ้างอิงอาจารย์ที่ปรึกษา
  title         TEXT NOT NULL,              -- ชื่อโครงงาน 10–200 ตัวอักษร
  category      TEXT NOT NULL,              -- หมวดหมู่ (Web App / AI / IoT ฯลฯ)
  team_size     INTEGER NOT NULL CHECK (team_size BETWEEN 1 AND 4),
  scope         TEXT NOT NULL,              -- ขอบเขตโครงงาน 15–1000 ตัวอักษร
  status        TEXT NOT NULL DEFAULT 'submitted'
                CHECK (status IN ('submitted','reviewing','approved','rejected')),
  created_at    TEXT NOT NULL DEFAULT (datetime('now','localtime')),
  FOREIGN KEY (student_id) REFERENCES users(id),
  FOREIGN KEY (advisor_id) REFERENCES advisors(id)
);
```

### 4.2 Seed (`schema.sql` ต่อท้าย)

- `users`: นักศึกษา 4 คน + 1 staff (ใช้ตั้ง `STAFF_EMAIL` บน Render ต่อ) — รหัส seed ไว้ dev เท่านั้น
- `advisors`: 5 รายการ (ระบุโควตารับโครงงาน `max_quota` และความเชี่ยวชาญ `expertise`)
- `proposals`: 5 รายการ **ครอบคลุม 4 สถานะ** (`submitted` · `reviewing` · `approved` · `rejected`)

### 4.3 API endpoints

| method | path | ใครเรียก | ผลลัพธ์ |
|---|---|---|---|
| GET  | `/api/health` | ทุกคน | 200 + `{ok, driver, db}` |
| GET  | `/api/proposals` | ทุกคน | 200 · รองรับ `?status=` · **JOIN advisors & users** ให้ได้ `advisorName`, `studentName` · `?q=` (⭐) |
| GET  | `/api/proposals/:id` | ทุกคน | 200 · ไม่พบ → 404 |
| POST | `/api/proposals` | ทุกคน | 201 · body ผิด → 400 + รายการ error · โควตาอาจารย์เต็ม/สมาชิกเกิน → 400 |
| PUT  | `/api/proposals/:id` | staff | 200 · 401/403 ตามปกติ · เปลี่ยน `status` |
| DELETE | `/api/proposals/:id` | staff | 204 · 401/403 |
| GET  | `/api/advisors` | ทุกคน | 200 · ส่งโควตาคงเหลือ (`availableQuota`) มาด้วย · ใช้ป้อน dropdown ฟอร์มเสนอหัวข้อ |
| POST | `/api/auth/login` | ทุกคน | 200 + token · ผิด → 401 (ข้อความเดียวกันทั้ง 2 กรณี) |

### 4.4 Validator (pure functions · `api/src/validators/proposalValidator.js`)

- `studentName` 2–100 ตัวอักษร
- `title` 10–200 ตัวอักษร
- `category` ต้องไม่เป็นค่าว่าง
- `advisorId` ต้องมีในตาราง `advisors` · และ **availableQuota > 0** ตอนตรวจ
- `teamSize` จำนวนเต็ม 1–4 คน
- `scope` 15–1000 ตัวอักษร
- `status` (เฉพาะ PUT) อยู่ใน enum 4 สถานะ (`submitted`, `reviewing`, `approved`, `rejected`)

### 4.5 หน้าเว็บ (React · `frontend/src/pages/`)

| หน้า | path | ใครเห็น | สิ่งที่แสดง |
|---|---|---|---|
| ProposalsList | `/` | ทุกคน | ตาราง/การ์ดข้อเสนอโครงงาน · ตัวกรองสถานะ · ลิงก์ไปรายละเอียด |
| ProposalDetail | `/proposals/:id` | ทุกคน | ข้อมูลโครงงาน + ชื่ออาจารย์ที่ปรึกษา · ปุ่มเปลี่ยนสถานะ/ลบ **เฉพาะ staff** |
| NewProposal | `/new` | ทุกคน | ฟอร์มยื่นหัวข้อ · dropdown อาจารย์จาก API (แสดงโควตาคงเหลือ) · แสดง error จาก API |
| Dashboard | `/dashboard` | ทุกคน | นับจำนวนโครงงานแยกตามสถานะ |
| Login | `/login` | ทุกคน | ฟอร์ม login เจ้าหน้าที่หลักสูตร · 401 → แสดง error เดียวกันทั้ง 2 กรณี |

### 4.6 ความปลอดภัย (จาก LAB 13 — คงไว้)

- scrypt + salt (`utils/password.js`) · JWT หมดอายุ (เช่น 2 ชั่วโมง)
- `authenticate` middleware ตรวจ Bearer token · `requireRole('staff')`
- `JWT_SECRET` จาก env · ไม่มี → fail fast
- production ไม่ส่ง stack trace
- บน Render ตั้ง `STAFF_EMAIL` + `STAFF_PASSWORD` → `upsertStaff` ตอน start (D3)

### 4.7 โครงโฟลเดอร์เป้าหมาย

```
engse203-final-2-T-05/
├── README.md                        (จาก PROJECT_README_TEMPLATE)
├── TEAM.md                          ✅ มีแล้ว
├── PLAN.md                          ← ไฟล์นี้
├── TEAM_CONTRACT.md                 (จาก kit · กรอกเอง)
├── API_CONTRACT.md                  (เขียนร่วมกันใน M1)
├── DEBUG_LOG.md                     (จาก kit · เติม bug จริง ≥ 2)
├── DEPLOY.md                        (คู่ = เลือกทำ · 3 คน = บังคับ)
├── RELEASE_CHECKLIST.md             (จาก kit · ติ๊กก่อน v1.0.0)
├── check-project.mjs                (จาก kit)
├── project.config.json              (กรอกจาก example)
├── render.yaml                      (ลบ rootDir · JWT_SECRET generateValue)
├── .github/workflows/check.yml      (CI)
├── api/
│   ├── .env.example
│   ├── package.json
│   ├── data/schema.sql              (เขียนใหม่ตามข้อ 4.1)
│   ├── src/
│   │   ├── server.js                (เพิ่ม upsertStaff · route /api/advisors)
│   │   ├── config.js                (เปลี่ยนชื่อ db เป็น proposals.db)
│   │   ├── routes/
│   │   │   ├── proposalRoutes.js    (เปลี่ยนชื่อจาก requests)
│   │   │   ├── advisorRoutes.js     (ใหม่)
│   │   │   └── authRoutes.js
│   │   ├── controllers/proposalController.js
│   │   ├── services/
│   │   │   ├── proposalService.js   (เปลี่ยนชื่อ · JOIN advisors · nextId 'PR-')
│   │   │   ├── advisorService.js    (ใหม่ · คำนวณ availableQuota)
│   │   │   └── authService.js
│   │   ├── validators/proposalValidator.js
│   │   └── middleware/auth.js
│   └── tests/
│       ├── unit/ (≥ 10)
│       └── integration/ (≥ 12 · 401 · 403)
└── frontend/
    ├── vite.config.js               (base: "/" · ไม่ใช่ /engse203-student-labs...)
    ├── package.json
    └── src/
        ├── App.jsx
        ├── services/
        │   ├── apiClient.js         (แนบ Authorization ที่เดียว · 401 → logout)
        │   └── proposalService.js
        ├── hooks/useAuth.js
        ├── pages/ (ตามข้อ 4.5)
        ├── components/
        └── utils/proposalSummary.js (+ test ≥ 3)
```

---

## 5. งานในแต่ละ session ต่อจากนี้

### Session ถัดไป (15 ต.ค.) — ปิด M1 + เริ่ม M2

เป้าหมาย: tag `v0.1.0` + schema ของ T05 คอมไพล์ผ่าน

- [ ] คัดลอก LAB 13 เข้า repo + วาง kit (คำสั่งข้อ 3)
- [ ] กรอก `project.config.json` (resource = `/api/proposals` · existingId = `PR-001` · validSample · invalidSample · ruleSamples)
- [ ] กรอก `TEAM_CONTRACT.md`
- [ ] เขียน `schema.sql` ใหม่ตามข้อ 4.1 + seed + ลบตาราง `requests`
- [ ] แก้ `requestService.js` → `proposalService.js` (เปลี่ยน `LIKE 'REQ-%'`, prefix `PR-`, ชื่อตารางใน `loadSeed`)
- [ ] แก้ `config.js` ชื่อฐานข้อมูลเป็น `proposals.db`
- [ ] `npm run db:setup --prefix api` ผ่าน · `/api/health` ตอบ 200
- [ ] เริ่มเขียน `API_CONTRACT.md` ร่วมกับ `<ชื่อเพื่อนคู่ทีม>`
- [ ] commit · push · tag `v0.1.0`

### Session ที่ 2 (15–16 ต.ค.) — ปิด M2

เป้าหมาย: tag `v0.5.0` + deploy ครั้งแรก

จักรพงศ์ (BE+DevOps):
- [ ] controllers · routes · validators ของ proposals ครบ R2 · R3
- [ ] route `/api/advisors` คืน `availableQuota`
- [ ] unit test ≥ 6 (ยังไม่ต้องครบ 10) · integration ≥ 6 (รวม 401 · 403)
- [ ] ตั้ง Render service · `render.yaml` · `STAFF_EMAIL` · `STAFF_PASSWORD` · deploy ได้
- [ ] tag `v0.5.0`

`<ชื่อเพื่อนคู่ทีม>` (FE) ทำขนาน:
- [ ] `apiClient` + `useAuth`
- [ ] หน้า ProposalsList · ProposalDetail · NewProposal · Login
- [ ] ปุ่มเปลี่ยนสถานะ/ลบ ซ่อนให้คนทั่วไป

### Session ที่ 3 (17–20 ต.ค.) — เสริมคุณภาพ

- [ ] unit test ครบ 12 (รวมค่าขอบของกฎ teamSize 1–4 คน · title 10–200 ตัวอักษร · availableQuota)
- [ ] integration test ครบ 13
- [ ] frontend test ≥ 3
- [ ] Dashboard
- [ ] `DEBUG_LOG.md` ≥ 2 bug จริง + regression test
- [ ] CI ผ่าน · เพิ่มป้าย CI ใน README
- [ ] ⭐ เลือก 1–2 ข้อ: ค้นหา (`?q=`) · จำกัดการเดารหัส

### Peer Review (ส่งนอก repo)

- **ครั้งที่ 1 — 17 ต.ค.** (ไม่คิดคะแนน · เตือนกันเอง) · ส่งผ่านฟอร์มที่อาจารย์ประกาศ
- **ครั้งที่ 2 — 22 ต.ค. หลังสัมภาษณ์** (คิดคะแนน I3) · 4 ข้อ + หลักฐาน 1 ตัวอย่างต่อเพื่อน + ประเมินตัวเอง 3 ข้อ

### Session ที่ 4 (21 ต.ค.) — ปิด M3

- [ ] กรอก `README.md` ครบตามแม่แบบ (สมาชิก · ความสามารถ · ภาพหน้าจอ · บัญชีทดสอบ · การใช้ AI)
- [ ] `RELEASE_CHECKLIST.md` ติ๊กครบพร้อมหลักฐาน
- [ ] `check-project.mjs --online` ผ่าน
- [ ] ลบ bug วันสุดท้าย · ห้ามเพิ่มฟีเจอร์
- [ ] tag `v1.0.0` + push · ส่งลิงก์ใน LMS **ก่อนเวลาสอบ 22 ต.ค.**

---

## 6. จุดระวังที่คนเจอบ่อย (จากเอกสารจริง)

ตอนเปลี่ยนชื่อ `requests` → `proposals` มี **4 จุดที่พังแบบเงียบ** ถ้าลืม:

| ไฟล์ | ผูกไว้กับ | ถ้าไม่เปลี่ยน |
|---|---|---|
| `api/src/services/proposalService.js` → `loadSeed()` | `sqlite_master … name='requests'` | `schema.sql` ถูก DROP ทุก restart → ข้อมูลหายทุกครั้ง |
| `proposalService.js` → `nextId()` | `LIKE 'REQ-%'` + prefix `REQ-` | POST แรกได้ 500 หรือรหัสซ้ำ (ต้องเปลี่ยนเป็น `PR-`) |
| `api/scripts/setup-db.mjs` | ข้อความ + FK ของ `requests` | `db:setup` รายงานผิดตาราง |
| `api/src/config.js` · `.env.example` | ไฟล์ `campus.db` | ยังรันได้ แต่ควรเปลี่ยนเป็น `proposals.db` |

**ข้ออื่น:**

- ไม่ commit `node_modules` · `*.db` · `.env` (ตรวจด้วย `git ls-files | grep -E '\.(env|db)$|node_modules'`)
- `JWT_SECRET` บน Render ใช้ `generateValue: true` ใน `render.yaml`
- บน production รหัส staff seed (`staff1234`) **ต้องใช้ไม่ได้** — ตั้งจาก env แทน (ROLES D3)
- Render free tier: SQLite รีเซ็ตทุก redeploy → ยอมรับได้สำหรับทีมคู่ **ต้องเขียนไว้ใน README**

---

## 7. การแบ่งงาน

| งาน | จักรพงศ์ (BE+DevOps) | `<ชื่อเพื่อนคู่ทีม>` (FE) |
|---|---|---|
| schema · service · validator · auth · test ฝั่ง API | ✅ | — |
| หน้าเว็บทั้งหมด · apiClient · login · test ฝั่ง FE | — | ✅ |
| CI · render.yaml · deploy · README · RELEASE_CHECKLIST · DEBUG_LOG | ✅ | — |
| `API_CONTRACT.md` | ร่วมกัน | ร่วมกัน |
| PR · review · commit ≥ 10 คน | ≥ 2 PR + review 1 | ≥ 2 PR + review 1 |

**กฎ:** ทุกงานผ่าน PR + review · commit กระจาย ≥ 5 วัน (ไม่ commit ทั้งหมดคืนสุดท้าย) · ใช้ branch `feature/be-<เรื่อง>` และ `feature/fe-<เรื่อง>`

---

## 8. ⭐ คะแนนเพิ่ม — เลือกได้สูงสุด 2 ข้อ

ที่น่าทำ (ให้ผลคุ้มกับเวลา):

1. **ค้นหา** (`?q=` + ช่องค้นหา) — เพิ่ม WHERE ใน `proposalService.js` · เพิ่ม input ใน ProposalsList · ง่าย คุ้ม
2. **จำกัดการเดารหัส** (ผิด 5 ครั้ง / 15 นาที → 429) — in-memory Map · ไม่ต้องใช้ library · 1–2 ชั่วโมง

ที่ควรเลี่ยงถ้าเวลาน้อย: Turso (เป็นงานของทีม 3 คน) · PWA (service worker) · coverage ≥ 85% (ต้องเขียน test เยอะมาก)

---

## 9. บัญชีทดสอบ (ตั้งใจไว้)

| ที่ไหน | อีเมล | รหัส | หมายเหตุ |
|---|---|---|---|
| dev (จาก seed) | `staff@rmutl.ac.th` | `staff1234` | ใช้แค่ local |
| production | <ตั้งใหม่ · อีเมลเดียวกับ seed ก็ได้> | ≥ 12 ตัว · **ไม่อยู่ใน git** | ตั้งผ่าน Render env + แจ้งผู้สอนทางช่องทางใน LMS |

---

## 10. ลิงก์เอกสารต้นฉบับ

- `~/class/engse203-lab/labs/final-term-project/README.md` — ข้อกำหนด MVP (R1–R8)
- `~/class/engse203-lab/labs/final-term-project/TOPICS.md` — ตาราง T05 + ตัวอย่างแปลง Campus Service
- `~/class/engse203-lab/labs/final-term-project/ROLES.md` — งานละเอียดต่อบทบาท (F1–F9 · B1–B8 · D1–D6)
- `~/class/engse203-lab/labs/final-term-project/RUBRIC.md` — เกณฑ์คะแนน
- `~/class/engse203-lab/labs/final-term-project/kit/` — แม่แบบไฟล์
- `~/class/port/labs/week-13/source/` — จุดเริ่มต้นของโค้ด