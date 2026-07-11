# 工学云 API 接口文档

## 基本信息

| 项目 | 说明 |
|------|------|
| 基础 URL | `https://api.moguding.net:9000/` |
| 请求方式 | 全部为 `POST` |
| Content-Type | `application/json; charset=utf-8` |
| User-Agent | `Dart/2.17 (dart:io)` |
| 版本 | `5.31.6` |
| 加密方式 | AES-ECB-PKCS5Padding，默认密钥 `23DbtQHR2UMbH6mJ` |
| 签名方式 | MD5(str1 + str2 + ... + 盐值 `3478cbbc33f84bd00d75d7dfa69e0daa`) |

---

## 认证机制

登录后返回 `token`，后续请求需在 Header 中携带：

| Header | 说明 |
|--------|------|
| `authorization` | 登录返回的 token |
| `userid` | 登录返回的 userId |
| `rolekey` | 登录返回的 roleKey（如 `student`） |
| `sign` | MD5 签名（部分接口需要） |
| `version` | `5.31.6` |

---

## 1. 滑块验证码（登录前）

### 1.1 获取验证码

```
POST session/captcha/v1/get
```

**请求体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `clientUid` | string | 客户端唯一标识（UUID 去横线） |
| `captchaType` | string | 固定值 `blockPuzzle` |

**响应体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | int | 200 成功 |
| `data.jigsawImageBase64` | string | 滑块图片 Base64 |
| `data.originalImageBase64` | string | 背景图 Base64 |
| `data.token` | string | 验证码 token |
| `data.secretKey` | string | 加密密钥 |

### 1.2 校验验证码

```
POST session/captcha/v1/check
```

**请求体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `pointJson` | string | AES 加密的滑块坐标数据 |
| `token` | string | 上一步获取的 token |
| `captchaType` | string | 固定值 `blockPuzzle` |

**响应体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | int | 200 成功，6111 失败需重试 |

---

## 2. 点选验证码（打卡时触发）

### 2.1 获取验证码

```
POST /attendence/clock/v1/get
```

**请求体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `clientUid` | string | 客户端唯一标识（UUID 去横线） |
| `captchaType` | string | 固定值 `clickWord` |

**响应体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `data.originalImageBase64` | string | 验证码图片 Base64 |
| `data.wordList` | list | 需要点选的文字列表 |
| `data.token` | string | 验证码 token |
| `data.secretKey` | string | 加密密钥 |

### 2.2 校验验证码

```
POST /attendence/clock/v1/check
```

**请求体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `pointJson` | string | AES 加密的点选坐标数据 |
| `token` | string | 上一步获取的 token |
| `captchaType` | string | 固定值 `clickWord` |

**响应体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | int | 200 成功，6111 失败需重试 |

---

## 3. 用户登录

```
POST session/user/v6/login
```

**请求体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `phone` | string | AES 加密的手机号 |
| `password` | string | AES 加密的密码 |
| `captcha` | string | 滑块验证码通过后的加密结果 |
| `loginType` | string | 固定值 `android` |
| `uuid` | string | UUID 去横线 |
| `device` | string | 固定值 `android` |
| `version` | string | `5.31.6` |
| `t` | string | AES 加密的当前时间戳（毫秒） |

**响应体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | int | 200 成功 |
| `data` | string | AES 加密的用户信息 JSON |

**解密后的 `data`：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `token` | string | 认证 token |
| `userId` | string | 用户 ID |
| `userType` | string | `student` 或 `teacher` |
| `roleKey` | string | 角色 key |
| `phone` | string | 手机号 |
| `nikeName` | string | 昵称 |
| `orgJson` | object | 学校/班级/专业信息 |
| `headImg` | string | 头像文件名 |

---

## 4. 获取实习计划

```
POST practice/plan/v3/getPlanByStu
```

**需要签名：** `sign = MD5(userId + roleKey + 盐值)`

**请求体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `pageSize` | int | 固定值 `999999` |
| `t` | string | AES 加密的当前时间戳（毫秒） |

**响应体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | int | 200 成功 |
| `data` | list | 实习计划列表 |

**`data[0]` 主要字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `planId` | string | 计划 ID（打卡时需要） |
| `startTime` | string | 实习开始时间 |
| `endTime` | string | 实习结束时间 |
| `postName` | string | 岗位名称 |
| `address` | string | 实习地址 |

---

## 5. 获取打卡记录

### 5.1 学生端

```
POST attendence/clock/v2/listSynchro
```

### 5.2 教师端

```
POST attendence/clock/teacher/v1/listSynchro
```

**请求体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `startTime` | string | 当月第一天 `YYYY-MM-DD HH:MM:SS` |
| `endTime` | string | 当月最后一天 `YYYY-MM-DD 00:00:00Z` |
| `t` | string | AES 加密的当前时间戳（毫秒） |

**响应体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | int | 200 成功 |
| `data` | list | 打卡记录列表（按时间倒序） |

**`data[0]` 主要字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | string | 打卡类型：`START` / `END` / `HOLIDAY` |
| `createTime` | string | 打卡时间 |
| `address` | string | 打卡地址 |
| `attendanceId` | string | 打卡记录 ID |

---

## 6. 提交打卡

### 6.1 学生端

```
POST attendence/clock/v6/save
```

**需要签名：** `sign = MD5(device + type + planId + userId + address + 盐值)`

### 6.2 教师端

```
POST attendence/clock/teacher/v2/save
```

**请求体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `type` | string | 打卡类型：`START` / `END` / `HOLIDAY` |
| `planId` | string | 实习计划 ID |
| `userId` | string | 用户 ID |
| `device` | object | 设备信息（品牌、型号、系统等） |
| `country` | string | 固定值 `中国` |
| `state` | string | 固定值 `NORMAL` |
| `createTime` | string | 当前时间 `YYYY-MM-DD HH:MM:SS` |
| `lastDetailAddress` | string | 上次打卡地址 |
| `description` | string | 备注 |
| `attachments` | null | 附件 |
| `longitude` | float | 经度 |
| `latitude` | float | 纬度 |
| `address` | string | 打卡地址 |
| `province` | string | 省份 |
| `city` | string | 城市 |
| `area` | string | 区县 |
| `t` | string | AES 加密的当前时间戳（毫秒） |
| `version` | string | `5.31.6` |
| `captcha` | string | 点选验证码通过后的加密结果（仅 msg=302 时携带） |

**响应体：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | int | 200 成功 |
| `msg` | string | 状态码 |
| `data` | object/null | 打卡记录 |

**`msg` 常见值：**

| msg | 含义 |
|-----|------|
| `success` | 打卡成功 |
| `302` | 触发点选验证码，需携带 `captcha` 重新请求 |
| `304` | 需要人脸认证，脚本无法处理 |

**成功时 `data` 示例：**

```json
{
  "createTime": "2026-07-01 18:57:51",
  "attendanceId": "9b9d0320c3b74056a6448a66e191d4f7"
}
```

---

## 7. 邮件通知（SMTP）

非 HTTP API，通过 SMTP_SSL 发送邮件通知打卡结果。

| 配置项 | 说明 |
|--------|------|
| `smtp.host` | SMTP 服务器地址 |
| `smtp.port` | SMTP 端口 |
| `smtp.username` | 发件邮箱账号 |
| `smtp.password` | 发件邮箱密码 |
| `smtp.from` | 发件人名称 |
| `smtp.to` | 收件人列表 |

---

## 打卡模式

| 模式 | 配置值 | 说明 |
|------|--------|------|
| 单次打卡 | `single` | 根据 `clockIn.type` 决定打 START/END/HOLIDAY |
| 一天两次 | `twice_daily` | 依次打 START（上班）+ END（下班） |

---

## 通用响应格式

```json
{
  "code": 200,
  "msg": "success",
  "data": {}
}
```

| code | 含义 |
|------|------|
| 200 | 请求成功 |
| 6111 | 验证码校验失败 |
| 其他 | 错误 |