# Weak JWT Validation ≠ JWT Validation — Header Check ไม่พอ

**Date**: 2026-03-20
**Severity**: CRITICAL
**Category**: security
**Reported by**: Security-Oracle
**Project**: FA Tools (iAgencyAIA)

## What Happened

Full security audit พบว่า 14/16 Supabase Edge Functions มี "auth check" แต่จริงๆ แค่เช็คว่า Authorization header มีอยู่ไหม — ไม่ได้ validate JWT token จริง ใครก็ส่ง `Bearer abc123` มาผ่านได้

## Root Cause

Dev implement auth แบบ:
```typescript
// ❌ WRONG — แค่เช็คว่า header มี ไม่ได้ validate token
const authHeader = req.headers.get('authorization');
if (!authHeader) return 401;
```

แทนที่จะ:
```typescript
// ✅ CORRECT — validate token กับ Supabase
const token = authHeader.replace('Bearer ', '');
const { data: { user }, error } = await supabase.auth.getUser(token);
if (error || !user) return 401;
```

## Impact

- **Users affected**: ทุกคนที่ใช้ FA Tools — unauthenticated access เป็นไปได้
- **Duration**: ตั้งแต่ deploy จนแก้ (commit 800d2ffb)
- **Data affected**: ข้อมูลลูกค้า, insurance applications, leads — ทุก table ที่ Edge Functions เข้าถึง

## Fix

สร้าง shared auth middleware `_shared/auth.ts` ที่ validate JWT จริง + deploy ทุก 16 functions (commit 800d2ffb)

## Prevention

1. **ใช้ shared middleware เสมอ** — อย่า implement auth เอง ใช้ `_shared/auth.ts`
2. **Security review ทุก Edge Function ใหม่** — เช็คว่า import `requireAuth()`
3. **Test auth ด้วย invalid token** — ไม่ใช่แค่ test ว่า valid token ผ่าน ต้อง test ว่า fake token ถูก reject

## Lesson

> "มี auth check" ≠ "มี auth" — ถ้าไม่ validate token signature กับ Supabase ก็เท่ากับไม่มี auth เลย
