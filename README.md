# MoonTV

<div align="center">
  <img src="public/logo.png" alt="MoonTV Logo" width="120">
</div>

> 🎬 **MoonTV** 是一个开箱即用的、跨平台的影视聚合播放器。它基于 **Next.js 14** + **Tailwind&nbsp;CSS** + **TypeScript** 构建，支持多资源搜索、在线播放、收藏同步、播放记录、云端存储，让你可以随时随地畅享海量免费影视内容。

<div align="center">

![Next.js](https://img.shields.io/badge/Next.js-14-000?logo=nextdotjs)
![TailwindCSS](https://img.shields.io/badge/TailwindCSS-3-38bdf8?logo=tailwindcss)
![TypeScript](https://img.shields.io/badge/TypeScript-4.x-3178c6?logo=typescript)
![License](https://img.shields.io/badge/License-MIT-green)
![Docker Ready](https://img.shields.io/badge/Docker-ready-blue?logo=docker)

</div>

---

## ✨ 功能特性

- 🔍 **多源聚合搜索**：一次搜索立刻返回全源结果。
- 📄 **丰富详情页**：支持剧集列表、演员、年份、简介等完整信息展示。
- ▶️ **流畅在线播放**：集成 HLS.js & ArtPlayer。
- ❤️ **收藏 + 继续观看**：支持 Kvrocks/Redis/Upstash 存储，多端同步进度。
- 📱 **PWA**：离线缓存、安装到桌面/主屏，移动端原生体验。
- 🌗 **响应式布局**：桌面侧边栏 + 移动底部导航，自适应各种屏幕尺寸。
- 👿 **智能去广告**：自动跳过视频中的切片广告（实验性）。

### 注意：部署后项目为空壳项目，无内置播放源和直播源，需要自行收集

<details>
  <summary>点击查看项目截图</summary>
  <img src="public/screenshot1.png" alt="项目截图" style="max-width:600px">
  <img src="public/screenshot2.png" alt="项目截图" style="max-width:600px">
  <img src="public/screenshot3.png" alt="项目截图" style="max-width:600px">
</details>

### 请不要在 B站、小红书、微信公众号、抖音、今日头条或其他中国大陆社交平台发布视频或文章宣传本项目，不授权任何“科技周刊/月刊”类项目或站点收录本项目。

## 🗺 目录

- [技术栈](#技术栈)
- [部署](#部署)
  - [一键部署](#zeabur-一键部署)
  - [Docker 部署](#Kvrocks-存储推荐)
- [配置文件](#配置文件)
- [订阅](#订阅)
- [自动更新](#自动更新)
- [环境变量](#环境变量)
- [客户端](#客户端)
- [AndroidTV 使用](#AndroidTV-使用)
- [Roadmap](#roadmap)
- [安全与隐私提醒](#安全与隐私提醒)
- [License](#license)
- [致谢](#致谢)

## 技术栈

| 分类      | 主要依赖                                                                                              |
| --------- | ----------------------------------------------------------------------------------------------------- |
| 前端框架  | [Next.js 14](https://nextjs.org/) · App Router                                                        |
| UI & 样式 | [Tailwind&nbsp;CSS 3](https://tailwindcss.com/)                                                       |
| 语言      | TypeScript 4                                                                                          |
| 播放器    | [ArtPlayer](https://github.com/zhw2590582/ArtPlayer) · [HLS.js](https://github.com/video-dev/hls.js/) |
| 代码质量  | ESLint · Prettier · Jest                                                                              |
| 部署      | Docker                                                                    |

## 部署

本项目**仅支持 Docker 或其他基于 Docker 的平台** 部署。

### zeabur 一键部署

点击下方按钮即可一键部署，自动配置 LunaTV + Kvrocks 数据库：

[![Deploy on Zeabur](https://zeabur.com/button.svg)](https://zeabur.com/templates/8MPTQU/deploy)

**优势**：
- ✅ 无需配置，一键启动（自动部署完整环境）
- ✅ 自动 HTTPS 和全球 CDN 加速
- ✅ 持久化存储，数据永不丢失
- ✅ 免费额度足够个人使用

**⚠️ 重要提示**：部署完成后，需要在 Zeabur 中为 LunaTV 服务设置访问域名（Domain）才能在浏览器中访问。详见下方 [设置访问域名](#5-设置访问域名必须) 步骤。

### Kvrocks 存储（推荐）

```yml
services:
  moontv-core:
    image: ghcr.io/moontechlab/lunatv:latest
    container_name: moontv-core
    restart: on-failure
    ports:
      - '3000:3000'
    environment:
      - USERNAME=admin
      - PASSWORD=admin_password
      - NEXT_PUBLIC_STORAGE_TYPE=kvrocks
      - KVROCKS_URL=redis://moontv-kvrocks:6666
    networks:
      - moontv-network
    depends_on:
      - moontv-kvrocks
  moontv-kvrocks:
    image: apache/kvrocks
    container_name: moontv-kvrocks
    restart: unless-stopped
    volumes:
      - kvrocks-data:/var/lib/kvrocks
    networks:
      - moontv-network
networks:
  moontv-network:
    driver: bridge
volumes:
  kvrocks-data:
```

### Redis 存储（有一定的丢数据风险）

```yml
services:
  moontv-core:
    image: ghcr.io/moontechlab/lunatv:latest
    container_name: moontv-core
    restart: on-failure
    ports:
      - '3000:3000'
    environment:
      - USERNAME=admin
      - PASSWORD=admin_password
      - NEXT_PUBLIC_STORAGE_TYPE=redis
      - REDIS_URL=redis://moontv-redis:6379
    networks:
      - moontv-network
    depends_on:
      - moontv-redis
  moontv-redis:
    image: redis:alpine
    container_name: moontv-redis
    restart: unless-stopped
    networks:
      - moontv-network
    # 请开启持久化，否则升级/重启后数据丢失
    volumes:
      - ./data:/data
networks:
  moontv-network:
    driver: bridge
```

### Upstash 存储

1. 在 [upstash](https://upstash.com/) 注册账号并新建一个 Redis 实例，名称任意。
2. 复制新数据库的 **HTTPS ENDPOINT 和 TOKEN**
3. 使用如下 docker compose
```yml
services:
  moontv-core:
    image: ghcr.io/moontechlab/lunatv:latest
    container_name: moontv-core
    restart: on-failure
    ports:
      - '3000:3000'
    environment:
      - USERNAME=admin
      - PASSWORD=admin_password
      - NEXT_PUBLIC_STORAGE_TYPE=upstash
      - UPSTASH_URL=上面 https 开头的 HTTPS ENDPOINT
      - UPSTASH_TOKEN=上面的 TOKEN
```

### ☁️ Zeabur 部署（推荐）

Thanks to @SzeMeng76

Zeabur 是一站式云端部署平台，使用预构建的 Docker 镜像可以快速部署，无需等待构建。

**部署步骤：**

1. **添加 KVRocks 服务**（先添加数据库）
   - 点击 "Add Service" > "Docker Images"
   - 输入镜像名称：`apache/kvrocks`
   - 配置端口：`6666` (TCP)
   - **记住服务名称**（通常是 `apachekvrocks`）
   - **配置持久化卷（重要）**：
     * 在服务设置中找到 "Volumes" 部分
     * 点击 "Add Volume" 添加新卷
     * Volume ID: `kvrocks-data`（可自定义，仅支持字母、数字、连字符）
     * Path: `/var/lib/kvrocks/db`
     * 保存配置

   > 💡 **重要提示**：持久化卷路径必须设置为 `/var/lib/kvrocks/db`（KVRocks 数据目录），这样配置文件保留在容器内，数据库文件持久化，重启后数据不会丢失！

2. **添加 LunaTV 服务**
   - 点击 "Add Service" > "Docker Images"
   - 输入镜像名称：`ghcr.io/moontechlab/lunatv:latest`
   - 配置端口：`3000` (HTTP)

3. **配置环境变量**

   在 LunaTV 服务的环境变量中添加：

   ```env
   # 必填：管理员账号
   USERNAME=admin
   PASSWORD=your_secure_password

   # 必填：存储配置
   NEXT_PUBLIC_STORAGE_TYPE=kvrocks
   KVROCKS_URL=redis://apachekvrocks:6666

   # 可选：站点配置
   SITE_BASE=https://your-domain.zeabur.app
   NEXT_PUBLIC_SITE_NAME=LunaTV Enhanced
   ANNOUNCEMENT=欢迎使用 LunaTV Enhanced Edition

   # 可选：豆瓣代理配置（推荐）
   NEXT_PUBLIC_DOUBAN_PROXY_TYPE=cmliussss-cdn-tencent
   NEXT_PUBLIC_DOUBAN_IMAGE_PROXY_TYPE=cmliussss-cdn-tencent
   ```

   **注意**：
   - 使用服务名称作为主机名：`redis://apachekvrocks:6666`
   - 如果服务名称不同，请替换为实际名称
   - 两个服务必须在同一个 Project 中

4. **部署完成**
   - Zeabur 会自动拉取镜像并启动服务
   - 等待服务就绪后，需要手动设置访问域名（见下一步）

#### 5. 设置访问域名（必须）

   - 在 LunaTV 服务页面，点击 "Networking" 或 "网络" 标签
   - 点击 "Generate Domain" 生成 Zeabur 提供的免费域名（如 `xxx.zeabur.app`）
   - 或者绑定自定义域名：
     * 点击 "Add Domain" 添加你的域名
     * 按照提示配置 DNS CNAME 记录指向 Zeabur 提供的目标地址
   - 设置完域名后即可通过域名访问 LunaTV

6. **绑定自定义域名（可选）**
   - 在服务设置中点击 "Domains"
   - 添加你的自定义域名
   - 配置 DNS CNAME 记录指向 Zeabur 提供的域名

#### 🔄 更新 Docker 镜像

当 Docker 镜像有新版本发布时，Zeabur 不会自动更新。需要手动触发更新。

**更新步骤：**

1. **进入服务页面**
   - 点击需要更新的服务（LunaTV 或 KVRocks）

2. **重启服务**
   - 点击 **"服务状态"** 页面，再点击 **"重启当前版本"** 按钮
   - Zeabur 会自动拉取最新的 `latest` 镜像并重新部署

> 💡 **提示**：
> - 使用 `latest` 标签时，Restart 会自动拉取最新镜像
> - 生产环境推荐使用固定版本标签（如 `v5.5.6`）避免意外更新

## 配置文件

完成部署后为空壳应用，无播放源，需要站长在管理后台的配置文件设置中填写配置文件（后续会支持订阅）

配置文件示例如下：

```json
{
  "cache_time": 7200,
  "api_site": {
    "dyttzy": {
      "api": "http://xxx.com/api.php/provide/vod",
      "name": "示例资源",
      "detail": "http://xxx.com"
    }
    // ...更多站点
  },
  "custom_category": [
    {
      "name": "华语",
      "type": "movie",
      "query": "华语"
    }
  ]
}
```

- `cache_time`：接口缓存时间（秒）。
- `api_site`：你可以增删或替换任何资源站，字段说明：
  - `key`：唯一标识，保持小写字母/数字。
  - `api`：资源站提供的 `vod` JSON API 根地址。
  - `name`：在人机界面中展示的名称。
  - `detail`：（可选）部分无法通过 API 获取剧集详情的站点，需要提供网页详情根 URL，用于爬取。
- `custom_category`：自定义分类配置，用于在导航中添加个性化的影视分类。以 type + query 作为唯一标识。支持以下字段：
  - `name`：分类显示名称（可选，如不提供则使用 query 作为显示名）
  - `type`：分类类型，支持 `movie`（电影）或 `tv`（电视剧）
  - `query`：搜索关键词，用于在豆瓣 API 中搜索相关内容

custom_category 支持的自定义分类已知如下：

- movie：热门、最新、经典、豆瓣高分、冷门佳片、华语、欧美、韩国、日本、动作、喜剧、爱情、科幻、悬疑、恐怖、治愈
- tv：热门、美剧、英剧、韩剧、日剧、国产剧、港剧、日本动画、综艺、纪录片

也可输入如 "哈利波特" 效果等同于豆瓣搜索

MoonTV 支持标准的苹果 CMS V10 API 格式。

## 订阅

将完整的配置文件 base58 编码后提供 http 服务即为订阅链接，可在 MoonTV 后台/Helios 中使用。

## 自动更新

可借助 [watchtower](https://github.com/containrrr/watchtower) 自动更新镜像容器

dockge/komodo 等 docker compose UI 也有自动更新功能

## 环境变量

| 变量                                | 说明                                         | 可选值                           | 默认值                                                                                                                     |
| ----------------------------------- | -------------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| USERNAME                            | 站长账号           | 任意字符串                       | 无默认，必填字段                                                                                                                     |
| PASSWORD                            | 站长密码           | 任意字符串                       | 无默认，必填字段                                                                                                                     |
| SITE_BASE                           | 站点 url              |       形如 https://example.com                  | 空                                                                                                                     |
| NEXT_PUBLIC_SITE_NAME               | 站点名称                                     | 任意字符串                       | MoonTV                                                                                                                     |
| ANNOUNCEMENT                        | 站点公告                                     | 任意字符串                       | 本网站仅提供影视信息搜索服务，所有内容均来自第三方网站。本站不存储任何视频资源，不对任何内容的准确性、合法性、完整性负责。 |
| NEXT_PUBLIC_STORAGE_TYPE            | 播放记录/收藏的存储方式                      | redis、kvrocks、upstash | 无默认，必填字段                                                                                                               |
| KVROCKS_URL                           | kvrocks 连接 url                               | 连接 url                         | 空                                                                                                                         |
| REDIS_URL                           | redis 连接 url                               | 连接 url                         | 空                                                                                                                         |
| UPSTASH_URL                         | upstash redis 连接 url                       | 连接 url                         | 空                                                                                                                         |
| UPSTASH_TOKEN                       | upstash redis 连接 token                     | 连接 token                       | 空                                                                                                                         |
| NEXT_PUBLIC_SEARCH_MAX_PAGE         | 搜索接口可拉取的最大页数                     | 1-50                             | 5                                                                                                                          |
| NEXT_PUBLIC_DOUBAN_PROXY_TYPE       | 豆瓣数据源请求方式                           | 见下方                           | direct                                                                                                                     |
| NEXT_PUBLIC_DOUBAN_PROXY            | 自定义豆瓣数据代理 URL                       | url prefix                       | (空)                                                                                                                       |
| NEXT_PUBLIC_DOUBAN_IMAGE_PROXY_TYPE | 豆瓣图片代理类型                             | 见下方                           | direct                                                                                                                     |
| NEXT_PUBLIC_DOUBAN_IMAGE_PROXY      | 自定义豆瓣图片代理 URL                       | url prefix                       | (空)                                                                                                                       |
| NEXT_PUBLIC_DISABLE_YELLOW_FILTER   | 关闭色情内容过滤                             | true/false                       | false                                                                                                                      |
| NEXT_PUBLIC_FLUID_SEARCH | 是否开启搜索接口流式输出 | true/ false | true |

NEXT_PUBLIC_DOUBAN_PROXY_TYPE 选项解释：

- direct: 由服务器直接请求豆瓣源站
- cors-proxy-zwei: 浏览器向 cors proxy 请求豆瓣数据，该 cors proxy 由 [Zwei](https://github.com/bestzwei) 搭建
- cmliussss-cdn-tencent: 浏览器向豆瓣 CDN 请求数据，该 CDN 由 [CMLiussss](https://github.com/cmliu) 搭建，并由腾讯云 cdn 提供加速
- cmliussss-cdn-ali: 浏览器向豆瓣 CDN 请求数据，该 CDN 由 [CMLiussss](https://github.com/cmliu) 搭建，并由阿里云 cdn 提供加速
- custom: 用户自定义 proxy，由 NEXT_PUBLIC_DOUBAN_PROXY 定义

NEXT_PUBLIC_DOUBAN_IMAGE_PROXY_TYPE 选项解释：

- direct：由浏览器直接请求豆瓣分配的默认图片域名
- server：由服务器代理请求豆瓣分配的默认图片域名
- img3：由浏览器请求豆瓣官方的精品 cdn（阿里云）
- cmliussss-cdn-tencent：由浏览器请求豆瓣 CDN，该 CDN 由 [CMLiussss](https://github.com/cmliu) 搭建，并由腾讯云 cdn 提供加速
- cmliussss-cdn-ali：由浏览器请求豆瓣 CDN，该 CDN 由 [CMLiussss](https://github.com/cmliu) 搭建，并由阿里云 cdn 提供加速
- custom: 用户自定义 proxy，由 NEXT_PUBLIC_DOUBAN_IMAGE_PROXY 定义

## 客户端

v100.0.0 以上版本可配合 [Selene](https://github.com/MoonTechLab/Selene) 使用，移动端体验更加友好，数据完全同步

## AndroidTV 使用

目前该项目可以配合 [OrionTV](https://github.com/zimplexing/OrionTV) 在 Android TV 上使用，可以直接作为 OrionTV 后端

已实现播放记录和网页端同步

## 安全与隐私提醒

### 请设置密码保护并关闭公网注册

为了您的安全和避免潜在的法律风险，我们要求在部署时**强烈建议关闭公网注册**：

### 部署要求

1. **设置环境变量 `PASSWORD`**：为您的实例设置一个强密码
2. **仅供个人使用**：请勿将您的实例链接公开分享或传播
3. **遵守当地法律**：请确保您的使用行为符合当地法律法规
4. 
### 配置视频源json
```json
{
"cache_time": 9200,
"api_site": {
"api_1": {
"name": "TV-1080资源",
"api": "https://api.1080zyku.com/inc/api_mac10.php",
"detail": "https://api.1080zyku.com"
},
"api_2": {
"name": "AV-155资源",
"api": "https://155api.com/api.php/provide/vod",
"detail": "https://155api.com"
},
"api_3": {
"name": "TV-360资源",
"api": "https://360zy.com/api.php/provide/vod",
"detail": "https://360zy.com"
},
"api_4": {
"name": "TV-CK资源",
"api": "https://ckzy.me/api.php/provide/vod",
"detail": "https://ckzy.me"
},
"api_5": {
"name": "TV-U酷资源",
"api": "https://api.ukuapi.com/api.php/provide/vod",
"detail": "https://api.ukuapi.com"
},
"api_6": {
"name": "TV-U酷资源",
"api": "https://api.ukuapi88.com/api.php/provide/vod",
"detail": "https://api.ukuapi88.com"
},
"api_7": {
"name": "TV-ikun资源",
"api": "https://ikunzyapi.com/api.php/provide/vod",
"detail": "https://ikunzyapi.com"
},
"api_8": {
"name": "TV-wujinapi无尽",
"api": "https://api.wujinapi.cc/api.php/provide/vod",
"detail": ""
},
"api_9": {
"name": "TV-丫丫点播",
"api": "https://cj.yayazy.net/api.php/provide/vod",
"detail": "https://cj.yayazy.net"
},
"api_10": {
"name": "TV-光速资源",
"api": "https://api.guangsuapi.com/api.php/provide/vod",
"detail": "https://api.guangsuapi.com"
},
"api_11": {
"name": "TV-卧龙点播",
"api": "https://collect.wolongzyw.com/api.php/provide/vod",
"detail": "https://collect.wolongzyw.com"
},
"api_12": {
"name": "TV-卧龙资源",
"api": "https://collect.wolongzy.cc/api.php/provide/vod",
"detail": ""
},
"api_13": {
"name": "TV-卧龙资源",
"api": "https://wolongzyw.com/api.php/provide/vod",
"detail": "https://wolongzyw.com"
},
"api_14": {
"name": "TV-天涯资源",
"api": "https://tyyszy.com/api.php/provide/vod",
"detail": "https://tyyszy.com"
},
"api_15": {
"name": "TV-如意资源",
"api": "https://cj.rycjapi.com/api.php/provide/vod",
"detail": ""
},
"api_16": {
"name": "TV-小猫咪资源",
"api": "https://zy.xmm.hk/api.php/provide/vod",
"detail": "https://zy.xmm.hk"
},
"api_17": {
"name": "TV-新浪点播",
"api": "https://api.xinlangapi.com/xinlangapi.php/provide/vod",
"detail": "https://api.xinlangapi.com"
},
"api_18": {
"name": "TV-无尽资源",
"api": "https://api.wujinapi.com/api.php/provide/vod",
"detail": ""
},
"api_19": {
"name": "TV-无尽资源",
"api": "https://api.wujinapi.me/api.php/provide/vod",
"detail": ""
},
"api_20": {
"name": "TV-无尽资源",
"api": "https://api.wujinapi.net/api.php/provide/vod",
"detail": ""
},
"api_21": {
"name": "TV-旺旺短剧",
"api": "https://wwzy.tv/api.php/provide/vod",
"detail": "https://wwzy.tv"
},
"api_22": {
"name": "TV-旺旺资源",
"api": "https://api.wwzy.tv/api.php/provide/vod",
"detail": "https://api.wwzy.tv"
},
"api_23": {
"name": "TV-暴风资源",
"api": "https://bfzyapi.com/api.php/provide/vod",
"detail": ""
},
"api_24": {
"name": "TV-最大点播",
"api": "http://zuidazy.me/api.php/provide/vod",
"detail": "http://zuidazy.me"
},
"api_25": {
"name": "TV-最大资源",
"api": "https://api.zuidapi.com/api.php/provide/vod",
"detail": "https://api.zuidapi.com"
},
"api_26": {
"name": "TV-樱花资源",
"api": "https://m3u8.apiyhzy.com/api.php/provide/vod",
"detail": ""
},
"api_27": {
"name": "TV-步步高资源",
"api": "https://api.yparse.com/api/json",
"detail": ""
},
"api_28": {
"name": "TV-牛牛点播",
"api": "https://api.niuniuzy.me/api.php/provide/vod",
"detail": "https://api.niuniuzy.me"
},
"api_29": {
"name": "TV-电影天堂资源",
"api": "http://caiji.dyttzyapi.com/api.php/provide/vod",
"detail": "http://caiji.dyttzyapi.com"
},
"api_30": {
"name": "AV-百万资源",
"api": "https://api.bwzyz.com/api.php/provide/vod",
"detail": "https://api.bwzyz.com"
},
"api_31": {
"name": "TV-百度云资源",
"api": "https://api.apibdzy.com/api.php/provide/vod",
"detail": "https://api.apibdzy.com"
},
"api_32": {
"name": "TV-神马云",
"api": "https://api.1080zyku.com/inc/apijson.php/",
"detail": "https://api.1080zyku.com"
},
"api_33": {
"name": "TV-索尼资源",
"api": "https://suoniapi.com/api.php/provide/vod",
"detail": ""
},
"api_34": {
"name": "TV-红牛资源",
"api": "https://www.hongniuzy2.com/api.php/provide/vod",
"detail": "https://www.hongniuzy2.com"
},
"api_35": {
"name": "TV-茅台资源",
"api": "https://caiji.maotaizy.cc/api.php/provide/vod",
"detail": "https://caiji.maotaizy.cc"
},
"api_36": {
"name": "TV-虎牙资源",
"api": "https://www.huyaapi.com/api.php/provide/vod",
"detail": "https://www.huyaapi.com"
},
"api_37": {
"name": "TV-豆瓣资源",
"api": "https://caiji.dbzy.tv/api.php/provide/vod",
"detail": "https://caiji.dbzy.tv"
},
"api_38": {
"name": "TV-豆瓣资源",
"api": "https://dbzy.tv/api.php/provide/vod",
"detail": "https://dbzy.tv"
},
"api_39": {
"name": "TV-豪华资源",
"api": "https://hhzyapi.com/api.php/provide/vod",
"detail": "https://hhzyapi.com"
},
"api_40": {
"name": "TV-速博资源",
"api": "https://subocaiji.com/api.php/provide/vod",
"detail": ""
},
"api_41": {
"name": "TV-量子资源",
"api": "https://cj.lziapi.com/api.php/provide/vod",
"detail": ""
},
"api_42": {
"name": "TV-金鹰点播",
"api": "https://jinyingzy.com/api.php/provide/vod",
"detail": "https://jinyingzy.com"
},
"api_43": {
"name": "TV-金鹰资源",
"api": "https://jyzyapi.com/api.php/provide/vod",
"detail": "https://jyzyapi.com"
},
"api_44": {
"name": "TV-閃電资源",
"api": "https://sdzyapi.com/api.php/provide/vod",
"detail": "https://sdzyapi.com"
},
"api_45": {
"name": "TV-非凡资源",
"api": "https://cj.ffzyapi.com/api.php/provide/vod",
"detail": "https://cj.ffzyapi.com"
},
"api_46": {
"name": "TV-飘零资源",
"api": "https://p2100.net/api.php/provide/vod",
"detail": "https://p2100.net"
},
"api_47": {
"name": "TV-魔爪资源",
"api": "https://mozhuazy.com/api.php/provide/vod",
"detail": "https://mozhuazy.com"
},
"api_48": {
"name": "TV-魔都动漫",
"api": "https://caiji.moduapi.cc/api.php/provide/vod",
"detail": "https://caiji.moduapi.cc"
},
"api_49": {
"name": "TV-魔都资源",
"api": "https://www.mdzyapi.com/api.php/provide/vod",
"detail": "https://www.mdzyapi.com"
},
"api_50": {
"name": "TV-黑木耳",
"api": "https://json.heimuer.xyz/api.php/provide/vod",
"detail": "https://json.heimuer.xyz"
},
"api_51": {
"name": "TV-黑木耳点播",
"api": "https://json02.heimuer.xyz/api.php/provide/vod",
"detail": "https://json02.heimuer.xyz"
},
"api_52": {
"name": "AV-91麻豆",
"api": "https://91md.me/api.php/provide/vod",
"detail": "https://91md.me"
},
"api_53": {
"name": "AV-AIvin",
"api": "http://lbapiby.com/api.php/provide/vod",
"detail": ""
},
"api_54": {
"name": "AV-JKUN资源",
"api": "https://jkunzyapi.com/api.php/provide/vod",
"detail": "https://jkunzyapi.com"
},
"api_55": {
"name": "AV-souav资源",
"api": "https://api.souavzy.vip/api.php/provide/vod",
"detail": "https://api.souavzy.vip"
},
"api_56": {
"name": "AV-乐播资源",
"api": "https://lbapi9.com/api.php/provide/vod",
"detail": ""
},
"api_57": {
"name": "AV-奥斯卡资源",
"api": "https://aosikazy.com/api.php/provide/vod",
"detail": "https://aosikazy.com"
},
"api_58": {
"name": "AV-奶香香",
"api": "https://Naixxzy.com/api.php/provide/vod",
"detail": "https://Naixxzy.com"
},
"api_59": {
"name": "AV-森林资源",
"api": "https://slapibf.com/api.php/provide/vod",
"detail": "https://slapibf.com"
},
"api_60": {
"name": "AV-淫水机资源",
"api": "https://www.xrbsp.com/api/json.php",
"detail": "https://www.xrbsp.com"
},
"api_61": {
"name": "AV-玉兔资源",
"api": "https://apiyutu.com/api.php/provide/vod",
"detail": "https://apiyutu.com"
},
"api_62": {
"name": "AV-番号资源",
"api": "http://fhapi9.com/api.php/provide/vod",
"detail": ""
},
"api_63": {
"name": "AV-白嫖资源",
"api": "https://www.kxgav.com/api/json.php",
"detail": "https://www.kxgav.com"
},
"api_64": {
"name": "AV-精品资源",
"api": "https://www.jingpinx.com/api.php/provide/vod",
"detail": "https://www.jingpinx.com"
},
"api_65": {
"name": "AV-美少女资源",
"api": "https://www.msnii.com/api/json.php",
"detail": "https://www.msnii.com"
},
"api_66": {
"name": "AV-老色逼资源",
"api": "https://apilsbzy1.com/api.php/provide/vod",
"detail": "https://apilsbzy1.com"
},
"api_67": {
"name": "AV-色南国",
"api": "https://api.sexnguon.com/api.php/provide/vod",
"detail": "https://api.sexnguon.com"
},
"api_68": {
"name": "AV-色猫资源",
"api": "https://api.maozyapi.com/inc/apijson_vod.php",
"detail": "https://api.maozyapi.com"
},
"api_69": {
"name": "AV-辣椒资源",
"api": "https://apilj.com/api.php/provide/vod",
"detail": "https://apilj.com"
},
"api_70": {
"name": "AV-香奶儿资源",
"api": "https://www.gdlsp.com/api/json.php",
"detail": "https://www.gdlsp.com"
},
"api_71": {
"name": "AV-鲨鱼资源",
"api": "https://shayuapi.com/api.php/provide/vod",
"detail": "https://shayuapi.com"
},
"api_72": {
"name": "AV-黄AV资源",
"api": "https://www.pgxdy.com/api/json.php",
"detail": "https://www.pgxdy.com"
},
"ffzynew": {
"api": "https://api.ffzyapi.com/api.php/provide/vod",
"name": "非凡影视new",
"detail": "http://ffzy5.tv"
},
"jisu": {
"api": "https://jszyapi.com/api.php/provide/vod",
"name": "极速资源",
"detail": "https://jszyapi.com"
},
"mozhua": {
"api": "https://mozhuazy.com/api.php/provide/vod",
"name": "魔爪资源"
},
"mdzy": {
"api": "https://www.mdzyapi.com/api.php/provide/vod",
"name": "魔都资源"
},
"kauiboziyuan": {
"api": "https://gayapi.com/api.php/provide/vod",
"name": "快播资源网站"
},
"xingbaziyuan": {
"api": "https://xingba111.com/api.php/provide/vod",
"name": "杏吧资源"
},
"liangziziyuan": {
"api": "https://cj.lziapi.com/api.php/provide/vod",
"name": "量子资源"
},
"senlinziyuan": {
"api": "https://slapibf.com/api.php/provide/vod",
"name": "森林资源"
},
"aiduanjucc": {
"api": "https://www.aiduanju.cc/",
"name": "爱短剧.cc"
},
"huaweiba": {
"api": "https://huawei8.live/api.php/provide/vod",
"name": "华为吧资源"
},
"taopian": {
"api": "https://taopianapi.com/cjapi/sda/vod",
"name": "淘片资源"
},
"hongniuziyuan": {
"api": "https://www.hongniuzy3.com/api.php/provide/vod",
"name": "红牛资源"
},
"suonisandian": {
"api": "https://xsd.sdzyapi.com/api.php/provide/vod",
"name": "索尼-闪电资源"
},
"yayaziyuan": {
"api": "https://cj.yayazy.net/api.php/provide/vod",
"name": "鸭鸭资源"
},
"jinyingziyuan": {
"api": "https://jyzyapi.com/provide/vod",
"name": "金鹰资源采集网"
},
"fengchao": {
"api": "https://api.fczy888.me/api.php/provide/vod",
"name": "蜂巢片库"
},
"jinmaziyuan2": {
"api": "https://api.jmzy.com/api.php/provide/vod",
"name": "金马资源网"
},
"dadiziy": {
"api": "https://dadiapi.com/api.php/provide/vod",
"name": "大地资源网络"
},
"huangseziy": {
"api": "https://hsckzy888.com/api.php/provide/vod",
"name": "黄色资源啊啊"
},
"xiaojiziy": {
"api": "https://api.xiaojizy.live/provide/vod",
"name": "小鸡资源"
},
"kauicheziyuan": {
"api": "https://caiji.kuaichezy.org/api.php/provide",
"name": "快车资源阿"
},
"xinlangaa": {
"api": "https://api.xinlangapi.com/xinlangapi.php/provide/vod",
"name": "新浪资源阿"
},
"lajiaoziyu": {
"api": "https://apilj.com/api.php/provide",
"name": "辣椒资源黄黄"
},
"youzhidianying": {
"api": "https://api.yzzy-api.com/inc/ldg_api_all.php/provide/vod",
"name": "优质资源库1080zyk6.com高清"
},
"iqiyi": {
"api": "https://www.iqiyizyapi.com/api.php/provide/vod",
"name": "iqiyi资源"
},
"xibaocaiji": {
"api": "https://www.xxibaozyw.com/api.php/provide/vod",
"name": "细胞采集黄色"
},
"qiqiqiqi": {
"api": "https://www.qiqidys.com/api.php/provide/vod/",
"name": "七七影视"
},
"yingshigongchang": {
"api": "https://cj.lziapi.com/api.php/provide/vod/",
"name": "影视工厂"
},
"fantuanyingshi": {
"api": "https://www.fantuan.tv/api.php/provide/vod/",
"name": "饭团影视"
}
}
}
```

### 重要声明

- 本项目仅供学习和个人使用
- 请勿将部署的实例用于商业用途或公开服务
- 如因公开分享导致的任何法律问题，用户需自行承担责任
- 项目开发者不对用户的使用行为承担任何法律责任
- 本项目不在中国大陆地区提供服务。如有该项目在向中国大陆地区提供服务，属个人行为。在该地区使用所产生的法律风险及责任，属于用户个人行为，与本项目无关，须自行承担全部责任。特此声明

## License

[MIT](LICENSE) © 2025 MoonTV & Contributors

## 致谢

- [ts-nextjs-tailwind-starter](https://github.com/theodorusclarence/ts-nextjs-tailwind-starter) — 项目最初基于该脚手架。
- [LibreTV](https://github.com/LibreSpark/LibreTV) — 由此启发，站在巨人的肩膀上。
- [ArtPlayer](https://github.com/zhw2590582/ArtPlayer) — 提供强大的网页视频播放器。
- [HLS.js](https://github.com/video-dev/hls.js) — 实现 HLS 流媒体在浏览器中的播放支持。
- [Zwei](https://github.com/bestzwei) — 提供获取豆瓣数据的 cors proxy
- [CMLiussss](https://github.com/cmliu) — 提供豆瓣 CDN 服务
- 感谢所有提供免费影视接口的站点。

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=MoonTechLab/LunaTV&type=Date)](https://www.star-history.com/#MoonTechLab/LunaTV&Date)
