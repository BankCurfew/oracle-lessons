# iCompare Off-by-One — Rider Data ไม่ถูก Save

**Date**: 2026-03-20
**Severity**: HIGH
**Category**: production
**Reported by**: แบงค์
**Project**: FA Tools (iAgencyAIA)

## What Happened

ลูกค้าเปิด shared iCompare link แล้วเห็นข้อมูลผิด — คนละแผนเลย rider data ไม่ตรงกับที่ FA ทำ

## Root Cause

Off-by-one bug ใน `CompareMode.tsx` line 1395:

```js
// BUG: length = 1 → planIndex = 1 → เขียนที่ index 1 (ผิด!)
const planIndex = catSel?.plans[packageIndex]?.length || 0;

// FIX: ต้อง -1 เพื่อชี้ slot ที่เพิ่งสร้าง
const planIndex = Math.max((catSel?.plans[packageIndex]?.length || 1) - 1, 0);
```

เมื่อเพิ่ม rider → สร้าง empty slot `{}` ที่ index 0 → แต่เขียนข้อมูลที่ index 1 → index 0 ยังเป็น `{}` → rider ถูก save ไม่สมบูรณ์

## Impact

- **Users affected**: ลูกค้าที่ได้รับ iCompare link ที่มี rider
- **Duration**: ไม่ทราบ (bug อาจมีมานาน)
- **Data affected**: Rider entries ใน shared proposals แสดงผิด

## Fix

1. แก้ `planIndex` calculation ใน CompareMode.tsx
2. เพิ่ม defensive filter ใน SharedProposal.tsx กรอง empty entries

## Prevention

1. **Unit test สำหรับ handleAddRider** — test ว่า rider data ถูก save ถูก index
2. **QA test shared link ทุกครั้ง** ที่แก้ iCompare/CompareMode
3. **Array index calculation** ต้อง review ระวัง off-by-one เสมอ

## Lesson

> Off-by-one เป็น bug คลาสสิก — ทุกครั้งที่ใช้ `.length` เป็น index ต้องถามตัวเองว่าต้อง `-1` ไหม
