# iCompare Fix ทำ Production หน้าขาว

**Date**: 2026-03-20
**Severity**: CRITICAL
**Category**: production
**Reported by**: แบงค์
**Project**: FA Tools (iAgencyAIA)

## What Happened

FE แก้ bug off-by-one ใน iCompare rider data แต่ commit มี build error → Loveable build ไม่ผ่าน → production แสดงหน้าขาว ลูกค้าเข้าเว็บไม่ได้

## Root Cause

1. FE commit defensive filter `plans.filter(p => p?.planName)` แต่ไม่ได้เช็คว่า `plans` อาจเป็น `undefined` (data เก่า)
2. `TypeError: Cannot read properties of undefined` → blank page
3. **ไม่ได้ test บน Loveable build ก่อน push** — test แค่ local build

## Impact

- **Users affected**: ลูกค้าทุกคนที่เปิด shared link
- **Duration**: ~30 นาที
- **Data affected**: ไม่มี data loss แต่เข้าเว็บไม่ได้

## Fix

แบงค์ revert เอง ผ่าน Loveable โดยตรง

## Prevention

1. **ทุก commit ที่แก้ SharedProposal/iCompare** ต้อง test shared link ทุก format ก่อน push
2. **เพิ่ม `Array.isArray()` guard** ทุกที่ที่ iterate over data ที่อาจเป็น undefined
3. **Loveable build ≠ local build** — ต้องเช็คว่า Loveable build ผ่านก่อนถือว่า deploy สำเร็จ
4. **QA checklist ถาวร**: test shared link หลังทุก publish

## Lesson

> ข้อมูลถึงลูกค้าสำคัญที่สุด — fix bug แล้วสร้าง bug ใหม่ที่ร้ายกว่าเดิม ต้อง test ให้ครบก่อน push เสมอ
