# 武侠广告落地页

## 预约下载业务model

```typescript
const reserveDatas = reactive({
  sign_type: "md5",           // 签名类型
  account_type: "phone",      // 账号类型（手机号）
  area_code: "",             // 国际区号
  account: "",               // 用户账号（手机号）
  client_type: geteQuipment(0), // 客户端类型
  system_type: geteQuipment(1), // 系统类型
  cch_name: "",              // 渠道标识
  ts: "",                    // 时间戳（动态添加）
  nonce: ""                  // 随机数（动态添加）
});

```

### 1. **用户身份验证模型**

- `account_type`: 固定为"phone"，表示使用手机号验证
- `account`: 用户输入的手机号码
- `area_code`: 国际区号，支持不同地区用户

### 2. **设备信息模型**

- `client_type`: 客户端类型（通过 `geteQuipment(0)` 获取）
- `system_type`: 操作系统类型（通过 `geteQuipment(1)` 获取）

### 3. **安全验证模型**

- `sign_type`: 签名算法类型（MD5）
- `ts`: 请求时间戳（防重放攻击）
- `nonce`: 随机字符串（增加签名复杂度）

### 4. **营销追踪模型**

- `cch_name`: 广告渠道标识，用于数据归属和效果统计

#### 触发时机

只有当用户**主动点击预约按钮**时，才会将包含 `cch_name`（渠道信息）的完整 `reserveDatas` 提交给后台。

**数据流程**：

1. 页面加载 → 收集基础信息（IP、设备信息、渠道标识）
2. 用户填写手机号 → 验证输入
3. 用户点击预约 → **触发数据提交**
4. 提交成功 → 上报GA事件