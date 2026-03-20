# Sequential Supabase Queries ทำ Shared Link โหลดช้า 3-4 วินาที

**Date**: 2026-03-20
**Severity**: HIGH
**Category**: performance
**Reported by**: FE-Oracle
**Project**: FA Tools (iAgencyAIA)

## What Happened

SharedProposal page (iPlan, iCompare, iLink) ที่แชร์ให้ลูกค้า โหลดช้ามาก ~3-4 วินาที ทั้งที่ข้อมูลไม่ได้เยอะ — ลูกค้ารอนานโดยเฉพาะบน mobile

## Root Cause

`useFetchPolicyYearData` hook ทำ **11+ sequential Supabase queries** ทั้งที่ส่วนใหญ่ไม่ depend กัน:

```
entryRow → paymentPeriod → mainData → specialDiscount → saAdjustments → payouts → rider1 → rider2 → rider3 → calcType
```

แต่ละ query ~100-300ms → รวม 1.3-3.9 วินาที (แค่ hook เดียว ยังไม่รวม component render)

**สาเหตุที่ sequential:** เขียน `await` ต่อเนื่องโดยไม่ได้วิเคราะห์ว่า queries ไหน independent

## Impact

- **Users affected**: ลูกค้าทุกคนที่เปิด shared link
- **Duration**: ทุกครั้งที่เปิด link (persistent, ไม่ใช่ one-time)
- **Data affected**: ไม่ผิด แค่ช้า

## Fix

Restructure เป็น 2 parallel phases:

```
Phase 1 (Promise.all): entryRow | saAdjustments | payouts | allRiders
Phase 2 (Promise.all): mainData | specialDiscount | calcType
```

**ไม่เปลี่ยน logic** — same queries, same fallbacks, same output. แค่ reorder.

Expected: 3-4s → 1-1.5s load time

## Prevention

1. **วิเคราะห์ dependency ก่อนเขียน await** — ถ้า query ไม่ depend on ผลก่อนหน้า ใช้ `Promise.all()`
2. **Rule: ห้าม sequential await ใน loop** — `for + await` ต้องเปลี่ยนเป็น `Promise.all(array.map(...))`
3. **Performance budget**: shared link ต้องโหลด < 2 วินาทีบน 4G
4. **ทุก hook ที่มี > 3 Supabase calls** ต้อง review parallel opportunity

## Lesson

> Sequential await เป็น performance killer ที่มองไม่เห็น — 10 queries × 200ms = 2 วินาทีที่ลูกค้ารอ ทั้งที่ parallel ได้เหลือ 200ms
