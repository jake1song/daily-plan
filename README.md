# 📅 每日安排 · AI 自动生成

> Cherry Studio + DeepSeek → GitHub Actions → 腾讯云 COS → 手机直达

---

## 三步部署

### 第一步：腾讯云 COS 建站

1. 打开 [腾讯云 COS 控制台](https://console.cloud.tencent.com/cos)
2. **创建存储桶**：
   - 名称：`daily-schedule`（全球唯一，如有重复加后缀）
   - 地域：选离你近的（如 `广州`）
   - 访问权限：**公有读，私有写**
3. **开启静态网站**：
   - 进入存储桶 → 基础配置 → 静态网站 → 开启
   - 索引文档填 `index.html` → 保存
4. **绑定域名**：
   - 域名与传输管理 → 自定义源站域名 → 添加域名 `szk333333.fun`
   - 记下系统生成的 CNAME 值
5. **DNS 解析**：
   - 打开域名控制台 → 添加两条 CNAME 记录：
   - `www` → CNAME 值
   - `@` → CNAME 值
6. 等 5 分钟，访问 http://szk333333.fun 看到页面即成功

### 第二步：配置 GitHub 仓库

1. 点右上角绿色按钮 **Use this template** → **Create new repository**
2. 仓库名填 `daily-plan`，设为 **Public**
3. 添加 4 个 Secrets（Settings → Secrets and variables → Actions）：

| 名称 | 值 |
|------|-----|
| `TENCENT_SECRET_ID` | 你的腾讯云 SecretId |
| `TENCENT_SECRET_KEY` | 你的腾讯云 SecretKey |
| `BUCKET_NAME` | 存储桶名称（如 `daily-schedule-123456`）|
| `TENCENT_REGION` | 地域代码（如 `ap-guangzhou`）|

4. 手动测试：点 Actions → **Build and Deploy to Tencent COS** → **Run workflow**
5. 刷新 http://szk333333.fun 查看效果

### 第三步：Cherry Studio 配置

**1. 添加 DeepSeek 模型**
- 设置 → 模型服务 → 添加：
  - 名称：`DeepSeek`
  - API 地址：`https://api.deepseek.com/v1`
  - API Key：从 [platform.deepseek.com](https://platform.deepseek.com) 获取
  - 模型：`deepseek-v4-flash`

**2. 创建定时任务**
- 每天 `07:00` 触发
- 提示词：
  ```
  你是我的私人时间管家。请根据我的日常习惯（早起上午高效工作、下午处理杂项、傍晚健身、晚上阅读学习），为今天 {{date}} 制定一份详细的时间安排。

  输出格式（严格遵守）：
  - 只输出安排内容，不要任何寒暄
  - 分三个段落：
    ☕️ 上午 (08:00-12:00)
    ☀️ 下午 (12:00-18:00)
    🌙 晚上 (18:00-23:00)
  - 每个段落内用"· 时间 - 事项"逐条列出
  - 事项后面加一句简短的建议或鼓励
  ```
- 执行动作 → HTTP 请求：
  - **URL**：`https://api.github.com/repos/你的用户名/daily-plan/dispatches`
  - **Method**：`POST`
  - **Headers**：
    - `Authorization: token 你的GitHubToken`
    - `Content-Type: application/json`
  - **Body**：
    ```json
    {
      "event_type": "update_schedule",
      "client_payload": {
        "content": "{{output}}",
        "date": "{{date}}"
      }
    }
    ```

---

## 获取所需凭证

### GitHub Token
头像 → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token → 勾选 `repo` + `workflow` → 生成并复制

### 腾讯云密钥
控制台 → 访问管理 → 访问密钥 → API 密钥管理 → 新建密钥 → 复制 SecretId 和 SecretKey

---

## 项目结构

```
.github/workflows/deploy.yml   # GitHub Actions 工作流
index.html                      # 占位首页
README.md                       # 本说明书
```

---

部署过程有问题随时告诉我，每一步的报错或截图发来，我帮你排查。
