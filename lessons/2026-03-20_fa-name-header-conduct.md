# FA Name ใน Header/Footer เป็น Conduct — ห้ามเปลี่ยน

**Date**: 2026-03-20
**Severity**: HIGH
**Category**: conduct
**Reported by**: แบงค์
**Project**: FA Tools (iAgencyAIA)

## What Happened

Security audit flag "hardcoded FA name" เป็น security issue → Dev/FE เปลี่ยนเป็น generic "FA Name" → แบงค์สั่ง revert ทันที เพราะชื่อจริงใน header/footer เป็น conduct requirement

## Root Cause

Security Oracle ไม่รู้ว่า FA name ใน header/footer เป็น conduct (business requirement) ไม่ใช่ hardcoded bug → flag ผิด → Dev/FE แก้ตาม → พัง conduct

## Impact

- **Users affected**: ลูกค้าเห็น "FA Name" แทนชื่อจริง
- **Duration**: จนกว่าจะ revert
- **Data affected**: ไม่มี — display only

## Fix

FE revert header/footer กลับมาแสดงชื่อจริง (commit 343ac449)

## Prevention

1. **Conduct items ต้องมี tag** ใน code — comment `// CONDUCT: ห้ามเปลี่ยน`
2. **Security audit ต้องเช็ค conduct list** ก่อน flag — ไม่ใช่ทุก hardcoded string เป็น bug
3. **ถามแบงค์ก่อนเปลี่ยน** ข้อมูลที่แสดงให้ลูกค้า

## Lesson

> ไม่ใช่ทุก "hardcoded" เป็น bug — บาง hardcoded เป็น business requirement ต้องเข้าใจ context ก่อนแก้
