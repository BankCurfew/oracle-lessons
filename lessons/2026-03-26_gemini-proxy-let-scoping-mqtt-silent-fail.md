# `let` Scoping ใน try/catch ทำให้ MQTT Response ไม่เคย Publish

**Date**: 2026-03-26
**Severity**: CRITICAL
**Category**: integration
**Reported by**: Dev-Oracle + แบงค์
**Project**: Gemini Browser Proxy (claude-browser-proxy)

## What Happened

Gemini Proxy Chrome Extension เชื่อมต่อ MQTT ได้ปกติ (badge เขียว, status online) แต่เมื่อส่ง command เช่น `get_response`, `get_url`, `chat` ผ่าน MQTT → **ไม่มี response กลับมาเลย** มีแค่บาง command เช่น `list_tabs` ที่ทำงานได้

## Root Cause

`let tab;` ถูก declare ภายใน `try {}` block แต่ถูกอ้างอิงภายนอก `try/catch`:

```javascript
async function handleCommand(topic, command) {
  let result;
  try {
    // First switch — returns early (WORKS)
    switch (command.action) {
      case 'list_tabs': publish(...); return;
    }

    let tab;  // ← BLOCK-SCOPED to try
    // ... resolve tab ...

    // Second switch (BROKEN)
    switch (command.action) {
      case 'get_response': /* uses tab */ break;
    }
  } catch (err) {
    result = { error: err.message };
  }

  // OUTSIDE try — tab doesn't exist here!
  const response = { ...result, tabId: tab?.id };  // ← ReferenceError
  publish(TOPICS.response, response, true);         // ← NEVER REACHED
}
```

**`let`/`const` เป็น block-scoped** — ตัวแปรที่ declare ใน `try {}` ไม่สามารถเข้าถึงได้นอก block นั้น ทำให้เกิด `ReferenceError: tab is not defined`

เนื่องจาก `handleCommand` เป็น async function ที่ถูกเรียกโดยไม่มี `await` → ReferenceError กลายเป็น **silent unhandled promise rejection** ที่ไม่มี error ปรากฏที่ไหนเลย

## Impact

- **Users affected**: ทุก Oracle agent ที่ใช้ Gemini Proxy ผ่าน MQTT
- **Duration**: ตั้งแต่ code ถูกเขียน (ไม่รู้ว่าพังตั้งแต่เมื่อไหร่)
- **Data affected**: command ทุกตัวใน second switch (get_response, get_url, get_text, chat, wait_response, screenshot ฯลฯ) ไม่เคย return response

## Fix

ย้าย `let tab;` ออกมาที่ function scope:

```javascript
async function handleCommand(topic, command) {
  let result;
  let tab;  // ← MOVED HERE

  try {
    // ... first switch ...
    // ... tab resolution (assigns to outer tab) ...
    // ... second switch ...
  } catch (err) {
    result = { error: err.message };
  }

  const response = { ...result, tabId: tab?.id };  // ← NOW WORKS
  publish(TOPICS.response, response, true);
}
```

## Prevention

1. **ตัวแปรที่ใช้ข้าม try/catch ต้อง declare ก่อน try** — `let`/`const` ใน try ไม่มีผลข้างนอก
2. **async function ที่ถูกเรียกต้อง handle rejection** — `handleCommand(cmd)` ควรเป็น `handleCommand(cmd).catch(err => ...)` เพื่อไม่ให้ error หายไป
3. **Test ทุก code path** — ถ้ามี 2 switch, test ทั้ง 2; first switch ทำงานไม่ได้แปลว่า second switch ทำงาน
4. **Response publish ควรอยู่ใน case handler** — เหมือน `list_tabs` ที่ publish ใน case → ไม่พึ่ง shared code ข้างล่าง

## Lesson

> **`let` ใน try block ตายอยู่ใน try block** — ถ้าต้องใช้ตัวแปรหลัง catch, declare ก่อน try เสมอ ยิ่งถ้าเป็น async function ที่ไม่มี await, ReferenceError จะกลายเป็น silent fail ที่หาสาเหตุยากมาก
