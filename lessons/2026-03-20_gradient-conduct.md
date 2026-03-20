# Gradient Direction เป็น Conduct — อ่อนไปเข้มเสมอ

**Date**: 2026-03-20
**Severity**: MEDIUM
**Category**: conduct
**Reported by**: แบงค์
**Project**: FA Tools (iAgencyAIA)

## What Happened

แบงค์เจอ gradient ผิดทิศทางในหน้า iPlan — เข้ม→อ่อน แทนที่จะเป็น อ่อน→เข้ม ซึ่งเป็นกฎ conduct ที่แบงค์กำหนดไว้

## Root Cause

Developer ไม่ทราบกฎ gradient direction → ใช้ค่าที่ "ดูดี" แทนที่จะตาม spec

## Impact

- **Users affected**: ทุกคนที่เห็น UI
- **Duration**: จนกว่าจะตรวจเจอ
- **Data affected**: ไม่มี — visual only

## Fix

FE scan ทั้ง codebase — พบ 19 violations → แก้ทั้งหมด → 0 violations remaining

## Prevention

1. **Design System doc** ต้องระบุชัด: `from: อ่อน → to: เข้ม` เสมอ
2. **DocCon audit** gradient direction เป็น checklist item
3. **CSS variable** สำหรับ gradient — ไม่ hardcode ค่าสี

## Lesson

> Conduct rules ไม่ใช่ suggestion — เป็นกฎตายตัว ต้องอ่าน design spec ก่อน code ทุกครั้ง
