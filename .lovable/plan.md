

## Problem

In `create-client/index.ts` line 159, the logic is:

```typescript
if (protocol === "socks" || salesProtocol === "mixed") {
  // generates SOCKS5 username+password
}
```

This means whenever the admin sets protocol to "mixed" (or the inbound is socks), it always generates SOCKS5 credentials — ignoring what the inbound's actual protocol is. When the admin sets a region to "vless", the `salesProtocol` variable correctly becomes `"vless"`, but the inbound on the 3x-ui panel might actually be a vless inbound. The current code should use the **inbound's actual protocol** (`protocol` from the panel) as the primary decision, and only fall back to socks5 generation when the protocol is literally "socks".

## Plan

### 1. Fix protocol decision logic in `create-client/index.ts`

Change line 159 from:
```typescript
if (protocol === "socks" || salesProtocol === "mixed") {
```
to:
```typescript
if (protocol === "socks") {
```

This ensures:
- If the 3x-ui inbound is **socks** protocol → generate username+password (SOCKS5)
- If the inbound is **vless/vmess/trojan** → generate UUID via `addClient` API

The `salesProtocol` from the region/config is already used to select which inbound to target (via `salesInboundId`). The actual client generation method should be determined by the inbound's real protocol, not the admin's label.

### 2. Add "trojan" and "socks" options to admin region protocol dropdown

Update the `<select>` in `AdminDashboard.tsx` (line 1111) to include all supported protocols:
- Mixed → Socks (用户名+密码)
- Vless (UUID)
- Vmess (UUID)  
- Trojan (UUID)

Rename "Mixed" to "Socks5" for clarity since that's what it actually generates.

### Files Changed
- `supabase/functions/create-client/index.ts` — fix protocol branching logic
- `src/pages/AdminDashboard.tsx` — update protocol dropdown options

