# STEPS — คู่มือทำตามข้อ · T05 เสนอหัวข้อโครงงาน

> ติ๊ก `[x]` เมื่อทำเสร็จ · ทุกข้อมี **คำสั่งที่ต้องรัน** + **วิธีตรวจว่าผ่าน** · อ่าน [PLAN.md](PLAN.md) ก่อนถ้ายังไม่เข้าใจภาพรวม

**ตัวย่อ:** `BE` = จักรพงศ์ · `FE` = `<ชื่อเพื่อนคู่ทีม>` · `BOTH` = ทำร่วมกัน
**เครื่องหมาย:** ⚠ = จุดระวังพังเงียบ · 🏷 = ต้อง tag

---

# M1 · ตั้งต้นจาก LAB 13 → `v0.1.0`

> เป้าหมาย: โค้ด LAB 13 ของ BE อยู่ใน repo ทีม · schema เป็นของ T05 แล้ว · `/api/health` ตอบ 200
> ควรจบภายในวันนี้ (15 ต.ค.)

## M1.1 · ตรวจ repo ก่อนเริ่ม `BE`

```bash
cd ~/engse203-final-2-T-05
git pull origin main                 # เอางานของเพื่อนในทีมมา (ถ้ามี)
git status                           # ต้อง clean
ls                                   # ตอนนี้เห็นแค่ TEAM.md + PLAN.md + STEPS.md
```

- [ ] ✅ ผ่านเมื่อ: `git status` บอก `nothing to commit`

## M1.2 · ตรวจ LAB 13 ว่าผ่าน 23/23 `BE`

```bash
cd ~/class/port/labs/week-13/source
node --disable-warning=ExperimentalWarning check-week13.mjs --inclass
```

- [ ] ✅ ผ่านเมื่อ: ขึ้น `23/23` · ไม่งั้นต้องแก้ให้ครบก่อน (เป็นคะแนน A5 อยู่แล้ว)

## M1.3 · คัดลอกโค้ด LAB 13 เข้า repo ทีม `BE`

```bash
cd ~/engse203-final-2-T-05
rsync -a --exclude node_modules --exclude '*.db' --exclude .env \
  ~/class/port/labs/week-13/source/ ./
```

- [ ] ✅ ผ่านเมื่อ: `ls` เห็น `api/` `frontend/` `package.json` `render.yaml` · **ไม่เห็น** `node_modules/` หรือ `*.db`

## M1.4 · วาง kit (ไม่ทับ TEAM.md) `BE`

```bash
cp -rn ~/class/engse203-lab/labs/final-term-project/kit/. ./
# บังคับทับ workflow CI ให้เป็นของ Final Project
cp -f ~/class/engse203-lab/labs/final-term-project/kit/.github/workflows/check.yml \
      .github/workflows/check.yml
ls
```

- [ ] ✅ ผ่านเมื่อ: เห็น `check-project.mjs` · `project.config.example.json` · `DEBUG_LOG.md` · `DEPLOY.md` · `RELEASE_CHECKLIST.md` · `TEAM_CONTRACT.md` · `PROJECT_README_TEMPLATE.md`
- [ ] TEAM.md ไม่ถูกทับ (`cat TEAM.md | head -5` ยังเห็นชื่อทีม T-05)

## M1.5 · ลบไฟล์ LAB เดิมที่ไม่ใช้ `BE`

```bash
rm -f BUG_REPORTS.md DEMO.md DATABASE_CHOICES.md TEST_CASES.md check-week1*.mjs
rm -rf api/scripts/check-project.mjs api/data/initialRequests.json \
       frontend/public/data frontend/src/services/requestStorage.js
```

- [ ] ✅ ผ่านเมื่อ: `ls` ไม่เห็นไฟล์ 7 ตัวข้างบน

## M1.6 · สร้าง `.env` ของเครื่องตัวเอง `BE`

```bash
cp api/.env.example api/.env
echo "JWT_SECRET=$(openssl rand -hex 32)" >> api/.env
# ไม่ commit — อยู่ใน .gitignore แล้ว
```

- [ ] ✅ ผ่านเมื่อ: `cat api/.env | grep JWT_SECRET` เห็นค่า (ไม่ว่าง)
- [ ] `git status` ไม่เห็น `api/.env` อยู่ในรายการ untracked

## M1.7 · แทน README `BE`

```bash
cp PROJECT_README_TEMPLATE.md README.md
# ลบคอมเมนต์ <!-- ... --> และกรอกชื่อระบบตั้งต้น (เติมภายหลังก็ได้)
```

- [ ] ✅ ผ่านเมื่อ: `head README.md` ขึ้น `# เสนอหัวข้อโครงงาน...` (หรือตามที่กรอก)

## M1.8 · แก้ `package.json` ที่ root `BE`

เปิด `package.json` ที่ root · หา key `"check"` แล้วเปลี่ยนเป็น:

```json
"check": "node --disable-warning=ExperimentalWarning check-project.mjs"
```

- [ ] ✅ ผ่านเมื่อ: `cat package.json | grep '"check"'` แสดงบรรทัดนี้

## M1.9 · กรอก `project.config.json` `BE`

```bash
cp project.config.example.json project.config.json
```

เปิดแก้ (ดูโค้ดเต็มใน `BUILD.md` หัวข้อ M1-C17): `resource = "/api/proposals"` · `existingId = "PR-001"` · `statusField = "status"` · `statusFilter = "submitted"` · `statusUpdate = {status: "reviewing"}` · `validSample`/`invalidSample`/`ruleSamples` ตามจริง

- [ ] ✅ ผ่านเมื่อ: ไฟล์ไม่มีคำว่า `requests` หรือ `REQ-` อยู่เลย

## M1.10 · กรอก `TEAM_CONTRACT.md` `BOTH`

เปิดกรอก: สมาชิก · บทบาท · กฎการ merge · ช่องทางคุยกัน · เวลาส่งงานประจำวัน

- [ ] ✅ ผ่านเมื่อ: ไม่มีช่องว่าง `<...>` ค้างในไฟล์

## M1.11 · เขียน `api/data/schema.sql` ใหม่ `BE`

ลบตาราง `requests` ทั้งหมด · สร้าง 3 ตาราง:
`users` (นักศึกษา + staff) · `advisors` (อาจารย์ที่ปรึกษา + `max_quota`) · `proposals` (ข้อเสนอโครงงาน มี FK ไป `users` และ `advisors`)

ดูเนื้อหา SQL เต็มใน [BUILD.md §M1-C3](BUILD.md)

- [ ] seed อย่างน้อย 1 staff + 5 advisors + 5 proposals (ครบ 4 สถานะ: `submitted`, `reviewing`, `approved`, `rejected`)
- [ ] ✅ ผ่านเมื่อ: `grep -c 'INSERT INTO proposals' api/data/schema.sql` หรือจำนวนแถวใน VALUES ≥ 5
- [ ] `grep 'CREATE TABLE requests' api/data/schema.sql` ไม่พบ (ลบแล้ว)

## M1.12 · ⚠ แก้ `proposalService.js` 4 จุดพังเงียบ `BE`

เปลี่ยนชื่อไฟล์ก่อน:
```bash
git mv api/src/services/requestService.js api/src/services/proposalService.js
```

จุดที่ **ต้อง** เปลี่ยน (ไม่งั้นข้อมูลหายทุก restart — ดูโค้ดเต็มใน `BUILD.md` §M1-C6):

| จุด | ของเดิม | ของใหม่ |
|---|---|---|
| 1. `loadSeed()` | `WHERE name='requests'` | `WHERE name='proposals'` |
| 2. `nextId()` | `LIKE 'REQ-%'` | `LIKE 'PR-%'` |
| 2. `nextId()` | prefix `REQ-` | prefix `PR-` |
| 3. ชื่อตาราง & ฟังก์ชันโควตา | `requests` | `proposals` + เพิ่ม `availableQuotaFor(advisorId)` |
| 4. SELECT ภายใน | `FROM requests` | `FROM proposals p LEFT JOIN users u ... LEFT JOIN advisors a ...` |

- [ ] ✅ ผ่านเมื่อ: `grep -n "requests\|REQ-" api/src/services/proposalService.js` ไม่มีผล

## M1.13 · ⚠ แก้ import ที่อื่นให้ตรงชื่อใหม่ `BE`

เปลี่ยนชื่อไฟล์ที่เหลือ:
```bash
cd ~/engse203-final-2-T-05/api/src
git mv controllers/requestController.js controllers/proposalController.js
git mv routes/requestRoutes.js          routes/proposalRoutes.js
git mv validators/requestValidator.js   validators/proposalValidator.js
```

ตรวจเช็ก import ที่ค้างอยู่:
```bash
grep -rln "requestService\|requestController\|requestValidator\|requestRoutes" api/src
# ทุกไฟล์ที่เจอ ต้องแก้ import (เช่น authService.js และ app.js)
```

- [ ] ✅ ผ่านเมื่อ: `grep -rn "requestService\|requestValidator\|requestController" api/src` ไม่พบ

## M1.14 · ⚠ แก้ `setup-db.mjs` `BE`

```bash
grep -n "requests" api/scripts/setup-db.mjs
# แก้ข้อความและชื่อตารางเป็น proposals
```

- [ ] ✅ ผ่านเมื่อ: `grep "requests" api/scripts/setup-db.mjs` ไม่พบ (หรือเจอแค่ในคอมเมนต์)

## M1.15 · ⚠ แก้ `config.js` + `.env.example` ชื่อ db `BE`

เปลี่ยน `campus.db` → `proposals.db` ใน:
- `api/src/config.js`
- `api/.env.example`
- `api/.env` (ของเครื่องตัวเอง)
- `.gitignore` (ถ้าระบุชื่อไฟล์ตรง)

- [ ] ✅ ผ่านเมื่อ: `grep -rn "campus.db\|campus.sqlite" api/` ไม่พบ

## M1.16 · ติดตั้ง + รัน db:setup `BE`

```bash
npm install --prefix api
npm install --prefix frontend
npm run db:setup --prefix api
```

- [ ] ✅ ผ่านเมื่อ: `ls api/data/*.db` เห็น `proposals.db`
- [ ] ✅ `sqlite3 api/data/proposals.db "SELECT COUNT(*) FROM proposals;"` ≥ 5

## M1.17 · เปิด API ทดสอบ `BE`

```bash
npm run dev --prefix api &
sleep 2
curl http://localhost:3001/api/health
curl http://localhost:3001/api/proposals
curl http://localhost:3001/api/proposals/PR-001
```

- [ ] ✅ `/api/health` ตอบ `{"ok":true,...}`
- [ ] ✅ `/api/proposals` ตอบ array ≥ 5 รายการ **พร้อม advisorName และ studentName** (JOIN ทำงาน)
- [ ] ✅ `/api/proposals/PR-001` ตอบ 200
- [ ] ปิด server: `kill %1`

## M1.18 · เริ่ม `API_CONTRACT.md` `BOTH`

สร้างไฟล์ `API_CONTRACT.md` เขียนทุก endpoint ใน PLAN §4.3 · ตัวอย่าง request/response · status code · ข้อความ error

> เพื่อนฝั่ง FE เริ่มทำหน้าเว็บด้วย mock data ตาม contract นี้ได้ ไม่ต้องรอ BE เสร็จ

- [ ] ✅ ผ่านเมื่อ: ครบ 8 endpoints ใน PLAN §4.3

## M1.19 · Commit + tag `v0.1.0` 🏷 `BE`

```bash
rm -rf api/node_modules frontend/node_modules api/data/*.db
git add -A
git commit -m "M1: migrate LAB 13 to T05 proposals + advisors schema"
git tag v0.1.0
git push origin main
git push origin v0.1.0
```

- [ ] ✅ ผ่านเมื่อ: `git tag` เห็น `v0.1.0` · บน GitHub เห็น tag แล้ว

---

# M2 · API + Login + Deploy ครั้งแรก → `v0.5.0`

> เป้าหมาย: ครบ R2 ใน API + หน้า login ใช้ได้ใน local + deploy Render ครั้งแรก
> **กำหนด: 16 ต.ค. 23:59**

## ส่วนของ BE

### M2.B1 · `advisorService.js` + route `BE`

สร้างไฟล์ `api/src/services/advisorService.js` · function `listAdvisors()` ที่ SELECT advisors + คำนวณ `availableQuota = max_quota - COUNT(proposals ที่ status IN ('submitted','reviewing','approved'))` (ไม่นับ `rejected`)

สร้าง route `api/src/routes/advisorRoutes.js` และ mount ที่ `/api/advisors` ใน `app.js` (ดูโค้ดใน `BUILD.md` §M1-C7, §M1-C11, §M1-C12)

- [ ] ✅ `curl localhost:3001/api/advisors` คืน array ที่มี `availableQuota` ทุกตัว
- [ ] ✅ `availableQuota` คำนวณถูกต้องและไม่ติดลบ

### M2.B2 · `proposalValidator.js` `BE`

สร้าง pure function `validateProposalInput(input, { advisors })` (ดูโค้ดใน `BUILD.md` §M1-C8):

- `studentName` 2–100 ตัวอักษร
- `title` 10–200 ตัวอักษร
- `category` ต้องไม่ว่าง
- `advisorId` ต้องมีใน `advisors` ที่ส่งมา + **availableQuota > 0**
- `teamSize` จำนวนเต็ม 1–4 คน
- `scope` 15–1000 ตัวอักษร

คืน array ของข้อความ error (`errors`)

- [ ] ✅ pure function (ไม่มี side effect · ไม่เรียก db โดยตรง)

### M2.B3 · เรียก validator ใน controller `BE`

ใน `createProposal` controller: ดึง `advisors` จาก `advisorService.listAdvisors()` → ส่งเข้า `validateProposalInput(req.body, { advisors })` → ถ้ามี error ตอบ 400 + `{ error, details: errors }`

- [ ] ✅ `curl -X POST -d '{"teamSize":10,...}'` ตอบ 400 + ข้อความแจ้งเตือนจำนวนสมาชิก

### M2.B4 · POST ของจริง `BE`

```bash
curl -X POST http://localhost:3001/api/proposals \
  -H "Content-Type: application/json" \
  -d '{"studentName":"ทดสอบ ระบบ","advisorId":"ADV-001","title":"ระบบจัดการโครงงานวิศวกรรมซอฟต์แวร์ออนไลน์","category":"Web App","teamSize":2,"scope":"พัฒนาเว็บแอปพลิเคชันสำหรับยื่นหัวข้อโครงงานและติดตามสถานะ"}'
```

- [ ] ✅ ตอบ 201 + object มี `id = PR-00X`
- [ ] ✅ ลอง POST ซ้ำ · id เพิ่มขึ้น (`PR-006`, `PR-007`, ...)

### M2.B5 · auth ยังทำงาน `BE`

```bash
# login (รหัสจาก seed)
TOKEN=$(curl -s -X POST localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"staff@rmutl.ac.th","password":"staff1234"}' | jq -r .token)

# ลองลบโดยไม่มี token → 401
curl -i -X DELETE localhost:3001/api/proposals/PR-004

# ลองลบด้วย token → 204
curl -i -X DELETE -H "Authorization: Bearer $TOKEN" localhost:3001/api/proposals/PR-004
```

- [ ] ✅ ไม่มี token = 401 · มี token staff = 204
- [ ] ✅ ลองใช้ token ของ role `student` แล้ว PUT/DELETE → 403

### M2.B6 · เพิ่ม `upsertStaff` ใน `server.js` `BE`

ใน `api/src/server.js` ต่อจาก `await loadSeed();`:

```js
import { loadSeed, upsertStaff } from './services/proposalService.js';
import { hashPassword } from './utils/password.js';

if (process.env.STAFF_EMAIL && process.env.STAFF_PASSWORD) {
  upsertStaff({
    email: process.env.STAFF_EMAIL,
    name: 'เจ้าหน้าที่หลักสูตร',
    passwordHash: hashPassword(process.env.STAFF_PASSWORD)
  });
}
```

- [ ] ✅ ทดสอบ local: `STAFF_EMAIL=staff@rmutl.ac.th STAFF_PASSWORD=newpass123456 npm run dev --prefix api` → login ด้วยรหัสใหม่ได้

### M2.B7 · unit tests ≥ 6 `BE`

สร้าง `api/tests/unit/proposalValidator.test.js` ที่ทดสอบค่าขอบ (ดูโค้ดเต็ม 12 ข้อใน `BUILD.md` §M2.5-T1):

- `teamSize` = 0 (ไม่ผ่าน), 4 (ผ่าน), 5 (ไม่ผ่าน) ← **ค่าขอบของกฎเฉพาะ**
- `title` ยาว 9 ตัวอักษร (ไม่ผ่าน), 10 ตัวอักษร (ผ่าน) ← **ค่าขอบของกฎเฉพาะ**
- เลือกอาจารย์ที่ `availableQuota = 0` (ไม่ผ่าน) ← **ค่าขอบของกฎเฉพาะ**
- `scope` สั้นกว่า 15 ตัวอักษร (ไม่ผ่าน)

- [ ] ✅ `npm test --prefix api -- unit` ผ่าน ≥ 6 ข้อ

### M2.B8 · integration tests ≥ 6 `BE`

สร้าง `api/tests/integration/proposals.test.js` (ดูโค้ดเต็ม 13 ข้อใน `BUILD.md` §M2.5-T2):

- GET `/api/proposals` → 200 + array + มี `advisorName`
- GET `/api/proposals/PR-999` → 404
- POST `/api/proposals` ด้วย body ถูก → 201
- POST ด้วย body ผิด / สมาชิกเกิน 4 คน → 400 + details
- PUT / DELETE โดยไม่มี token → **401**
- PUT ด้วย token student → **403**
- PUT / DELETE ด้วย token staff → **200 / 204**

- [ ] ✅ `npm test --prefix api -- integration` ผ่าน ≥ 6 ข้อ

### M2.B9 · ตั้ง Render service `BE/DevOps`

1. [ ] ไปที่ Render Dashboard → New → Blueprint → ชี้ที่ repo `engse203-final-2-T-05`
2. [ ] ตั้ง Environment variables:
   - `JWT_SECRET` → `render.yaml` ตั้ง `generateValue: true` อยู่แล้ว · ไม่ต้องกรอก
   - `STAFF_EMAIL` → อีเมลเดียวกับใน seed (`staff@rmutl.ac.th`)
   - `STAFF_PASSWORD` → รหัสยาว ≥ 12 ตัว · **เก็บไว้แจ้งอาจารย์**
   - `NODE_ENV=production`
3. [ ] ลบบรรทัด `rootDir` จาก `render.yaml` ถ้ายังมี
4. [ ] Deploy

- [ ] ✅ ผ่านเมื่อ: ได้ URL (`https://xxx.onrender.com`) · `/api/health` ตอบ 200

### M2.B10 · ทดสอบ production login `BE/DevOps`

```bash
# ต้อง login ด้วย STAFF_PASSWORD ของ production (ไม่ใช่ staff1234)
curl -X POST https://<url>[.onrender.com/api/auth/login](https://.onrender.com/api/auth/login) \
  -H "Content-Type: application/json" \
  -d '{"email":"staff@rmutl.ac.th","password":"<PROD_PASSWORD>"}'

# ลอง login ด้วย staff1234 → ต้อง 401 (ไม่ผ่าน)
curl -X POST https://<url>[.onrender.com/api/auth/login](https://.onrender.com/api/auth/login) \
  -H "Content-Type: application/json" \
  -d '{"email":"staff@rmutl.ac.th","password":"staff1234"}'
```

- [ ] ✅ รหัส production ผ่าน · รหัส seed ไม่ผ่าน

## ส่วนของ FE (ทำขนาน)

### M2.F1 · `apiClient` + `proposalService` + `useAuth` `FE`

- `frontend/src/services/apiClient.js` — คงจาก LAB 13 (แนบ `Authorization: Bearer <token>` ที่เดียว)
- `frontend/src/services/proposalService.js` — ฟังก์ชันเรียก `/api/proposals`, `/api/advisors`, `/api/auth/login` (ดู `BUILD.md` §M2-F2)
- `frontend/src/hooks/useAuth.js` — จัดการสถานะ login ของเจ้าหน้าที่หลักสูตร (ดู `BUILD.md` §M2-F4)

- [ ] ✅ `apiClient` เป็น central point เดียว

### M2.F2 · หน้า Login `FE`

- `/login` form email + password (`LoginPage.jsx`)
- call `POST /api/auth/login` ผ่าน `proposalService`
- สำเร็จ → เก็บ token → redirect `/`
- ล้มเหลว → แสดงข้อความ error ตามที่ API ส่งมา
- ปุ่ม Logout ที่ nav → ล้าง token + redirect `/login`

- [ ] ✅ login + logout ใช้ได้ใน local

### M2.F3 · หน้า Proposals List `FE`

- `/` (`ProposalsListPage.jsx`) fetch `GET /api/proposals` → แสดงรายการโครงงาน
- แสดง `advisorName` และ `studentName` จาก JOIN
- Dropdown filter สถานะ (`submitted`, `reviewing`, `approved`, `rejected`) → เพิ่ม `?status=` ใน query
- loading / error / empty state

- [ ] ✅ กรองสถานะได้ · ลิงก์ไป `/proposals/:id` ได้

### M2.F4 · หน้า Proposal Detail `FE`

- `/proposals/:id` (`ProposalDetailPage.jsx`) fetch `GET /api/proposals/:id`
- 404 → แสดงข้อความไม่พบข้อเสนอโครงงาน
- ปุ่ม "เปลี่ยนสถานะ" + "ลบข้อเสนอโครงงาน" **แสดงเฉพาะเมื่อ `isStaff === true`**
- กดปุ่มเปลี่ยนสถานะ → เรียก `PUT /api/proposals/:id`

- [ ] ✅ นักศึกษา/คนทั่วไปเปิดดูได้ แต่ไม่เห็นปุ่มจัดการของเจ้าหน้าที่

### M2.F5 · หน้า New Proposal `FE`

- `/new` (`NewProposalPage.jsx`) form กรอก `studentName` · `title` · `category` · เลือกอาจารย์จาก `GET /api/advisors` (แสดง `availableQuota/maxQuota` และ disable ถ้าโควตา = 0) · `teamSize` (1–4) · `scope`
- POST สำเร็จ → redirect `/proposals/:newId`
- POST 400 → แสดงรายการ `errors` จาก API

- [ ] ✅ ยื่นเสนอหัวข้อได้ · เห็น error ชัดเจนเมื่อกรอกผิดกฎ

### M2.F6 · ตั้งค่า Vite สำหรับ production `FE`

แก้ `frontend/vite.config.js`:
```js
export default defineConfig({
  plugins: [react()],
  base: '/',                        // ไม่ใช่ /engse203-student-labs...
  build: { outDir: 'dist', emptyOutDir: true },
});
```

- [ ] ✅ `npm run build --prefix frontend` ได้ `dist/` + `index.html`

## รวมท้าย M2

### M2.X1 · ทดสอบ end-to-end ทั้งคู่ `BOTH`

- [ ] local: FE ยิง API local ได้ · login · ยื่นเสนอหัวข้อโครงงาน · เปลี่ยนสถานะ · ลบ
- [ ] production: เปิด URL จริงบน Render · ทำเหมือนกัน

### M2.X2 · Commit + tag `v0.5.0` 🏷 `BE`

```bash
git add -A
git commit -m "M2: proposals API + validator + login + first deploy"
git tag v0.5.0
git push && git push origin v0.5.0
```

- [ ] ✅ ผ่านเมื่อ: tag `v0.5.0` อยู่บน GitHub

---

# M2.5 · เสริมคุณภาพ (17–20 ต.ค.)

> ไม่มี tag · แต่เป็นงานหนักของคะแนน A5

## M2.5.1 · unit tests ครบ 10+ `BE`

ใส่โค้ด `api/tests/unit/proposalValidator.test.js` จาก `BUILD.md` §M2.5-T1 (มีครบ 12 ข้อ รวมค่าขอบของกฎ `teamSize` 1–4 คน, `title` 10–200 ตัวอักษร และ `availableQuota`)

- [ ] ✅ `npm test --prefix api -- unit` รายงาน ≥ 10 ข้อ

## M2.5.2 · integration tests ครบ 12+ `BE`

ใส่โค้ด `api/tests/integration/proposals.test.js` จาก `BUILD.md` §M2.5-T2 (มีครบ 13 ข้อ รวม 401 · 403 · เจ้าหน้าที่ทำได้สำเร็จ)

- [ ] ✅ `npm test --prefix api -- integration` รายงาน ≥ 12 ข้อ

## M2.5.3 · frontend tests ≥ 3 `FE`

สร้าง `frontend/src/utils/proposalSummary.test.js` ทดสอบ pure function `summarizeByStatus(proposals)`:
- อาร์เรย์ว่าง → คืน `{ submitted: 0, reviewing: 0, approved: 0, rejected: 0 }`
- นับจำนวนแต่ละสถานะได้ถูกต้อง
- ข้ามสถานะที่ไม่รู้จักโดยไม่พัง

- [ ] ✅ `npm test --prefix frontend` รายงาน ≥ 3 ข้อ

## M2.5.4 · Dashboard page `FE`

- `/dashboard` (`DashboardPage.jsx`) fetch `GET /api/proposals` · นับแยกสถานะด้วย `summarizeByStatus` · แสดงเป็น card 4 ใบ

- [ ] ✅ ตัวเลขตรงกับใน `/` ที่กรองแต่ละสถานะ

## M2.5.5 · `DEBUG_LOG.md` ≥ 2 bug จริง `BE`

เขียน bug ที่เจอจริงระหว่างทำโปรเจกต์ (ดูเนื้อหาพร้อมใช้ใน `BUILD.md` §M2.5-T3):
- Bug #1: โควตาอาจารย์ที่ปรึกษา (`availableQuota`) ไม่คืนค่าเมื่อโครงงานถูกเปลี่ยนสถานะเป็น `rejected`
- Bug #2: ข้อมูลหายทุกครั้งที่ restart server เพราะ `loadSeed()` เช็กชื่อตาราง `requests` แทนที่จะเป็น `proposals`

- [ ] ✅ ครบ 2 bug · มี regression test ใน `tests/`

## M2.5.6 · CI badge ใน README `BE/DevOps`

เพิ่มบรรทัดใน README (ใต้ชื่อระบบ):

```md
![CI](https://github.com/<ชื่อ-org-หรือ-user>/engse203-final-2-T-05/actions/workflows/check.yml/badge.svg)
```

- [ ] ✅ เห็นป้ายสีเขียวบน GitHub

## M2.5.7 · ⭐ เลือก 1–2 ข้อ `BOTH`

แนะนำ: **ค้นหา** (`?q=` รองรับไว้ใน `proposalService.js` แล้ว แค่เพิ่มช่องค้นหาในหน้า `ProposalsListPage.jsx`) · **จำกัดการเดารหัส** (429)

- [ ] ✅ ทำ + มี test + สาธิตได้ตอนสัมภาษณ์

---

# M3 · ปิดงาน → `v1.0.0`

> **กำหนด: 22 ต.ค. ก่อนเวลาสอบ**

## M3.1 · กรอก README ครบ `BE/DevOps`

เปิด `README.md` · กรอกทุกส่วนตามแม่แบบ:
- สมาชิก (ตารางครบ)
- ความสามารถของระบบ (ตารางนักศึกษาทั่วไป / เจ้าหน้าที่หลักสูตร)
- ภาพหน้าจอ desktop + มือถือ (upload เป็น png ใน `/docs/` หรือ GitHub attachments)
- ข้อมูล (schema diagram หรือ list ตาราง `users`, `advisors`, `proposals` + ความสัมพันธ์)
- วิธีติดตั้ง · รัน · test · deploy
- URL + บัญชีทดสอบ (dev เท่านั้น · production ไม่เขียนใน repo)
- **การใช้ AI** — บอกตรงๆ ว่าใช้ AI ช่วยปรับโครงสร้างเอกสารและ review schema/validator · ยืนยันว่าอธิบายโค้ดทุกบรรทัดได้

- [ ] ✅ ครบทุกหัวข้อ · ไม่มี `<...>` ค้าง

## M3.2 · `API_CONTRACT.md` ตรงกับโค้ด `BOTH`

เปิดเทียบทีละ endpoint:
- method · path · request body · response body · status code · error messages
- ตรงกับที่ API จริงคืน (ยิง curl เทียบ)

- [ ] ✅ ไม่มี field ที่ contract บอกแต่ API ไม่คืน (หรือกลับกัน)

## M3.3 · ติ๊ก `RELEASE_CHECKLIST.md` ครบ `BE/DevOps`

เปิดทุกข้อ · ติ๊กพร้อม**หลักฐาน** (จำนวน test · ลิงก์ CI · URL · ภาพ)

ข้อที่ทำไม่ได้ → ติ๊กแล้วเขียน `ยกเว้น: <เหตุผล>` (เช่น Turso สำหรับทีมคู่)

- [ ] ✅ ทุกข้อติ๊กหมด

## M3.4 · ทดสอบด้วย `check-project.mjs` `BE`

```bash
node --disable-warning=ExperimentalWarning check-project.mjs
node --disable-warning=ExperimentalWarning check-project.mjs --online
```

- [ ] ✅ ไม่มี FAIL แดง (ข้ามข้อ `--skip-build` ได้)
- [ ] ถ้าเจอ FAIL → แก้แล้วรันซ้ำจนผ่าน

## M3.5 · ตรวจเรื่อง security รอบสุดท้าย `BE/DevOps`

```bash
# ไม่มี secret
git ls-files | grep -E '\.env$' | grep -v example
# ควรว่าง (ไม่มี .env จริง)

# ไม่มี node_modules
git ls-files | grep node_modules
# ควรว่าง

# ไม่มี .db
git ls-files | grep '\.db$'
# ควรว่าง

# audit
npm audit --prefix api
npm audit --prefix frontend
```

- [ ] ✅ ผ่านทั้ง 4 เช็ค · ไม่มี high/critical ค้างใน audit

## M3.6 · แจ้งรหัส production ให้อาจารย์ `BE/DevOps`

- [ ] ส่ง `STAFF_EMAIL` + `STAFF_PASSWORD` ของ Render ให้อาจารย์ตามช่องทางที่ประกาศใน LMS (ไม่ใช่ใน repo · ไม่ใช่ใน PR)

## M3.7 · Tag `v1.0.0` 🏷 `BE`

```bash
# ต้องแน่ใจว่าทุกอย่าง commit แล้ว
git status                       # clean
git log --oneline -5             # commit ล่าสุดคือเวอร์ชันที่จะส่ง

git tag v1.0.0
git push origin main
git push origin v1.0.0
```

- [ ] ✅ tag `v1.0.0` อยู่บน GitHub
- [ ] ✅ URL ที่ deploy อยู่ที่ commit ของ tag นี้ (ไม่มี commit ใหม่หลัง tag)

## M3.8 · ส่งลิงก์ใน LMS `BOTH`

- [ ] ส่งลิงก์ repo + URL deploy ใน LMS
- [ ] ส่งก่อนเวลาสอบของ Section 2 (22 ต.ค.)
- [ ] **หลังจากนี้ห้าม commit — commit หลัง `v1.0.0` ไม่คิดคะแนน**

## M3.PR · ส่ง Peer Review `BOTH`

- [ ] **17 ต.ค.** ครั้งที่ 1 (ไม่คิดคะแนน · เตือนกันเอง) — ส่งผ่านฟอร์มที่อาจารย์ประกาศใน LMS
- [ ] **22 ต.ค. หลังสัมภาษณ์** ครั้งที่ 2 (คิดคะแนน I3) — ตอบ 4 ข้อ + หลักฐาน 1 ตัวอย่างต่อเพื่อน + ประเมินตัวเอง 3 ข้อ
- ⚠ ส่งเป็นรายบุคคล · **ไม่อยู่ใน repo** · เพื่อนไม่เห็นคะแนนที่คุณให้

## M3.9 · เตรียม demo 3 นาที `BOTH`

เตรียมเปิด:
1. URL deploy
2. ยื่นเสนอหัวข้อโครงงานใหม่เป็นนักศึกษาทั่วไป (ไม่ login)
3. Login เจ้าหน้าที่หลักสูตร
4. เปลี่ยนสถานะข้อเสนอโครงงาน `submitted` → `reviewing` หรือ `approved`
5. แสดงให้เห็นว่านักศึกษาทั่วไปกด DELETE ไม่ได้ (401/403 ใน DevTools)
6. เปิด GitHub Actions tab แสดง CI ผ่าน

ซ้อม 1–2 รอบก่อนวันสอบ

- [ ] ✅ ซ้อมแล้วทำได้ใน ≤ 3 นาที

---

# สรุปจำนวน milestones

| ขั้น | จำนวน sub-steps |
|---|---:|
| M1 | 19 |
| M2 (BE) | 10 |
| M2 (FE) | 6 |
| M2.X | 2 |
| M2.5 | 7 |
| M3 | 9 |
| **รวม** | **53** |

ทำได้ 53 ข้อนี้ครบ = ได้คะแนนเต็มส่วนผลงาน + คะแนนรายบุคคลสูงด้วย

> 💡 เปิดไฟล์นี้ใน VS Code แล้วติ๊กไปทีละข้อ · commit ไฟล์นี้บ่อยๆ จะเห็นความก้าวหน้าใน git log