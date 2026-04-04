

## Problem

In `proxy-3xui/index.ts` line 159, when looking up a client:
```typescript
expiryTime: entry.expiryTime || clientStats?.expiryTime || inbound.expiryTime || 0,
```
If a UUID client has no expiry set (`expiryTime = 0`), JavaScript's `||` treats `0` as falsy and falls through to `inbound.expiryTime` — the inbound-level expiry. This causes:
1. "Remaining days" shows the inbound's expiry instead of "unlimited"
2. On renewal, the new expiry is calculated by adding months to the inbound's expiry date, giving a wrong result

On the frontend side (`ClientPortal.tsx` line 295):
```typescript
expiryDate: res.expiryDate || Date.now() + 30 * 86400000,
```
Again, `0` (meaning unlimited) is treated as falsy and replaced with a 30-day default.

## Plan

### 1. Fix `proxy-3xui/index.ts` — stop falling back to inbound expiry

Line 159: Change `||` chain to only use client-level expiry, not inbound-level:
```typescript
expiryTime: entry.expiryTime || clientStats?.expiryTime || 0,
```
Remove `inbound.expiryTime` from the chain. `0` means "no expiry / unlimited".

### 2. Fix `ClientPortal.tsx` — handle `expiryDate = 0` as unlimited

**Login handler (line 294-299):** Preserve `0` as a valid value meaning unlimited:
```typescript
expiryDate: res.expiryDate ?? 0,
```

**`getDaysLeft` (line 311):** Return `-1` (or a sentinel) for unlimited:
```typescript
const getDaysLeft = () => {
  if (clientData.expiryDate === 0) return -1; // unlimited
  return Math.max(0, Math.ceil((clientData.expiryDate - Date.now()) / 86400000));
};
```

**Dashboard display (line 786-790):** Show "无限期" when unlimited:
```tsx
{getDaysLeft() < 0 ? (
  <>
    <span className="text-3xl font-extrabold text-foreground">无限期</span>
  </>
) : (
  <>
    <span className="text-5xl font-extrabold text-foreground">{getDaysLeft()}</span>
    <span className="...">天</span>
  </>
)}
```

**Renewal expiry calculation (line 449-452):** When current expiry is 0 (unlimited), start from `Date.now()` instead:
```typescript
const baseExpiry = clientData.expiryDate === 0 ? Date.now() : clientData.expiryDate;
const newExpiry = new Date(baseExpiry);
newExpiry.setDate(newExpiry.getDate() + checkoutData.months * 30);
```

### Files Changed
- `supabase/functions/proxy-3xui/index.ts` — remove `inbound.expiryTime` fallback
- `src/pages/ClientPortal.tsx` — handle `0` as unlimited in login, display, and renewal

