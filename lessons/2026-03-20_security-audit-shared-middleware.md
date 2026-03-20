# Security Audit 4 Rounds — Shared Middleware ป้องกัน Regression

**Date**: 2026-03-20
**Severity**: CRITICAL
**Category**: security
**Reported by**: Dev-Oracle
**Project**: iagencyaiafatools (FA Tools)

## What Happened

Security-Oracle ทำ full audit พบ 13 CRITICAL findings ใน FA Tools:
- 14/16 Edge Functions มี weak JWT validation (แค่เช็ค header exists ไม่ได้ validate token)
- 15/16 Edge Functions ใช้ CORS wildcard `Access-Control-Allow-Origin: *`
- `parse-fund-peer-avg` ไม่มี auth เลย — ใครก็เขียน DB ได้
- History tables มี INSERT policy `WITH CHECK (true)` — ใครก็ปลอม audit record ได้

ใช้ 4 rounds verification กว่าจะ PASS: R1→R2→R3→R4

## Root Cause

แต่ละ Edge Function copy-paste CORS + auth pattern เอง → ไม่มี shared standard → บาง function ลืมใส่ auth, บาง function ใส่ weak auth (แค่เช็ค header ไม่ validate)

## Impact

- **Users affected**: ทุก FA ที่ใช้ระบบ + ลูกค้าที่มี data ใน DB
- **Duration**: ตั้งแต่ deploy จนกว่า fix (หลายเดือน)
- **Data affected**: fund performance data, audit history, ลูกค้า PII

## Fix

สร้าง shared middleware ใน `supabase/functions/_shared/`:
- `auth.ts` — `requireAuth(req)` validates JWT via `supabase.auth.getUser()`
- `cors.ts` — `getCorsHeaders(req)` returns origin-whitelisted headers

Apply ทุก 16 functions + history table RLS migration (4 commits)

## Prevention

1. **ทุก Edge Function ใหม่** ต้อง import `_shared/auth.ts` + `_shared/cors.ts`
2. **ห้าม copy-paste** CORS headers หรือ auth check — ใช้ shared module เท่านั้น
3. **Code review checklist**: ถ้าเห็น `'Access-Control-Allow-Origin': '*'` = block merge
4. **CI test** (proposal #15): smoke test ว่าทุก function return 401 without auth

## Lesson

> ทุก function copy-paste auth เอง = รอวันที่จะลืม. สร้าง shared middleware 1 ที่แล้วบังคับใช้ — fix ที่ 1 จุด ไม่ใช่ 16 จุด.
