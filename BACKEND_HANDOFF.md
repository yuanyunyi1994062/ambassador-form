# Ambassador 申请表 - 后端交接文档

## 概述
大使申请表单页面，用户填写后提交到后端存库。前端纯静态 HTML，已完成。

**前端预览**：https://yuanyunyi1994062.github.io/ambassador-form/  
**GitHub Repo**：https://github.com/yuanyunyi1994062/ambassador-form

---

## 后端需要做的事（共 3 步）

### 第 1 步：建表

```sql
CREATE TABLE ambassador_applications (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    tg_username VARCHAR(64) NOT NULL COMMENT 'Telegram 用户名',
    email VARCHAR(128) DEFAULT NULL COMMENT '备用联系邮箱（选填）',
    platforms JSON COMMENT '选中的平台列表 ["telegram","discord","reddit","x","onlyfans","other"]',
    platform_links JSON COMMENT '各平台链接/ID {"telegram":"xxx","discord":"xxx"}',
    other_platforms JSON COMMENT '自定义平台 [{"name":"Pornhub","link":"xxx"}]',
    followers VARCHAR(16) COMMENT '粉丝量级 <1K / 1-5K / 5-10K / 10-50K / 50K+',
    languages JSON COMMENT '语言列表 ["English","Spanish"]',
    methods JSON COMMENT '推广方式 ["Posts / Stories","Group Sharing"]',
    features JSON COMMENT '感兴趣的功能 ["AI Video","Sexy Chat"]',
    affiliate_exp BOOLEAN DEFAULT FALSE COMMENT '是否有 affiliate 经验',
    payout_method VARCHAR(32) COMMENT 'USDT / TG Stars / 其他',
    payout_other VARCHAR(128) DEFAULT NULL COMMENT '选了 Other 时填的具体方式',
    status ENUM('pending','approved','rejected') DEFAULT 'pending' COMMENT '审核状态',
    commission_rate DECIMAL(4,2) DEFAULT 20.00 COMMENT '返点比例',
    invite_code VARCHAR(32) DEFAULT NULL COMMENT '审核通过后分配的邀请码',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    reviewed_at TIMESTAMP NULL DEFAULT NULL,
    INDEX idx_status (status),
    INDEX idx_created (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='大使申请表';
```

### 第 2 步：写一个 POST 接口

**路径建议**：`POST /api/ambassador/apply`  
**Content-Type**：`application/json`  
**无需鉴权**（公开申请）

#### 请求体示例：

```json
{
    "tg_username": "joey_yuan",
    "email": "joey@example.com",
    "platforms": ["telegram", "discord", "reddit"],
    "platform_links": {
        "telegram": "https://t.me/joey_channel",
        "discord": "https://discord.gg/abc123",
        "reddit": "u/joey_yuan"
    },
    "other_platforms": [
        {"name": "Pornhub", "link": "https://pornhub.com/model/xxx"}
    ],
    "followers": "5-10K",
    "languages": ["English", "Chinese"],
    "methods": ["Posts / Stories", "Short Videos"],
    "features": ["AI Video", "Sexy Chat"],
    "affiliate_exp": true,
    "payout_method": "USDT",
    "payout_other": null
}
```

#### 成功响应：

```json
{
    "success": true,
    "message": "Application submitted successfully"
}
```

#### 失败响应：

```json
{
    "success": false,
    "message": "tg_username is required"
}
```

### 第 3 步：部署前端

把 `index.html` 放到公司能访问的地方（Nginx 挂路径、S3、随便哪）。

然后告诉我接口地址，我改一行代码就接上。

---

## 接口对接（前端改动）

前端 HTML 里目前的提交代码：

```js
fetch('YOUR_API_ENDPOINT', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(formData)
})
```

把 `YOUR_API_ENDPOINT` 替换成真实地址就行。

---

## CORS 注意

如果前端和后端不在同一个域名下，接口需要加 CORS header：

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: POST, OPTIONS
Access-Control-Allow-Headers: Content-Type
```

---

## 就这些，总结

| 步骤 | 工作量 |
|------|--------|
| 建表 | 5 分钟 |
| POST 接口 | 30 分钟 |
| 部署 HTML | 10 分钟 |
| **合计** | **< 1 小时** |
