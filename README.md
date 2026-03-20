# Oracle Lessons Learned

> "ผิดได้ แต่ผิดซ้ำไม่ได้" — แบงค์

Knowledge base สำหรับทีม Oracle ทั้งหมด — รวบรวมข้อผิดพลาด บทเรียน และ best practices จากการทำงานจริง

## How to Contribute

ทุก Oracle สามารถเพิ่ม lesson ได้ โดยสร้างไฟล์ใน folder ที่เหมาะสม:

```bash
# สร้าง lesson ใหม่
cp templates/lesson-template.md lessons/YYYY-MM-DD_short-slug.md
# แก้ไขเนื้อหา แล้ว commit
```

## Structure

```
lessons/           # บทเรียนทั้งหมด (เรียงตามวันที่)
categories/        # Index แยกตามหมวด
  production.md    # Production incidents
  security.md      # Security lessons
  integration.md   # Integration (Loveable, Supabase, GitHub)
  performance.md   # Performance optimization
  conduct.md       # Conduct & design rules
  process.md       # Process & workflow
templates/         # Template สำหรับเขียน lesson
```

## Categories

| Category | Description |
|----------|-------------|
| `production` | Production incidents — หน้าขาว, deploy พัง, data ผิด |
| `security` | Security issues — PII leaks, auth bypass, key exposure |
| `integration` | Integration — Loveable, Supabase, GitHub, API |
| `performance` | Performance — speed, bundle size, query optimization |
| `conduct` | Conduct rules — gradient, header/footer, naming |
| `process` | Process — workflow, communication, testing |

## How to Search

```bash
# ค้นหา keyword
grep -r "keyword" lessons/

# ดู lessons ตาม category
cat categories/production.md

# ดู lessons ล่าสุด
ls -t lessons/ | head -10
```

## Rules

1. **ทุก lesson ต้องมี**: สาเหตุ, ผลกระทบ, วิธีแก้, วิธีป้องกัน
2. **ไม่โทษคน** — focus ที่ระบบและ process
3. **เขียนให้คนอื่นเข้าใจ** — คนที่ไม่ได้อยู่ตอนเกิดเรื่องต้องอ่านรู้เรื่อง
4. **Update เมื่อมีข้อมูลใหม่** — lesson ไม่ใช่ static

---

*Maintained by BoB-Oracle | Contributions from all 16 Oracles*
