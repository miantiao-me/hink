# 🥷🏻 hink

Link Shortener for Hackers.

English | [简体中文](#原理)

----

## How It Works

The principle is to use the hash of a Git empty commit as the unique identifier for a short link, storing the original long link in the commit message. When the short link is accessed, the system retrieves the long link by requesting the corresponding `.patch` file from GitHub and performs a redirect. Combined with the analytics dashboard of a WAF (Web Application Firewall), this creates a link shortening service with access statistics.

Currently, this solution has been tested and works on the following platforms:

- Cloudflare Workers/Snippets + Cloudflare WAF (Pro)
- Tencent EdgeOne (Free)
- Alibaba Cloud ESA (Free)

## Code

### ES Module Script Worker

Suitable for Cloudflare Workers/Snippets and Alibaba Cloud ESA.

```js
const GIT_REPO = "https://github.com/ccbikai/hink"
export default {
  async fetch(request) {
    const { pathname } = new URL(request.url)
    const gitPatch = `${GIT_REPO}/commit${pathname}.patch`
    const patch = await fetch(gitPatch, { cf: { cacheEverything: true, cacheTtlByStatus: { '200-299': 86400 } }}).then(res => res.text())
    const url = pathname === '/' ? GIT_REPO : patch.match(/^Subject:\s*\[PATCH\](.*)$/m)?.[1]?.trim()
    return Response.redirect(url || GIT_REPO)
  }
}
```

### Classic Script Worker

Suitable for Tencent EdgeOne.

```js
const GIT_REPO = "https://github.com/ccbikai/hink"
addEventListener("fetch", async (event) => {
  const { pathname } = new URL(event.request.url)
  const gitPatch = `${GIT_REPO}/commit${pathname}.patch`
  const patch = await fetch(gitPatch).then(res => res.text())
  const url = pathname === '/' ? GIT_REPO : patch.match(/^Subject:\s*\[PATCH\](.*)$/m)?.[1]?.trim()
  event.respondWith(new Response(null, { status: 302, headers: { Location: url || GIT_REPO } }))
});
```

## How to Use

### Create a Repository

1. Initialize a new Git repository using `git init`.
2. Create your first short link with `git commit --allow-empty -m "https://github.com/ccbikai/hink"`. After committing, you will get a commit hash (e.g., `1f2956e`). Remember to `git push` your changes.
3. Copy the code above, replace the `GIT_REPO` variable with your repository address, then deploy the function on the corresponding serverless platform and bind your domain.
4. You can now access the short link using the format `https://{your-domain}/{commit-hash}`, for example: <https://link.agi.li/1f2956e>.
5. You can view access statistics through your WAF's management panel:

    - **Cloudflare (Pro):** `Zone` -> `Analytics & Logs` -> `HTTP Traffic` -> `Add filter (domain:your.domain)`
    - **Tencent EdgeOne:** `EdgeOne` -> `Metrics` -> `L7 Access Requests` -> `Add Filter (Domain: your domain)`
    - **Alibaba Cloud ESA:** `ESA Console` -> `Websites` -> `Traffic and Log` -> `Traffic Analysis` -> `Total Requests` -> `Add Filter (Domain: your domain)`

### Advanced Usage

#### API

- Create a `.github/api` file (content can be anything) as the target for API updates.
- Use GitHub REST "Create or update file contents" to update that file. Each update creates a commit; put the long URL in the `message` field.
- The `content` field must be Base64, and updating an existing file requires the `sha` field.
- The short link remains `https://{your-domain}/{commit-sha}`.

Docs: <https://docs.github.com/en/rest/repos/contents?apiVersion=2022-11-28#create-or-update-file-contents>

```bash
curl -X PUT \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/OWNER/REPO/contents/.github/api" \
  -d '{
    "message": "https://example.com/long-url",
    "content": "cGluZwo=",
    "sha": "FILE_SHA",
    "branch": "master"
  }'
```

#### Custom Slug

- After the commit is created, use <https://github.com/not-an-aardvark/lucky-commit> to rewrite the last commit and customize the short slug.
- This tool only supports hex characters (0-9, a-f), so the slug is hex-only.

## Demos

### Cloudflare

![Cloudflare](https://github.com/user-attachments/assets/379befe9-b757-4112-a9ca-48cf24fa997e)

### Alibaba Cloud ESA

![Alibaba](https://github.com/user-attachments/assets/9b6f9de8-4895-47da-903e-7671c521d9e4)

### Tencent Cloud EdgeOne

![Tencent](https://github.com/user-attachments/assets/cabb9767-0182-4457-859e-006d540c4457)

----

## 原理

利用 Git 的空提交（empty commit）哈希值作为短链接的唯一标识符，并将原始的长链接存储在提交信息（Commit Message）中。当访问短链接时，系统通过请求对应哈希值的 GitHub `.patch` 文件来提取长链接，并进行重定向。结合 WAF（Web 应用防火墙）类工具的分析面板，即可实现一个支持访问统计的短链接服务。

目前，该方案已在以下平台测试通过：

- Cloudflare Workers/Snippets + Cloudflare WAF(Pro)
- Tencent EdgeOne(Free)
- Alibaba Cloud ESA(Free)

## 代码

### ES Worker

适用于 Cloudflare Workers/Snippets、Alibaba Cloud ESA。

```js
const GIT_REPO = "https://github.com/ccbikai/hink"
export default {
  async fetch(request) {
    const { pathname } = new URL(request.url)
    const gitPatch = `${GIT_REPO}/commit${pathname}.patch`
    const patch = await fetch(gitPatch, { cf: { cacheEverything: true, cacheTtlByStatus: { '200-299': 86400 } }}).then(res => res.text())
    const url = pathname === '/' ? GIT_REPO : patch.match(/^Subject:\s*\[PATCH\](.*)$/m)?.[1]?.trim()
    return Response.redirect(url || GIT_REPO)
  }
}
```

### 传统 Worker

适用于 Tencent EdgeOne。

```js
const GIT_REPO = "https://github.com/ccbikai/hink"
addEventListener("fetch", async (event) => {
  const { pathname } = new URL(event.request.url)
  const gitPatch = `${GIT_REPO}/commit${pathname}.patch`
  const patch = await fetch(gitPatch).then(res => res.text())
  const url = pathname === '/' ? GIT_REPO : patch.match(/^Subject:\s*\[PATCH\](.*)$/m)?.[1]?.trim()
  event.respondWith(new Response(null, { status: 302, headers: { Location: url || GIT_REPO } }))
});
```

## 使用方式

### 创建存储库

1. 使用 `git init` 初始化一个新的 Git 仓库。
2. 使用 `git commit --allow-empty -m "https://github.com/ccbikai/hink"` 创建第一个短链接。提交后，你将获得一个 Commit Hash（例如 `1f2956e`）。请记得执行 `git push` 将变更推送到远程仓库。
3. 复制上方的代码，将 `GIT_REPO` 变量替换为你的仓库地址，然后在相应的 Serverless 平台部署函数，并绑定你的域名。
4. 现在，你可以通过 `https://{你的域名}/{Commit-Hash}` 的格式来访问短链接，例如：<https://link.agi.li/1f2956e>。
5. 你可以通过 WAF 的管理面板查看访问统计：

- Cloudflare(Pro): `域名` -> `分析和日志` -> `HTTP 流量` -> `添加筛选（域名：你的域名）`
- Tencent EdgeOne: `EdgeOne` -> `指标分析` -> `L7 访问请求数` -> `添加筛选（域名：你的域名）`
- Alibaba Cloud ESA: `ESA 控制台` -> `站点管理` -> `流量和日志分析` -> `流量分析` -> `总请求数` -> `添加筛选（域名：你的域名）`

### 高级用法

#### API

- 创建一个 `.github/api` 文件（内容可随意），作为 API 更新的目标文件。
- 通过 GitHub REST 的“Create or update file contents”接口更新该文件，每次更新都会生成一个 commit；把长链接写到 `message` 字段。
- `content` 字段需要 Base64，更新已有文件时需要提供 `sha` 字段。
- 短链接仍然是 `https://{你的域名}/{commit-sha}`。

参考文档：<https://docs.github.com/en/rest/repos/contents?apiVersion=2022-11-28#create-or-update-file-contents>

```bash
curl -X PUT \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/OWNER/REPO/contents/.github/api" \
  -d '{
    "message": "https://example.com/long-url",
    "content": "cGluZwo=",
    "sha": "FILE_SHA",
    "branch": "master"
  }'
```

#### 自定义 Slug

- 在 commit 生成后，使用 <https://github.com/not-an-aardvark/lucky-commit> 修改最后一次 commit，从而定制短链 slug。
- 该工具只能生成 hex 字符（0-9、a-f），因此 slug 仅支持 hex。

## 演示

### Cloudflare Workers

![Cloudflare](https://github.com/user-attachments/assets/379befe9-b757-4112-a9ca-48cf24fa997e)

### 阿里云 ESA

![Alibaba](https://github.com/user-attachments/assets/9b6f9de8-4895-47da-903e-7671c521d9e4)

### 腾讯云 EdgeOne

![Tencent](https://github.com/user-attachments/assets/cabb9767-0182-4457-859e-006d540c4457)
