# BUILD — คู่มือลงมือจริง · T05 เสนอหัวข้อโครงงาน

> โค้ดในไฟล์นี้พร้อม copy-paste · pattern ตรงกับ LAB 13
> ใช้คู่กับ [STEPS.md](STEPS.md) (checklist) และ [PLAN.md](PLAN.md) (ภาพรวม)
>
> ⚠ **ก่อน copy-paste** อ่านคอมเมนต์ของทุกไฟล์ให้เข้าใจ · ตอนสัมภาษณ์ **ต้องอธิบายทุกบรรทัด** — ไม่งั้นได้ 0

---

# M1 · คัดลอก + rename → `v0.1.0`

## M1-C1 · คำสั่งคัดลอก

```bash
cd ~/engse203-final-2-T-05

# ① เอาโค้ด LAB 13 มา (ไม่เอา node_modules · *.db · .env)
rsync -a --exclude node_modules --exclude '*.db' --exclude .env \
  ~/class/port/labs/week-13/source/ ./

# ② วาง kit (ไม่ทับ TEAM.md)
cp -rn ~/class/engse203-lab/labs/final-term-project/kit/. ./

# ⚠ -n ไม่ทับ .github/workflows/check.yml ของ LAB 13 (ชื่อ "Week 12 Tests")
#   บังคับใช้ของ kit แทน (ชื่อ "CI")
cp -f ~/class/engse203-lab/labs/final-term-project/kit/.github/workflows/check.yml \
      .github/workflows/check.yml

# ③ ลบไฟล์ LAB เดิมที่ไม่ใช้แล้ว
rm -f BUG_REPORTS.md DEMO.md DATABASE_CHOICES.md TEST_CASES.md check-week1*.mjs
rm -rf api/scripts/check-project.mjs api/data/initialRequests.json \
       frontend/public/data frontend/src/services/requestStorage.js

# ④ ไฟล์ .env ของเครื่องตัวเอง
cp api/.env.example api/.env
echo "JWT_SECRET=$(openssl rand -hex 32)" >> api/.env

# ⑤ แทน README
cp PROJECT_README_TEMPLATE.md README.md

# ⑥ กรอก config
cp project.config.example.json project.config.json
```

## M1-C2 · แก้ `package.json` ที่ root

หา key `"check"` แล้วเปลี่ยนเป็น:

```json
"check": "node --disable-warning=ExperimentalWarning check-project.mjs"
```

## M1-C3 · `api/data/schema.sql` ใหม่ทั้งไฟล์

ลบของเดิมทิ้ง แล้วแทนด้วยไฟล์นี้:

```sql
-- ═══════════════════════════════════════════════════════════
-- T05 เสนอหัวข้อโครงงาน — โครงสร้างฐานข้อมูล
-- ENGSE203 Final Term Project · Section 2 · ทีม T-05
-- ═══════════════════════════════════════════════════════════

PRAGMA foreign_keys = ON;

DROP TABLE IF EXISTS proposals;
DROP TABLE IF EXISTS advisors;
DROP TABLE IF EXISTS users;

CREATE TABLE users (
  id            INTEGER PRIMARY KEY AUTOINCREMENT,
  name          TEXT NOT NULL,
  department    TEXT NOT NULL,
  email         TEXT NOT NULL UNIQUE,
  role          TEXT NOT NULL DEFAULT 'student'
                CHECK (role IN ('student', 'staff')),
  password_hash TEXT
);

CREATE TABLE advisors (
  id         TEXT PRIMARY KEY,                     -- ADV-001
  name       TEXT NOT NULL,
  max_quota  INTEGER NOT NULL CHECK (max_quota > 0),
  expertise  TEXT NOT NULL,
  department TEXT
);

CREATE TABLE proposals (
  id            TEXT PRIMARY KEY,                  -- PR-001
  student_id    INTEGER NOT NULL,
  advisor_id    TEXT NOT NULL,
  title         TEXT NOT NULL,
  category      TEXT NOT NULL,
  team_size     INTEGER NOT NULL CHECK (team_size BETWEEN 1 AND 4),
  scope         TEXT NOT NULL,
  status        TEXT NOT NULL DEFAULT 'submitted'
                CHECK (status IN ('submitted','reviewing','approved','rejected')),
  created_at    TEXT NOT NULL DEFAULT (datetime('now','localtime')),

  FOREIGN KEY (student_id) REFERENCES users(id),
  FOREIGN KEY (advisor_id) REFERENCES advisors(id)
);

INSERT INTO users (name, department, email) VALUES
  ('สมชาย ใจดี',       'วิศวกรรมซอฟต์แวร์', 'somchai@rmutl.ac.th'),
  ('สุภาวดี รักเรียน', 'วิศวกรรมซอฟต์แวร์', 'supawadee@rmutl.ac.th'),
  ('ธนกฤต ตั้งใจ',     'วิศวกรรมคอมพิวเตอร์', 'thanakrit@rmutl.ac.th'),
  ('ปรียา ขยันยิ่ง',   'วิศวกรรมซอฟต์แวร์', 'preeya@rmutl.ac.th');

-- บัญชี staff dev — รหัส staff1234 · production ตั้งใหม่ด้วย STAFF_EMAIL + STAFF_PASSWORD env
INSERT INTO users (name, department, email, role, password_hash) VALUES
  ('เจ้าหน้าที่หลักสูตร', 'สาขาวิศวกรรมซอฟต์แวร์', 'staff@rmutl.ac.th', 'staff',
   'scrypt$5e1f0c3a9b7d2e4f6a8c0b1d3e5f7a9c$2f81bcd58254b475f16ddaf1e278338fa0990ed132f8b206d0a88cb4108b6b4f90df05c0d97166e2a108ab36bf3d81f7884165fff13d4ec42a67cfa36771bc38');

INSERT INTO advisors (id, name, max_quota, expertise, department) VALUES
  ('ADV-001', 'ผศ.ดร. กิตติพงศ์ พัฒนาเวช', 3, 'Web Application & Cloud',      'วิศวกรรมซอฟต์แวร์'),
  ('ADV-002', 'อ.ดร. นภาลัย ปัญญาประดิษฐ์', 4, 'AI & Machine Learning',        'วิศวกรรมซอฟต์แวร์'),
  ('ADV-003', 'ผศ. ชัยวัฒน์ เน็ตเวิร์ค',   2, 'IoT & Embedded Systems',       'วิศวกรรมคอมพิวเตอร์'),
  ('ADV-004', 'อ. ศิริพร ข้อมูลดี',        3, 'Data Science & Analytics',     'วิศวกรรมซอฟต์แวร์'),
  ('ADV-005', 'อ. วีระยุทธ มือถือไว',      3, 'Mobile Application & UX/UI',   'วิศวกรรมซอฟต์แวร์');

INSERT INTO proposals (id, student_id, advisor_id, title, category, team_size, scope, status) VALUES
  ('PR-001', 1, 'ADV-001', 'ระบบนัดหมายอาจารย์ที่ปรึกษาออนไลน์',      'Web App',      2, 'พัฒนาระบบจองเวลาเข้าพบอาจารย์พร้อมแจ้งเตือนผ่านอีเมล', 'approved'),
  ('PR-002', 2, 'ADV-002', 'ระบบวิเคราะห์ความหวานเครื่องดื่มด้วย AI', 'AI / Data',    3, 'สร้างโมเดลทำนายระดับน้ำตาลจากข้อมูลโภชนาการและภาพถ่าย', 'submitted'),
  ('PR-003', 3, 'ADV-003', 'อุปกรณ์ตรวจวัดฝุ่น PM2.5 พลังงานแสงอาทิตย์', 'IoT',         2, 'พัฒนาบอร์ดไมโครคอนโทรลเลอร์ส่งค่าฝุ่นขึ้นแดชบอร์ดเรียลไทม์', 'reviewing'),
  ('PR-004', 4, 'ADV-001', 'เว็บแอปพลิเคชันจัดการคลังสินค้าขนาดเล็ก',  'Web App',      1, 'ระบบตัดสต็อกและออกรายงานสรุปยอดขายรายเดือน',            'rejected'),
  ('PR-005', 1, 'ADV-005', 'แอปพลิเคชันติดตามระยะทางวิ่งและสุขภาพ',    'Mobile App',   2, 'แอปมือถือบันทึกสถิติการวิ่งพร้อมคำนวณแคลอรีรายสัปดาห์',  'submitted');

CREATE INDEX IF NOT EXISTS idx_proposals_status  ON proposals(status);
CREATE INDEX IF NOT EXISTS idx_proposals_student ON proposals(student_id);
CREATE INDEX IF NOT EXISTS idx_proposals_advisor ON proposals(advisor_id);
```

## M1-C4 · Rename ไฟล์

```bash
cd ~/engse203-final-2-T-05/api/src

git mv services/requestService.js       services/proposalService.js
git mv controllers/requestController.js controllers/proposalController.js
git mv routes/requestRoutes.js          routes/proposalRoutes.js
git mv validators/requestValidator.js   validators/proposalValidator.js
```

## M1-C5 · `api/src/config.js`

เปลี่ยนบรรทัด `dbFile`:

```js
dbFile: process.env.DB_FILE ?? path.join(API_ROOT, 'data', 'proposals.db'),
```

และใน `api/.env.example` + `api/.env`:

```
DB_FILE=./data/proposals.db
```

## M1-C6 · `api/src/services/proposalService.js` — เขียนใหม่ทั้งไฟล์

```js
import { DatabaseSync } from 'node:sqlite';
import { readFileSync, existsSync } from 'node:fs';
import path from 'node:path';
import { fileURLToPath } from 'node:url';

const HERE = path.dirname(fileURLToPath(import.meta.url));
const API_ROOT = path.resolve(HERE, '../..');
const DB_FILE = process.env.DB_FILE ?? path.join(API_ROOT, 'data', 'proposals.db');
const SCHEMA_FILE = path.join(API_ROOT, 'data', 'schema.sql');

let db;
let driver = 'sqlite';

async function openDatabase() {
  const url = process.env.TURSO_DATABASE_URL;
  if (url) {
    const { default: Database } = await import('libsql');
    driver = 'turso';
    return new Database(url, { authToken: process.env.TURSO_AUTH_TOKEN });
  }
  driver = 'sqlite';
  return new DatabaseSync(DB_FILE);
}

/**
 * SELECT หลัก — JOIN advisors และ users
 * frontend ได้ advisorName · studentName สำเร็จ · ไม่ต้องยิงหลายรอบ
 */
const SELECT_SHAPE = `
  SELECT p.id,
         u.name           AS studentName,
         p.advisor_id     AS advisorId,
         a.name           AS advisorName,
         a.expertise      AS advisorExpertise,
         p.title,
         p.category,
         p.team_size      AS teamSize,
         p.scope,
         p.status,
         p.created_at     AS createdAt
  FROM proposals p
  LEFT JOIN users    u ON u.id = p.student_id
  LEFT JOIN advisors a ON a.id = p.advisor_id`;

export async function loadSeed() {
  db = await openDatabase();
  db.exec('PRAGMA foreign_keys = ON');
  // ⚠ ชื่อตารางต้องเป็น 'proposals' ไม่ใช่ 'requests'
  //    ไม่งั้น schema (ที่มี DROP) ถูกรันทุก start → ข้อมูลหายทุก restart
  const ready = db.prepare(
    "SELECT COUNT(*) c FROM sqlite_master WHERE type='table' AND name='proposals'"
  ).get().c;
  if (!ready && existsSync(SCHEMA_FILE)) {
    db.exec(readFileSync(SCHEMA_FILE, 'utf8'));
  }
}

export function getDbStatus() {
  try {
    if (!db) return { connected: false, reason: 'ยังไม่ได้เปิดฐานข้อมูล' };
    const n = db.prepare("SELECT COUNT(*) c FROM sqlite_master WHERE type='table'").get().c;
    return { connected: true, driver, tables: n };
  } catch (e) {
    return { connected: false, reason: e.message };
  }
}

export function findAll({ status, q } = {}) {
  const where = [];
  const params = [];
  if (status) { where.push('p.status = ?'); params.push(status); }
  if (q)      { where.push('(p.title LIKE ? OR p.scope LIKE ? OR a.name LIKE ? OR u.name LIKE ?)');
                const like = `%${q}%`; params.push(like, like, like, like); }
  const whereSql = where.length ? ` WHERE ${where.join(' AND ')}` : '';
  return db.prepare(`${SELECT_SHAPE}${whereSql} ORDER BY p.id`).all(...params);
}

export function findById(id) {
  return db.prepare(`${SELECT_SHAPE} WHERE p.id = ?`).get(id) ?? null;
}

/** โควตาคงเหลือของอาจารย์ที่ปรึกษา · ใช้โดย advisorService และ validator */
export function availableQuotaFor(advisorId, excludeProposalId = null) {
  const adv = db.prepare('SELECT max_quota FROM advisors WHERE id = ?').get(advisorId);
  if (!adv) return 0;
  const row = db.prepare(
    `SELECT COUNT(*) AS used
     FROM proposals
     WHERE advisor_id = ?
       AND status IN ('submitted','reviewing','approved')
       AND (? IS NULL OR id <> ?)`
  ).get(advisorId, excludeProposalId, excludeProposalId);
  return adv.max_quota - row.used;
}

/** สร้างรหัสถัดไป เช่น PR-006 */
function nextId() {
  const row = db.prepare(
    "SELECT id FROM proposals WHERE id LIKE 'PR-%' ORDER BY id DESC LIMIT 1"
  ).get();
  const n = row ? Number(String(row.id).replace('PR-', '')) + 1 : 1;
  return `PR-${String(n).padStart(3, '0')}`;
}

/** หา id ของนักศึกษาจากชื่อ · ไม่มีให้สร้างใหม่ (role=student) */
function resolveStudentId(name) {
  const found = db.prepare('SELECT id FROM users WHERE name = ?').get(name);
  if (found) return found.id;
  const slug = Date.now().toString(36);
  return db.prepare(
    "INSERT INTO users (name, department, email) VALUES (?, 'วิศวกรรมซอฟต์แวร์', ?)"
  ).run(name, `student-${slug}@rmutl.ac.th`).lastInsertRowid;
}

export function create(input) {
  const id = nextId();
  db.exec('BEGIN');
  try {
    const studentId = resolveStudentId(input.studentName.trim());
    db.prepare(
      `INSERT INTO proposals
         (id, student_id, advisor_id, title, category, team_size, scope)
       VALUES (?, ?, ?, ?, ?, ?, ?)`
    ).run(id, studentId, input.advisorId, input.title.trim(),
          input.category.trim(), Number(input.teamSize), input.scope.trim());
    db.exec('COMMIT');
  } catch (err) {
    db.exec('ROLLBACK');
    throw err;
  }
  return findById(id);
}

export function updateStatus(id, status) {
  const result = db.prepare('UPDATE proposals SET status = ? WHERE id = ?').run(status, id);
  return result.changes ? findById(id) : null;
}

export function remove(id) {
  const target = findById(id);
  if (!target) return null;
  db.prepare('DELETE FROM proposals WHERE id = ?').run(id);
  return target;
}

// ─────────────── User / Auth helpers (คงจาก LAB 13) ───────────────

export function listUsers() {
  return db.prepare('SELECT id, name, department FROM users ORDER BY id').all();
}

export function findUserByEmail(email) {
  return db.prepare(
    'SELECT id, name, email, role, password_hash AS passwordHash FROM users WHERE email = ?'
  ).get(String(email).trim().toLowerCase()) ?? null;
}

export function upsertStaff({ email, name, passwordHash }) {
  const normalized = String(email).trim().toLowerCase();
  if (findUserByEmail(normalized)) {
    db.prepare("UPDATE users SET role = 'staff', password_hash = ? WHERE email = ?")
      .run(passwordHash, normalized);
    return 'updated';
  }
  db.prepare(
    "INSERT INTO users (name, department, email, role, password_hash) VALUES (?, 'เจ้าหน้าที่หลักสูตร', ?, 'staff', ?)"
  ).run(name, normalized, passwordHash);
  return 'created';
}

// สำหรับ advisorService ใช้ (export db instance)
export function _getDb() { return db; }
```

## M1-C7 · `api/src/services/advisorService.js` — ไฟล์ใหม่

```js
import { _getDb, availableQuotaFor } from './proposalService.js';

/** รายชื่ออาจารย์ที่ปรึกษาทั้งหมด · มี availableQuota สำเร็จ (frontend ใช้ตอน dropdown) */
export function listAdvisors() {
  const db = _getDb();
  const rows = db.prepare(
    'SELECT id, name, max_quota AS maxQuota, expertise, department FROM advisors ORDER BY id'
  ).all();
  return rows.map(r => ({ ...r, availableQuota: availableQuotaFor(r.id) }));
}

export function findAdvisor(id) {
  const db = _getDb();
  const row = db.prepare(
    'SELECT id, name, max_quota AS maxQuota, expertise, department FROM advisors WHERE id = ?'
  ).get(id);
  if (!row) return null;
  return { ...row, availableQuota: availableQuotaFor(row.id) };
}
```

## M1-C8 · `api/src/validators/proposalValidator.js` — เขียนใหม่

```js
/**
 * proposalValidator.js — pure functions (test ได้ง่าย)
 *
 * กฎเฉพาะของ T05
 *   ① จำนวนสมาชิกในกลุ่ม (teamSize) ต้องเป็นจำนวนเต็ม 1–4 คน และชื่อโครงงาน (title) ยาว 10–200 ตัวอักษร
 *   ② อาจารย์ที่ปรึกษาที่เลือกต้องมีโควตาคงเหลือ (availableQuota > 0)
 */

export const STATUSES = ['submitted', 'reviewing', 'approved', 'rejected'];

export const MIN_NAME = 2;
export const MAX_NAME = 100;
export const MIN_TITLE = 10;
export const MAX_TITLE = 200;
export const MIN_SCOPE = 15;
export const MAX_SCOPE = 1000;
export const MIN_TEAM = 1;
export const MAX_TEAM = 4;

function readText(v) {
  return typeof v === 'string' ? v.trim() : '';
}

/**
 * ตรวจ body ของ POST /api/proposals
 *   @param input    — body
 *   @param context  — { advisors } ส่งเข้ามาเพื่อตรวจ availableQuota
 *                     pure function จึงรับ context เป็น parameter · ไม่ไปหยิบ db เอง
 */
export function validateProposalInput(input, context = {}) {
  if (!input || typeof input !== 'object' || Array.isArray(input)) {
    return ['ต้องส่งข้อมูลข้อเสนอโครงงาน'];
  }

  const errors = [];
  const name = readText(input.studentName);
  if (name.length < MIN_NAME || name.length > MAX_NAME) {
    errors.push(`ชื่อผู้เสนอต้องยาว ${MIN_NAME}–${MAX_NAME} ตัวอักษร`);
  }

  const title = readText(input.title);
  if (title.length < MIN_TITLE || title.length > MAX_TITLE) {
    errors.push(`ชื่อโครงงานต้องยาว ${MIN_TITLE}–${MAX_TITLE} ตัวอักษร`);
  }

  const category = readText(input.category);
  if (!category) {
    errors.push('ต้องระบุหมวดหมู่โครงงาน');
  }

  if (typeof input.advisorId !== 'string' || !input.advisorId) {
    errors.push('ต้องเลือกอาจารย์ที่ปรึกษา');
  } else if (context.advisors) {
    const adv = context.advisors.find(a => a.id === input.advisorId);
    if (!adv) {
      errors.push(`ไม่พบอาจารย์รหัส ${input.advisorId}`);
    } else if (adv.availableQuota <= 0) {
      errors.push(`อาจารย์ที่ปรึกษาโควตาเต็มแล้ว (เหลือ ${adv.availableQuota} กลุ่ม)`);
    }
  }

  const teamSize = Number(input.teamSize);
  if (!Number.isInteger(teamSize) || teamSize < MIN_TEAM || teamSize > MAX_TEAM) {
    errors.push(`จำนวนสมาชิกในกลุ่มต้องเป็นจำนวนเต็ม ${MIN_TEAM}–${MAX_TEAM} คน`);
  }

  const scope = readText(input.scope);
  if (scope.length < MIN_SCOPE || scope.length > MAX_SCOPE) {
    errors.push(`ขอบเขตโครงงานต้องยาว ${MIN_SCOPE}–${MAX_SCOPE} ตัวอักษร`);
  }

  return errors;
}

export function isValidStatus(status) {
  return STATUSES.includes(status);
}

/** คงจาก LAB 13 — ใช้โดย authRoutes */
export function validateLoginInput(input) {
  if (!input || typeof input !== 'object') return ['ต้องส่งอีเมลและรหัสผ่าน'];
  const errors = [];
  if (typeof input.email !== 'string' || !input.email.includes('@') || input.email.length > 254) {
    errors.push('รูปแบบอีเมลไม่ถูกต้อง');
  }
  if (typeof input.password !== 'string' || input.password.length === 0 || input.password.length > 200) {
    errors.push('กรุณาระบุรหัสผ่าน');
  }
  return errors;
}
```

## M1-C9 · `api/src/controllers/proposalController.js`

```js
import * as service from '../services/proposalService.js';
import * as advisorService from '../services/advisorService.js';
import { validateProposalInput, isValidStatus } from '../validators/proposalValidator.js';

export function listProposals(req, res) {
  const { status, q } = req.query;
  res.status(200).json(service.findAll({ status, q }));
}

export function getProposal(req, res) {
  const found = service.findById(req.params.id);
  if (!found) {
    return res.status(404).json({ error: `ไม่พบข้อเสนอโครงงานรหัส ${req.params.id}` });
  }
  res.status(200).json(found);
}

export function createProposal(req, res) {
  // ดึง advisors ส่งให้ validator — pure function รับทาง parameter
  const advisors = advisorService.listAdvisors();
  const errors = validateProposalInput(req.body, { advisors });
  if (errors.length > 0) {
    return res.status(400).json({ error: 'ข้อมูลข้อเสนอโครงงานไม่ถูกต้อง', details: errors });
  }
  const created = service.create(req.body);
  res.status(201).json(created);
}

export function updateProposalStatus(req, res) {
  const { status } = req.body ?? {};
  if (!isValidStatus(status)) {
    return res.status(400).json({
      error: 'สถานะต้องเป็น submitted, reviewing, approved หรือ rejected'
    });
  }
  const updated = service.updateStatus(req.params.id, status);
  if (!updated) {
    return res.status(404).json({ error: `ไม่พบข้อเสนอโครงงานรหัส ${req.params.id}` });
  }
  console.log(`[status] ${updated.id} → ${updated.status}`);
  res.status(200).json(updated);
}

export function deleteProposal(req, res) {
  const removed = service.remove(req.params.id);
  if (!removed) {
    return res.status(404).json({ error: `ไม่พบข้อเสนอโครงงานรหัส ${req.params.id}` });
  }
  res.status(204).end();
}
```

## M1-C10 · `api/src/routes/proposalRoutes.js`

```js
import { Router } from 'express';
import * as controller from '../controllers/proposalController.js';
import { authenticate, requireRole } from '../middleware/auth.js';

const router = Router();

router.get('/',    controller.listProposals);
router.post('/',   controller.createProposal);
router.get('/:id', controller.getProposal);

// เฉพาะเจ้าหน้าที่/กรรมการหลักสูตร
router.put('/:id',    authenticate, requireRole('staff'), controller.updateProposalStatus);
router.delete('/:id', authenticate, requireRole('staff'), controller.deleteProposal);

export default router;
```

## M1-C11 · `api/src/routes/advisorRoutes.js` — ไฟล์ใหม่

```js
import { Router } from 'express';
import * as service from '../services/advisorService.js';

const router = Router();

router.get('/', (req, res) => {
  res.status(200).json(service.listAdvisors());
});

router.get('/:id', (req, res) => {
  const adv = service.findAdvisor(req.params.id);
  if (!adv) return res.status(404).json({ error: `ไม่พบอาจารย์รหัส ${req.params.id}` });
  res.status(200).json(adv);
});

export default router;
```

## M1-C12 · `api/src/app.js` — เปลี่ยน import + mount

หา 2 บรรทัดนี้แล้วเปลี่ยน:

```js
// ของเดิม
import requestRoutes from './routes/requestRoutes.js';
app.use('/api/requests', requestRoutes);
```

เป็น:

```js
import proposalRoutes from './routes/proposalRoutes.js';
import advisorRoutes from './routes/advisorRoutes.js';
// ...
app.use('/api/proposals', proposalRoutes);
app.use('/api/advisors', advisorRoutes);
```

## M1-C13 · `api/src/server.js` — เพิ่ม upsertStaff

เขียนทั้งไฟล์ใหม่:

```js
import { config } from './config.js';
import { createApp } from './app.js';
import { loadSeed, upsertStaff } from './services/proposalService.js';
import { hashPassword } from './utils/password.js';

await loadSeed();

// production: ตั้งรหัส staff จาก env · บัญชี seed ใช้ไม่ได้บน production
if (process.env.STAFF_EMAIL && process.env.STAFF_PASSWORD) {
  upsertStaff({
    email: process.env.STAFF_EMAIL,
    name: 'เจ้าหน้าที่หลักสูตร',
    passwordHash: hashPassword(process.env.STAFF_PASSWORD),
  });
  console.log('[staff] production staff account updated from env');
}

const app = createApp();

app.listen((config.port ?? 3001), (err) => {
  if (err) throw err;
  console.log(`T05 เสนอหัวข้อโครงงาน พร้อมที่ http://localhost:${config.port ?? 3001}`);
});
```

## M1-C14 · `api/src/services/authService.js` — แก้ import

เปลี่ยน:
```js
import { findUserByEmail } from './requestService.js';
```
เป็น:
```js
import { findUserByEmail } from './proposalService.js';
```

## M1-C15 · `api/scripts/setup-db.mjs`

เปิดไฟล์ · หา `'requests'` ทุกจุด · เปลี่ยนเป็น `'proposals'` · แก้ข้อความ log

## M1-C16 · ตรวจ import ทั้งหมด

```bash
grep -rn "requestService\|requestValidator\|requestController" api/src
# ทุกบรรทัดที่เจอ ต้องแก้
```

## M1-C17 · `project.config.json`

> ⚠ อย่าลืมเช็ก `github` และ `role` ของสมาชิกในทีมให้ตรงกับ `TEAM.md` ของกลุ่มตัวเองนะครับ

```json
{
  "team": "T-05",
  "section": 2,
  "topic": "T05",
  "members": [
    { "name": "จักรพงศ์ หมื่นไชยศรี", "studentId": "68543210003-8", "github": "<ใส่-github-ตัวเอง>", "role": "be+devops" }
  ],
  "api": {
    "resource": "/api/proposals",
    "existingId": "PR-001",
    "statusField": "status",
    "statusFilter": "submitted",
    "statusUpdate": { "status": "reviewing" },
    "validSample": {
      "studentName": "ทดสอบ ระบบ",
      "advisorId": "ADV-002",
      "title": "ระบบจัดการเอกสารโครงงานวิศวกรรมซอฟต์แวร์ออนไลน์",
      "category": "Web App",
      "teamSize": 2,
      "scope": "พัฒนาเว็บแอปพลิเคชันสำหรับส่งหัวข้อโครงงานและติดตามสถานะการอนุมัติ"
    },
    "invalidSample": {
      "studentName": "ก",
      "advisorId": "ADV-002",
      "title": "สั้น",
      "category": "Web App",
      "teamSize": 10,
      "scope": "สั้นเกิน"
    },
    "ruleSamples": [
      {
        "rule": "จำนวนสมาชิกเกิน 4 คน",
        "body": {
          "studentName": "ทดสอบ ระบบ",
          "advisorId": "ADV-002",
          "title": "ระบบจัดการเอกสารโครงงานวิศวกรรมซอฟต์แวร์",
          "category": "Web App",
          "teamSize": 5,
          "scope": "พัฒนาเว็บแอปพลิเคชันสำหรับส่งหัวข้อโครงงานและติดตามสถานะ"
        }
      },
      {
        "rule": "ชื่อโครงงานสั้นกว่า 10 ตัวอักษร",
        "body": {
          "studentName": "ทดสอบ ระบบ",
          "advisorId": "ADV-002",
          "title": "เว็บสั้นๆ",
          "category": "Web App",
          "teamSize": 2,
          "scope": "พัฒนาเว็บแอปพลิเคชันสำหรับส่งหัวข้อโครงงานและติดตามสถานะ"
        }
      }
    ]
  },
  "devStaff": { "email": "staff@rmutl.ac.th", "password": "staff1234" },
  "deployUrl": "[https://engse203-final-2-t05.onrender.com](https://engse203-final-2-t05.onrender.com)"
}
```

> `deployUrl` ใส่ค่าจริงหลัง M2-D1

## M1-C18 · รัน + ทดสอบ

```bash
npm install --prefix api
npm install --prefix frontend
npm run db:setup --prefix api

npm run dev --prefix api
```

terminal อื่น:
```bash
curl http://localhost:3001/api/health | jq
curl http://localhost:3001/api/proposals | jq '.[0]'
curl http://localhost:3001/api/advisors | jq '.[0]'
curl http://localhost:3001/api/proposals/PR-001 | jq
```

ควรเห็น `studentName` · `advisorName` · `availableQuota` ครบ

## M1-C19 · Commit + tag `v0.1.0`

```bash
rm -rf api/node_modules frontend/node_modules api/data/*.db
git ls-files | grep -E "\.env$|\.db$|node_modules" && echo "⚠ ยังมี!" || echo "✓ สะอาด"

git add -A
git commit -m "M1: migrate LAB 13 to T05 proposals + advisors schema"
git tag v0.1.0
git push origin main
git push origin v0.1.0
```

---

# M2 · Frontend + Deploy → `v0.5.0`

## M2-F1 · `frontend/src/services/apiClient.js` — ตรวจ

LAB 13 มีให้แล้ว · ใช้ `VITE_API_BASE_URL` และมี `auth.setToken/getToken/clearToken` · ข้ามข้อนี้

## M2-F2 · `frontend/src/services/proposalService.js` — ไฟล์ใหม่

```js
import { apiFetch } from './apiClient.js';

export function listProposals({ status, q } = {}) {
  const params = new URLSearchParams();
  if (status) params.set('status', status);
  if (q) params.set('q', q);
  const qs = params.toString();
  return apiFetch(`/api/proposals${qs ? '?' + qs : ''}`);
}

export function getProposal(id) {
  return apiFetch(`/api/proposals/${id}`);
}

export function createProposal(input) {
  return apiFetch('/api/proposals', { method: 'POST', body: JSON.stringify(input) });
}

export function updateProposalStatus(id, status) {
  return apiFetch(`/api/proposals/${id}`, { method: 'PUT', body: JSON.stringify({ status }) });
}

export function deleteProposal(id) {
  return apiFetch(`/api/proposals/${id}`, { method: 'DELETE' });
}

export function listAdvisors() {
  return apiFetch('/api/advisors');
}

export function login(email, password) {
  return apiFetch('/api/auth/login', {
    method: 'POST',
    body: JSON.stringify({ email, password }),
  });
}
```

## M2-F3 · `frontend/src/pages/LoginPage.jsx`

```jsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { login } from '../services/proposalService.js';
import { auth } from '../services/apiClient.js';

export default function LoginPage() {
  const nav = useNavigate();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);

  async function onSubmit(e) {
    e.preventDefault();
    setError(''); setLoading(true);
    try {
      const result = await login(email, password);
      auth.setToken(result.token);
      localStorage.setItem('user', JSON.stringify(result.user));
      nav('/');
    } catch (err) {
      setError(err.message || 'เข้าสู่ระบบไม่สำเร็จ');
    } finally {
      setLoading(false);
    }
  }

  return (
    <main className="auth-page">
      <h1>เข้าสู่ระบบเจ้าหน้าที่หลักสูตร</h1>
      <form onSubmit={onSubmit}>
        <label>อีเมล<input type="email" value={email}
          onChange={e => setEmail(e.target.value)} required /></label>
        <label>รหัสผ่าน<input type="password" value={password}
          onChange={e => setPassword(e.target.value)} required /></label>
        {error && <p role="alert" className="error">{error}</p>}
        <button disabled={loading}>{loading ? 'กำลังเข้าสู่ระบบ…' : 'เข้าสู่ระบบ'}</button>
      </form>
    </main>
  );
}
```

## M2-F4 · `frontend/src/hooks/useAuth.js`

```js
import { auth } from '../services/apiClient.js';

export function useAuth() {
  const token = auth.getToken();
  let user = null;
  try { user = JSON.parse(localStorage.getItem('user') || 'null'); } catch { /* ignore */ }
  const isStaff = token && user?.role === 'staff';
  function logout() {
    auth.clearToken();
    localStorage.removeItem('user');
    window.location.href = '/login';
  }
  return { token, user, isStaff, logout };
}
```

## M2-F5 · `frontend/src/pages/ProposalsListPage.jsx`

```jsx
import { useEffect, useState } from 'react';
import { Link } from 'react-router-dom';
import { listProposals } from '../services/proposalService.js';
import LoadingState from '../components/LoadingState.jsx';
import ErrorState from '../components/ErrorState.jsx';

const STATUSES = ['', 'submitted', 'reviewing', 'approved', 'rejected'];

export default function ProposalsListPage() {
  const [proposals, setProposals] = useState([]);
  const [status, setStatus] = useState('');
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true); setError(null);
    listProposals({ status: status || undefined })
      .then(setProposals).catch(setError).finally(() => setLoading(false));
  }, [status]);

  if (loading) return <LoadingState/>;
  if (error) return <ErrorState message="{error.message}"/>;

  return (
    <main>
      <h1>รายการหัวข้อโครงงานที่เสนอ</h1>
      <label>กรองสถานะ:
        <select value={status} onChange={e => setStatus(e.target.value)}>
          {STATUSES.map(s => <option key={s} value={s}>{s || 'ทั้งหมด'}</option>)}
        </select>
      </label>
      {proposals.length === 0 ? <p>ไม่มีหัวข้อโครงงานในสถานะนี้</p> : (
        <ul className="proposals-list">
          {proposals.map(p => (
            <li key={p.id}>
              <Link to="{`/proposals/${p.id}`}">
                <strong>{p.id}</strong> · {p.title} ({p.category})
                <span className={`badge badge-${p.status}`}>{p.status}</span>
                <br />ผู้เสนอ: {p.studentName} ({p.teamSize} คน) · ที่ปรึกษา: {p.advisorName}
              </Link>
            </li>
          ))}
        </ul>
      )}
    </main>
  );
}
```

## M2-F6 · `frontend/src/pages/ProposalDetailPage.jsx`

```jsx
import { useEffect, useState } from 'react';
import { useParams, useNavigate } from 'react-router-dom';
import { getProposal, updateProposalStatus, deleteProposal } from '../services/proposalService.js';
import { useAuth } from '../hooks/useAuth.js';
import LoadingState from '../components/LoadingState.jsx';
import ErrorState from '../components/ErrorState.jsx';

const NEXT_STATUS = ['submitted', 'reviewing', 'approved', 'rejected'];

export default function ProposalDetailPage() {
  const { id } = useParams();
  const nav = useNavigate();
  const { isStaff } = useAuth();
  const [proposal, setProposal] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    getProposal(id).then(setProposal).catch(setError).finally(() => setLoading(false));
  }, [id]);

  async function onStatusChange(newStatus) {
    try {
      const updated = await updateProposalStatus(id, newStatus);
      setProposal(updated);
    } catch (err) { alert(err.message); }
  }

  async function onDelete() {
    if (!confirm(`ลบข้อเสนอโครงงาน ${id}?`)) return;
    try { await deleteProposal(id); nav('/'); }
    catch (err) { alert(err.message); }
  }

  if (loading) return <LoadingState/>;
  if (error) return <ErrorState message="{error.message}"/>;
  if (!proposal) return <p>ไม่พบข้อเสนอโครงงาน</p>;

  return (
    <main>
      <h1>ข้อเสนอโครงงาน {proposal.id}</h1>
      <dl>
        <dt>ชื่อโครงงาน</dt><dd>{proposal.title}</dd>
        <dt>หมวดหมู่</dt><dd>{proposal.category}</dd>
        <dt>ผู้เสนอหลัก</dt><dd>{proposal.studentName}</dd>
        <dt>จำนวนสมาชิก</dt><dd>{proposal.teamSize} คน</dd>
        <dt>อาจารย์ที่ปรึกษา</dt><dd>{proposal.advisorName} ({proposal.advisorExpertise})</dd>
        <dt>ขอบเขตโครงงาน</dt><dd>{proposal.scope}</dd>
        <dt>สถานะ</dt><dd>{proposal.status}</dd>
      </dl>

      {isStaff && (
        <section className="staff-actions">
          <h2>การจัดการ (เจ้าหน้าที่หลักสูตร)</h2>
          {NEXT_STATUS.filter(s => s !== proposal.status).map(s => (
            <button key={s} onClick={() => onStatusChange(s)}>เปลี่ยนเป็น {s}</button>
          ))}
          <button onClick={onDelete} className="danger">ลบข้อเสนอโครงงาน</button>
        </section>
      )}
    </main>
  );
}
```

## M2-F7 · `frontend/src/pages/NewProposalPage.jsx`

```jsx
import { useEffect, useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { listAdvisors, createProposal } from '../services/proposalService.js';

const CATEGORIES = ['Web App', 'Mobile App', 'AI / Data', 'IoT', 'อื่นๆ'];

export default function NewProposalPage() {
  const nav = useNavigate();
  const [advisors, setAdvisors] = useState([]);
  const [form, setForm] = useState({
    studentName: '', advisorId: '', title: '',
    category: 'Web App', teamSize: 1, scope: '',
  });
  const [errors, setErrors] = useState([]);
  const [submitting, setSubmitting] = useState(false);

  useEffect(() => { listAdvisors().then(setAdvisors); }, []);

  function set(k, v) { setForm(prev => ({ ...prev, [k]: v })); }

  async function onSubmit(e) {
    e.preventDefault();
    setErrors([]); setSubmitting(true);
    try {
      const created = await createProposal({ ...form, teamSize: Number(form.teamSize) });
      nav(`/proposals/${created.id}`);
    } catch (err) {
      setErrors(err.details ?? [err.message]);
    } finally {
      setSubmitting(false);
    }
  }

  return (
    <main>
      <h1>ยื่นเสนอหัวข้อโครงงานใหม่</h1>
      <form onSubmit={onSubmit}>
        <label>ชื่อนักศึกษาผู้ยื่น
          <input value={form.studentName}
            onChange={e => set('studentName', e.target.value)} required />
        </label>
        <label>ชื่อหัวข้อโครงงาน (10–200 ตัวอักษร)
          <input value={form.title}
            onChange={e => set('title', e.target.value)} required />
        </label>
        <label>หมวดหมู่โครงงาน
          <select value={form.category}
            onChange={e => set('category', e.target.value)} required>
            {CATEGORIES.map(c => <option key={c} value={c}>{c}</option>)}
          </select>
        </label>
        <label>อาจารย์ที่ปรึกษา
          <select value={form.advisorId}
            onChange={e => set('advisorId', e.target.value)} required>
            <option value="">-- เลือกอาจารย์ที่ปรึกษา --</option>
            {advisors.map(adv => (
              <option key={adv.id} value={adv.id} disabled={adv.availableQuota <= 0}>
                {adv.name} — {adv.expertise} (โควตาเหลือ {adv.availableQuota}/{adv.maxQuota})
              </option>
            ))}
          </select>
        </label>
        <label>จำนวนสมาชิกในกลุ่ม (1–4 คน)
          <input type="number" min="1" max="4"
            value={form.teamSize} onChange={e => set('teamSize', e.target.value)} required />
        </label>
        <label>ขอบเขตและวัตถุประสงค์โครงงาน (อย่างน้อย 15 ตัวอักษร)
          <textarea value={form.scope} rows="3"
            onChange={e => set('scope', e.target.value)} required />
        </label>
        {errors.length > 0 && (
          <ul role="alert" className="errors">
            {errors.map((er, i) => <li key={i}>{er}</li>)}
          </ul>
        )}
        <button disabled={submitting}>{submitting ? 'กำลังส่ง…' : 'ส่งข้อเสนอโครงงาน'}</button>
      </form>
    </main>
  );
}
```

## M2-F8 · `frontend/src/App.jsx` — ตั้ง routes

```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
import ProposalsListPage from './pages/ProposalsListPage.jsx';
import ProposalDetailPage from './pages/ProposalDetailPage.jsx';
import NewProposalPage from './pages/NewProposalPage.jsx';
import LoginPage from './pages/LoginPage.jsx';
import DashboardPage from './pages/DashboardPage.jsx';
import { useAuth } from './hooks/useAuth.js';

function Nav() {
  const { isStaff, user, logout } = useAuth();
  return (
    <nav>
      <Link to="/">รายการโครงงาน</Link>
      <Link to="/new">เสนอหัวข้อใหม่</Link>
      <Link to="/dashboard">สรุปผล</Link>
      {isStaff ? (
        <>
          <span>· {user.name}</span>
          <button onClick={logout}>ออกจากระบบ</button>
        </>
      ) : <Link to="/login">เข้าสู่ระบบ</Link>}
    </nav>
  );
}

export default function App() {
  return (
    <BrowserRouter>
      <Nav/>
      <Routes>
        <Route element="{<ProposalsListPage" path="/"/>} />
        <Route element="{<ProposalDetailPage" path="/proposals/:id"/>} />
        <Route element="{<NewProposalPage" path="/new"/>} />
        <Route element="{<DashboardPage" path="/dashboard"/>} />
        <Route element="{<LoginPage" path="/login"/>} />
      </Routes>
    </BrowserRouter>
  );
}
```

## M2-F9 · `frontend/src/pages/DashboardPage.jsx`

```jsx
import { useEffect, useState } from 'react';
import { listProposals } from '../services/proposalService.js';
import { summarizeByStatus } from '../utils/proposalSummary.js';

export default function DashboardPage() {
  const [summary, setSummary] = useState(null);
  useEffect(() => {
    listProposals().then(proposals => setSummary(summarizeByStatus(proposals)));
  }, []);
  if (!summary) return <p>กำลังโหลด…</p>;
  return (
    <main>
      <h1>สรุปภาพรวมข้อเสนอโครงงาน</h1>
      <div className="cards">
        {Object.entries(summary).map(([status, count]) => (
          <div key={status} className={`card status-${status}`}>
            <h2>{status}</h2><p className="count">{count}</p>
          </div>
        ))}
      </div>
    </main>
  );
}
```

## M2-F10 · `frontend/src/utils/proposalSummary.js`

```js
export function summarizeByStatus(proposals) {
  const init = { submitted: 0, reviewing: 0, approved: 0, rejected: 0 };
  return proposals.reduce((acc, p) => {
    if (acc[p.status] !== undefined) acc[p.status]++;
    return acc;
  }, init);
}
```

## M2-F11 · ลบไฟล์ LAB เดิม

```bash
cd ~/engse203-final-2-T-05/frontend/src
rm -f pages/NewRequestPage.jsx pages/RequestDetailPage.jsx pages/AboutPage.jsx
rm -f components/RequestCard.jsx components/RequestForm.jsx components/RequestList.jsx
rm -f services/requestService.js utils/requestSummary.js utils/requestSummary.test.js
# เก็บ LoadingState · ErrorState · FilterBar · SummaryPanel ไว้ reuse
```

## M2-F12 · `frontend/vite.config.js`

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  base: '/',
  build: { outDir: 'dist', emptyOutDir: true },
});
```

## M2-F13 · `frontend/.env.production`

```
VITE_API_BASE_URL=https://<render-service-name>.onrender.com
```

> 💡 ถ้า Express serve `frontend/dist` บน URL เดียวกันกับ API (เป็นแบบนั้นใน LAB 13 ด้วย `config.staticDir`) ให้เว้นไฟล์นี้ไปเลย · `apiClient` จะ fallback เป็น relative URL แล้วเรียก `/api/*` ของ origin เดียวกัน ใช้ได้ทันทีไม่ต้องกังวลเรื่อง CORS

## M2-D1 · ตั้ง Render

1. `render.yaml` → ลบบรรทัด `rootDir:` ถ้ามี
2. Push ขึ้น GitHub → Render Dashboard → New → Blueprint → ชี้ที่ repo
3. Environment variables:
   - `STAFF_EMAIL=staff@rmutl.ac.th` (ตรงกับ seed)
   - `STAFF_PASSWORD=<รหัสยาว ≥ 12 ตัว>` ← **เก็บไว้ · ไม่ commit**
   - `JWT_SECRET` → ปล่อย `render.yaml` generate
   - `NODE_ENV=production`
   - `CORS_ORIGIN=https://<render-url>.onrender.com`

## M2-D2 · ทดสอบ production

```bash
curl https://<url>[.onrender.com/api/health](https://.onrender.com/api/health)
curl https://<url>[.onrender.com/api/proposals](https://.onrender.com/api/proposals) | head -c 500

curl -X POST https://<url>[.onrender.com/api/auth/login](https://.onrender.com/api/auth/login) \
  -H "Content-Type: application/json" \
  -d '{"email":"staff@rmutl.ac.th","password":"<PROD_PASSWORD>"}'

# ลองด้วย staff1234 → ต้อง 401
curl -X POST https://<url>[.onrender.com/api/auth/login](https://.onrender.com/api/auth/login) \
  -H "Content-Type: application/json" \
  -d '{"email":"staff@rmutl.ac.th","password":"staff1234"}'
```

## M2-X · Tag `v0.5.0`

```bash
git add -A
git commit -m "M2: proposals API + FE pages + login + first Render deploy"
git tag v0.5.0 && git push && git push origin v0.5.0
```

---

# M2.5 · Tests + Debug Log

## M2.5-T1 · `api/tests/unit/proposalValidator.test.js`

```js
import { describe, it, expect } from 'vitest';
import { validateProposalInput } from '../../src/validators/proposalValidator.js';

const advisors = [
  { id: 'ADV-001', name: 'ผศ.ดร. กิตติพงศ์', availableQuota: 2, maxQuota: 3 },
  { id: 'ADV-002', name: 'อ.ดร. นภาลัย',     availableQuota: 0, maxQuota: 4 },
];

const base = {
  studentName: 'ทดสอบ ระบบ',
  advisorId: 'ADV-001',
  title: 'ระบบจัดการเอกสารโครงงานวิศวกรรมซอฟต์แวร์',
  category: 'Web App',
  teamSize: 2,
  scope: 'พัฒนาเว็บแอปพลิเคชันสำหรับยื่นหัวข้อโครงงานและติดตามสถานะ',
};

describe('validateProposalInput', () => {
  it('ข้อมูลถูกครบ → ไม่มี error', () => {
    expect(validateProposalInput(base, { advisors })).toEqual([]);
  });

  it('ชื่อผู้เสนอสั้นเกิน', () => {
    const errs = validateProposalInput({ ...base, studentName: 'ก' }, { advisors });
    expect(errs.some(e => e.includes('ชื่อผู้เสนอ'))).toBe(true);
  });

  it('advisorId ไม่มีในระบบ', () => {
    const errs = validateProposalInput({ ...base, advisorId: 'ADV-999' }, { advisors });
    expect(errs.some(e => e.includes('ไม่พบอาจารย์'))).toBe(true);
  });

  it('⭐ กฎเฉพาะ ค่าขอบ: teamSize = 0 (ต่ำกว่าเกณฑ์)', () => {
    const errs = validateProposalInput({ ...base, teamSize: 0 }, { advisors });
    expect(errs.some(e => e.includes('จำนวนสมาชิก'))).toBe(true);
  });

  it('⭐ กฎเฉพาะ ค่าขอบ: teamSize = 4 (ขอบบน · ผ่าน)', () => {
    const errs = validateProposalInput({ ...base, teamSize: 4 }, { advisors });
    expect(errs).toEqual([]);
  });

  it('⭐ กฎเฉพาะ ค่าขอบ: teamSize = 5 (เกิน 4 คน)', () => {
    const errs = validateProposalInput({ ...base, teamSize: 5 }, { advisors });
    expect(errs.some(e => e.includes('1–4 คน'))).toBe(true);
  });

  it('⭐ กฎเฉพาะ: เลือกอาจารย์ที่โควตาเต็มแล้ว (availableQuota = 0)', () => {
    const errs = validateProposalInput(
      { ...base, advisorId: 'ADV-002' }, { advisors }
    );
    expect(errs.some(e => e.includes('โควตาเต็ม'))).toBe(true);
  });

  it('⭐ กฎเฉพาะ ค่าขอบ: ชื่อโครงงานยาว 10 ตัวอักษรพอดี (ผ่าน)', () => {
    const errs = validateProposalInput(
      { ...base, title: '1234567890' }, { advisors }
    );
    expect(errs).toEqual([]);
  });

  it('⭐ กฎเฉพาะ ค่าขอบ: ชื่อโครงงานยาว 9 ตัวอักษร (สั้นเกิน)', () => {
    const errs = validateProposalInput(
      { ...base, title: '123456789' }, { advisors }
    );
    expect(errs.some(e => e.includes('ชื่อโครงงาน'))).toBe(true);
  });

  it('ไม่ได้ระบุหมวดหมู่โครงงาน', () => {
    const errs = validateProposalInput({ ...base, category: '   ' }, { advisors });
    expect(errs.some(e => e.includes('หมวดหมู่'))).toBe(true);
  });

  it('ขอบเขตโครงงาน (scope) สั้นเกิน 15 ตัวอักษร', () => {
    const errs = validateProposalInput({ ...base, scope: 'สั้นเกินไป' }, { advisors });
    expect(errs.some(e => e.includes('ขอบเขตโครงงาน'))).toBe(true);
  });

  it('ส่ง null → error', () => {
    expect(validateProposalInput(null)).toHaveLength(1);
  });
});
```

ครบ 12 ข้อ · ค่าขอบของกฎเฉพาะครบถ้วน

## M2.5-T2 · `api/tests/integration/proposals.test.js`

```js
import { describe, it, expect, beforeAll } from 'vitest';
import request from 'supertest';
import { createApp } from '../../src/app.js';
import { loadSeed } from '../../src/services/proposalService.js';

let app, staffToken;

beforeAll(async () => {
  process.env.DB_FILE = ':memory:';
  await loadSeed();
  app = createApp();

  const res = await request(app).post('/api/auth/login')
    .send({ email: 'staff@rmutl.ac.th', password: 'staff1234' });
  staffToken = res.body.token;
});

describe('GET /api/proposals', () => {
  it('200 + array', async () => {
    const res = await request(app).get('/api/proposals');
    expect(res.status).toBe(200);
    expect(Array.isArray(res.body)).toBe(true);
    expect(res.body.length).toBeGreaterThan(0);
  });

  it('มี advisorName จาก JOIN', async () => {
    const res = await request(app).get('/api/proposals');
    expect(res.body[0]).toHaveProperty('advisorName');
  });

  it('กรองสถานะด้วย ?status=', async () => {
    const res = await request(app).get('/api/proposals?status=approved');
    expect(res.body.every(p => p.status === 'approved')).toBe(true);
  });
});

describe('GET /api/proposals/:id', () => {
  it('ไม่พบ → 404', async () => {
    const res = await request(app).get('/api/proposals/PR-999');
    expect(res.status).toBe(404);
  });
});

describe('POST /api/proposals', () => {
  it('body ถูก → 201', async () => {
    const res = await request(app).post('/api/proposals').send({
      studentName: 'ทดสอบ สมบูรณ์',
      advisorId: 'ADV-004',
      title: 'ระบบวิเคราะห์ข้อมูลการลงทะเบียนเรียนด้วยเทคนิคเหมืองข้อมูล',
      category: 'AI / Data',
      teamSize: 2,
      scope: 'รวบรวมข้อมูลและสร้างโมเดลพยากรณ์แนวโน้มผลการเรียนของนักศึกษา',
    });
    expect(res.status).toBe(201);
    expect(res.body.id).toMatch(/^PR-\d{3}$/);
  });

  it('body ผิด → 400 + details', async () => {
    const res = await request(app).post('/api/proposals').send({ studentName: 'ก' });
    expect(res.status).toBe(400);
    expect(res.body.details.length).toBeGreaterThan(0);
  });

  it('สมาชิกกลุ่มเกิน 4 คน → 400', async () => {
    const res = await request(app).post('/api/proposals').send({
      studentName: 'ทดสอบ กฎ',
      advisorId: 'ADV-004',
      title: 'ระบบจัดการโครงงานสำหรับนักศึกษาปริญญาตรี',
      category: 'Web App',
      teamSize: 6,
      scope: 'พัฒนาระบบเว็บสำหรับส่งรายงานความก้าวหน้าโครงงาน',
    });
    expect(res.status).toBe(400);
  });
});

describe('PUT /api/proposals/:id — auth', () => {
  it('ไม่มี token → 401', async () => {
    const res = await request(app).put('/api/proposals/PR-001').send({ status: 'reviewing' });
    expect(res.status).toBe(401);
  });

  it('มี token staff → 200', async () => {
    const res = await request(app).put('/api/proposals/PR-002')
      .set('Authorization', `Bearer ${staffToken}`)
      .send({ status: 'approved' });
    expect(res.status).toBe(200);
    expect(res.body.status).toBe('approved');
  });
});

describe('DELETE /api/proposals/:id', () => {
  it('ไม่มี token → 401', async () => {
    const res = await request(app).delete('/api/proposals/PR-004');
    expect(res.status).toBe(401);
  });

  it('มี token staff → 204', async () => {
    const res = await request(app).delete('/api/proposals/PR-004')
      .set('Authorization', `Bearer ${staffToken}`);
    expect(res.status).toBe(204);
  });
});

// 403 — เซ็น token ด้วย role อื่น แล้วลอง PUT
import jwt from 'jsonwebtoken';
import { config } from '../../src/config.js';

describe('403 role check', () => {
  it('student → 403', async () => {
    const studentToken = jwt.sign(
      { sub: '999', name: 'fake', role: 'student' },
      config.jwtSecret
    );
    const res = await request(app).put('/api/proposals/PR-001')
      .set('Authorization', `Bearer ${studentToken}`)
      .send({ status: 'approved' });
    expect(res.status).toBe(403);
  });
});
```

ครบ 13 ข้อ · 401 · 403 · staff ทำได้

## M2.5-T3 · `DEBUG_LOG.md` ตัวอย่าง 2 bug

```markdown
# DEBUG_LOG · T05 เสนอหัวข้อโครงงาน

## Bug #1 — โควตาอาจารย์ที่ปรึกษา (availableQuota) ลดลงแม้โครงงานถูกปัดตก (rejected)

**เมื่อไหร่:** 2026-10-15 ตอนทดสอบเปลี่ยนสถานะโครงงานเป็น `rejected` แล้วเช็ก `GET /api/advisors`
**อาการ:** โครงงานที่ถูก `rejected` ไปแล้วยังถูกนำมานับรวมว่าใช้โควตาของอาจารย์อยู่ ทำให้ `availableQuota` เหลือน้อยกว่าความเป็นจริง
**คาดว่า:** โครงงานที่สถานะ `rejected` ต้องคืนโควตาให้อาจารย์ที่ปรึกษา
**ตามรอย:** ตรวจสอบฟังก์ชัน `availableQuotaFor()` ใน `proposalService.js` พบว่าเงื่อนไข SQL นับรวมทุกสถานะรวมถึง `'rejected'`
**แก้:** ใน `availableQuotaFor()` เปลี่ยน `status IN ('submitted','reviewing','approved','rejected')` → `status IN ('submitted','reviewing','approved')`
**กันกลับ:** เพิ่ม test ตรวจสอบว่าเมื่อโครงงานมีสถานะ `rejected` จะไม่ถูกหักออกจาก `availableQuota`

## Bug #2 — ข้อมูลหายทุกครั้งที่ restart server

**เมื่อไหร่:** 2026-10-14 ตอนตั้งค่า `schema.sql` ครั้งแรก
**อาการ:** เมื่อ restart API แล้วข้อเสนอโครงงานที่เพิ่งสร้างด้วย `POST /api/proposals` หายไปทั้งหมด กลับเป็นข้อมูล seed เริ่มต้น
**คาดว่า:** ข้อมูลที่เพิ่มใหม่ต้องคงอยู่ในไฟล์ `proposals.db` ระหว่างการ restart ในโหมด dev
**ตามรอย:** ฟังก์ชัน `loadSeed()` ใน `proposalService.js` ยังเช็กชื่อตารางเดิมจาก LAB 13 คือ `name='requests'` ซึ่งไม่มีอยู่จริง (เพราะเปลี่ยนชื่อตารางเป็น `proposals` แล้ว) ทำให้เงื่อนไข `!ready` เป็นจริงเสมอและรัน `schema.sql` ที่มีคำสั่ง `DROP TABLE` ทุกครั้งที่เปิดเซิร์ฟเวอร์
**แก้:** เปลี่ยน `name='requests'` → `name='proposals'` ใน `loadSeed()` และตรวจเช็กทุกไฟล์ด้วย `grep -rn "requests" api/src`
**กันกลับ:** เขียน integration test ที่เรียก `loadSeed()` ซ้ำแล้วตรวจว่าข้อมูลที่เพิ่งสร้างยังคงอยู่ในฐานข้อมูล
```

---

# M3 · Release → `v1.0.0`

## M3-R1 · README.md ติ๊กทุกส่วน

- หัว: ชื่อระบบ (`T05 เสนอหัวข้อโครงงาน`) · URL · เวอร์ชัน
- สมาชิก (ตารางครบ)
- ความสามารถของระบบ (นักศึกษา / เจ้าหน้าที่หลักสูตร)
- ภาพหน้าจอ (desktop + มือถือ · ≥ 2 ภาพ)
- ข้อมูล (ตาราง `users`, `advisors`, `proposals` + ความสัมพันธ์)
- ติดตั้ง · รัน · test
- Deploy (URL · ลิงก์ไป DEPLOY.md)
- บัญชีทดสอบ dev (production ไม่เขียน)
- **การใช้ AI** — เขียนตรงๆ:
  ```markdown
  ## การใช้ AI
  - ใช้ AI ช่วยปรับโครงสร้างเอกสาร PLAN · STEPS · BUILD guide ให้ตรงกับหัวข้อ T05
  - review schema design · validator rules
  - อธิบายโค้ดทุกบรรทัดได้ด้วยตัวเอง
  ```

## M3-R2 · `API_CONTRACT.md`

ทุก endpoint (`/api/proposals`, `/api/advisors`, `/api/auth/login`) · ตัวอย่าง request/response จริง (copy จาก curl) · ข้อความ error

## M3-R3 · ติ๊ก `RELEASE_CHECKLIST.md`

ทีละข้อ · ใส่หลักฐาน (จำนวน test · ลิงก์ CI run · URL · ภาพ)

## M3-R4 · check-project.mjs

```bash
node --disable-warning=ExperimentalWarning check-project.mjs
node --disable-warning=ExperimentalWarning check-project.mjs --online
```

ไม่มี FAIL แดง

## M3-R5 · ตรวจ security

```bash
git ls-files | grep -E '\.env$' | grep -v example   # ว่าง
git ls-files | grep node_modules                     # ว่าง
git ls-files | grep '\.db$'                          # ว่าง
npm audit --prefix api
npm audit --prefix frontend
```

## M3-R6 · Tag `v1.0.0`

```bash
git status                              # clean
git log --oneline -5

git tag v1.0.0
git push origin main
git push origin v1.0.0
```

## M3-R7 · ส่ง LMS

- ลิงก์ repo + URL deploy ก่อนเวลาสอบ 22 ต.ค.
- **หยุด commit** หลังจากนี้

---

# ลำดับ copy-paste

ค้นหา section ด้วย Ctrl+F:

| ขั้น | section |
|---|---|
| M1 (19 ข้อ) | M1-C1 → M1-C19 |
| M2 FE (10 ข้อ) | M2-F1 → M2-F13 |
| M2 Deploy (3 ข้อ) | M2-D1 · M2-D2 · M2-X |
| M2.5 Tests (3 ข้อ) | M2.5-T1 · M2.5-T2 · M2.5-T3 |
| M3 Release (7 ข้อ) | M3-R1 → M3-R7 |

**โค้ดที่พร้อม copy:**
- SQL: `schema.sql`
- Backend: `proposalService.js` · `advisorService.js` · `proposalValidator.js` · `proposalController.js` · `proposalRoutes.js` · `advisorRoutes.js` · `server.js`
- Frontend: `proposalService.js` · `LoginPage` · `ProposalsListPage` · `ProposalDetailPage` · `NewProposalPage` · `DashboardPage` · `App` · `useAuth` · `proposalSummary`
- Tests: `proposalValidator.test.js` (12 ข้อ) · `proposals.test.js` (13 ข้อ)
- Config: `project.config.json`
- Debug: `DEBUG_LOG.md` 2 bug