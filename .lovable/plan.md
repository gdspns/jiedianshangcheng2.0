

## Plan

### Problem
1. **查单页面**：订单只显示套餐名，没有标注是"购买开通"还是"在线续费"
2. **时长显示错误**：管理员后台和用户查单都用 `order.months` + "个月" 显示时长，但实际天数可能是3天等自定义值，应该用 `duration_days` 显示

### Changes

#### 1. `src/pages/ClientPortal.tsx` — 查单结果

**Line 1761** — 套餐名后加订单类型标签：
```tsx
<span className="font-bold text-foreground text-lg">{order.plan_name}</span>
<span className="text-xs ml-2 px-2 py-0.5 rounded-full bg-muted font-medium">
  {order.order_type === "buy_new" ? "购买开通" : "在线续费"}
</span>
```

**Line 1778-1779** — 时长显示改用 `duration_days`：
```tsx
<span className="font-bold text-foreground">{order.duration_days || order.months * 30}天</span>
```

#### 2. `src/pages/AdminDashboard.tsx` — 管理员订单列表

**Interface (line 91)** — 添加字段：
```typescript
duration_days?: number;
order_type?: string;
```

**Line 1338-1339** — 套餐列显示改为：
```tsx
<span className="font-medium">{order.plan_name}</span>
<span className="text-muted-foreground text-xs ml-1">
  ({order.duration_days || order.months * 30}天 · {order.order_type === "buy_new" ? "购买开通" : "续费"})
</span>
```

### Files Changed
- `src/pages/ClientPortal.tsx` — 查单结果加订单类型标签，时长用天数显示
- `src/pages/AdminDashboard.tsx` — 订单列表套餐列显示天数和订单类型

