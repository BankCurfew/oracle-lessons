# Loveable Supabase — ไม่มีสิทธิ์ ALTER TABLE

**Date**: 2026-03-20
**Severity**: HIGH
**Category**: integration
**Reported by**: Dev-Oracle
**Project**: iagencyaiafatools (FA Tools)

## What Happened

FE ต้องการ i18n columns (benefit_name_en, category_en, note_en, unit_en) ใน product_benefits table. Dev สร้าง migration file, พยายาม apply ผ่าน:
1. Supabase MCP → permission denied (project ไม่อยู่ใน MCP account)
2. Supabase Dashboard → login ด้วย GitHub ไม่เห็น project
3. psql / supabase CLI → ไม่มี service_role key

สาเหตุ: FA Tools Supabase project (`rugcuukelivcferjjzek`) ถูกสร้างโดย Loveable — อยู่ใน Loveable's org ไม่ใช่ BankCurfew's org

## Root Cause

Loveable สร้าง Supabase project ให้อัตโนมัติเมื่อ deploy → project อยู่ใน Loveable's Supabase org → เราไม่มี admin access สำหรับ DDL operations

## Impact

- **Users affected**: FE-Oracle blocked — ต้องรอ migration
- **Duration**: 30+ นาที debug ก่อนรู้ว่าไม่มีสิทธิ์
- **Data affected**: ไม่มี (ไม่ได้แก้อะไร)

## Fix

ยกเลิก migration. ต้อง apply ผ่าน Loveable dashboard หรือขอสิทธิ์จากแบงค์

## Prevention

1. **ก่อน plan migration** → เช็คก่อนว่ามี DDL access หรือไม่
2. **Loveable projects**: DDL ต้องทำผ่าน Loveable dashboard (SQL editor ใน Loveable) ไม่ใช่ Supabase dashboard โดยตรง
3. **Document DB ownership** ของทุก project:
   - `heciyiepgxqtbphepalf` (AIA Oracle) → BankCurfew's org ✅ มี access
   - `rugcuukelivcferjjzek` (FA Tools) → Loveable's org ❌ ไม่มี DDL access

## Lesson

> Loveable สร้าง Supabase project ให้ = เราเป็น user ไม่ใช่ admin. DDL operations (ALTER TABLE, CREATE POLICY) ต้องทำผ่าน Loveable ไม่ใช่ Supabase โดยตรง.
