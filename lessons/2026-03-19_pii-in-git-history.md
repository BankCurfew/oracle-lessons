# PII ใน Git History — ลบไฟล์ไม่พอ ต้อง Purge History

**Date**: 2026-03-19
**Severity**: CRITICAL
**Category**: security
**Reported by**: Security-Oracle
**Project**: Data-Oracle

## What Happened

Security audit พบ 125+ customer records (ชื่อ, เบอร์โทร, email, วันเกิด) ใน Data-Oracle git repository — แม้ลบไฟล์แล้วก็ยังอยู่ใน git history

## Root Cause

Training data ที่มี PII ถูก commit เข้า git โดยไม่ได้ redact ก่อน → ลบไฟล์แล้วแต่ git history ยังเก็บ data เดิมอยู่

## Impact

- **Users affected**: ลูกค้า 125+ คน (PDPA risk)
- **Duration**: ตั้งแต่ commit แรกจนถึง purge
- **Data affected**: ชื่อ, เบอร์โทร, email, วันเกิด

## Fix

Data-Oracle rewrite git history — purge ทุก PII ออก แล้ว force push. Verified: 0 phones, 0 emails, 0 IDs, 0 DOBs remaining

## Prevention

1. **ห้าม commit real customer data** เข้า git เด็ดขาด
2. **Pre-commit hook** scan PII patterns (phone, email, ID card)
3. **Training data ต้อง anonymize** ก่อน commit เสมอ
4. **.gitignore** ครอบคลุม data files ที่อาจมี PII

## Lesson

> `git rm` ลบไฟล์จาก working tree แต่ git history ยังเก็บทุกอย่าง — PII ต้อง purge history ไม่ใช่แค่ลบไฟล์
