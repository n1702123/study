---
marp: true
theme: default
paginate: true
backgroundColor: #1a1a2e
color: #eee
style: |
  section {
    font-family: 'Microsoft JhengHei', 'Noto Sans TC', sans-serif;
  }
  section.lead {
    text-align: center;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
  }
  section.lead h1 {
    font-size: 2.5em;
    color: #e94560;
  }
  section.lead h2 {
    color: #ccc;
    font-weight: normal;
  }
  h1 { color: #e94560; }
  h2 { color: #0f9ef7; }
  h3 { color: #f0c040; }
  code { background: #16213e; color: #0f9ef7; }
  pre { background: #16213e !important; border-radius: 8px; }
  pre code { color: #eee; }
  table { font-size: 0.85em; }
  th { background: #0f3460; }
  td { background: #16213e; }
  a { color: #0f9ef7; }
  blockquote { border-left: 4px solid #e94560; padding-left: 16px; color: #ccc; font-style: italic; }
  strong { color: #f0c040; }
  .emoji { font-size: 2em; }
  section.section-divider {
    text-align: center;
    display: flex;
    flex-direction: column;
    justify-content: center;
    background: linear-gradient(135deg, #0f3460 0%, #1a1a2e 100%);
  }
  section.section-divider h1 {
    font-size: 2.8em;
  }
  section.section-divider h2 {
    color: #ccc;
    font-weight: normal;
    font-size: 1.3em;
  }
---

<!-- _class: lead -->

# 🐳 Docker 教育訓練

## 從零開始學容器化

---

## 今天的目標

學完這堂課，你能夠：

1. **理解** Container 與 VM 的差異
2. **撰寫** Dockerfile 建立自己的 Image
3. **操作** Container 的生命週期與資料管理
4. **使用** Docker Compose 管理多容器應用

> 總時長約 2.5 小時（含休息與互動）

---

## 課程大綱

| 段落 | 主題 | 時間 |
|------|------|------|
| 一 | 基礎概念：VM vs Container、架構 | 30 min |
| 二 | Dockerfile 與 Image 管理 | 30 min |
| — | **休息** | 10 min |
| 三 | Container 操作、Volume、Network | 35 min |
| 四 | Docker Compose | 35 min |
| 五 | 面試重點整理 + Q&A | 15 min |

---

<!-- _class: section-divider -->

# 一、基礎概念
## 為什麼需要 Docker？

---

## 傳統部署的痛點

```
開發者：「在我的電腦上可以跑啊！」

維運人員：「但在 server 上就是跑不起來。」
```

常見問題：
- 不同環境的**套件版本不一致**
- 作業系統差異導致程式行為不同
- 安裝步驟複雜且容易出錯
- 多個應用在同一台 server 上互相干擾

---

## Docker 的解決方案

把所有東西打包成一個 **Container**，到處都能跑：

```
開發環境  →  測試環境  →  正式環境
   ✅           ✅           ✅
       （完全相同的 Container）
```

> Docker 讓你的程式在任何地方都能用**相同的方式**運行。

---

## VM vs Container

```
   Virtual Machine                    Container
┌────────────────────┐     ┌────────────────────┐
│ App A │ App B│App C│     │ App A│ App B│ App C│
│ Bins  │ Bins │Bins │     │ Bins │ Bins │ Bins │
│GuestOS│GuestOS│GstOS│     ├────────────────────┤
├────────────────────┤     │   Docker Engine    │
│    Hypervisor      │     ├────────────────────┤
├────────────────────┤     │     Host OS        │
│      Host OS       │     ├────────────────────┤
│     Hardware       │     │    Hardware        │
└────────────────────┘     └────────────────────┘
```

---

## VM vs Container 比較

| 比較項目 | VM | Container |
|---------|-----|-----------|
| 啟動速度 | 數分鐘 | **數秒** |
| 大小 | 數 GB | **數十 MB** |
| 效能損耗 | 較大（需要完整 OS） | **極小**（共享 kernel） |
| 隔離程度 | 完全隔離 | 行程級隔離 |
| 適用場景 | 需要不同 OS | **微服務、CI/CD** |

> 類比：VM = 整棟房子 🏠｜Container = 一間房間 🚪（共用水電）

---

## Docker 架構

```
┌─────────────┐    REST API    ┌──────────────────┐
│ Docker CLI  │ ─────────────► │  Docker Daemon    │
│ (Client)    │                │  (dockerd)        │
└─────────────┘                │                   │
                               │  ┌─────────────┐  │
 docker build ──────────────►  │  │  Images     │  │
 docker pull  ──────────────►  │  │  Containers │  │
 docker run   ──────────────►  │  │  Networks   │  │
                               │  │  Volumes    │  │
                               │  └─────────────┘  │
                               └──────────────────┘
```

Docker 採用 **Client-Server** 架構

---

## 核心組件

| 組件 | 說明 |
|------|------|
| **Docker Client** | 你輸入指令的地方（`docker` 命令） |
| **Docker Daemon** | 背景服務，管理所有 Container |
| **Docker Image** | 唯讀的模板，用來建立 Container |
| **Docker Container** | Image 的**執行實例** |
| **Docker Registry** | 存放 Image 的地方（如 Docker Hub） |

---

## Image 與 Container 的關係

```
Image     = 蛋糕食譜 🍰   → 唯讀、可分享
Container = 做出的蛋糕 🎂   → 可執行、可修改
```

- 一個 Image 可以建立**多個** Container
- Container 是 Image 的執行實例
- Container 停止後，變更**不會**影響原本的 Image

---

## 🖥️ Demo 1：基礎指令操作

```bash
# 你的第一個 Container
docker run hello-world

# 進入 Ubuntu Container
docker run -it ubuntu bash

# 在背景執行 Nginx
docker run -d -p 8080:80 --name my-nginx nginx:alpine

# 管理指令
docker ps          # 列出執行中的 Container
docker ps -a       # 列出所有 Container
docker logs my-nginx
docker exec -it my-nginx sh
docker stop my-nginx
docker rm my-nginx
```

---

## 常用指令速查表

| 指令 | 說明 |
|------|------|
| `docker run <image>` | 建立並啟動 Container |
| `docker ps` / `docker ps -a` | 列出 Container |
| `docker images` | 列出本地 Image |
| `docker pull <image>` | 下載 Image |
| `docker stop / start` | 停止 / 啟動 Container |
| `docker rm <container>` | 移除 Container |
| `docker rmi <image>` | 移除 Image |
| `docker logs <container>` | 查看 log |
| `docker exec -it <container> bash` | 進入 Container |

---

## 🧠 互動 1：快問快答

**Q1：Docker Container 和 VM 最大的差異是什麼？**

**Q2：執行 `docker run ubuntu` 後 Container 馬上停了，為什麼？**
- A. Image 壞掉了
- B. 沒有加 `-d`
- C. Container 內沒有持續執行的程式
- D. 記憶體不夠

**Q3：`docker ps` 和 `docker ps -a` 有什麼不同？**

---

<!-- _class: section-divider -->

# 二、Dockerfile 與 Image 管理
## 建立自己的 Docker Image

---

## 什麼是 Dockerfile？

一個純文字檔案，告訴 Docker **如何一步步建立** Image。

```
Dockerfile  →  docker build  →  Image  →  docker run  →  Container
 （設計圖）      （建造）        （成品）      （執行）      （實例）
```

---

## Dockerfile 基本結構

```dockerfile
# 基底 Image
FROM node:20-alpine

# 設定工作目錄
WORKDIR /app

# 複製 package.json（利用 cache）
COPY package*.json ./

# 安裝依賴
RUN npm install

# 複製其餘檔案
COPY . .

# 宣告 port
EXPOSE 3000

# 容器啟動時執行的指令
CMD ["node", "server.js"]
```

---

## 重要指令詳解

### FROM — 基底 Image
```dockerfile
FROM node:20-alpine   # 推薦用 alpine（更小）
FROM node:20-slim     # 或 slim 版本
FROM scratch          # 從空白開始
```

### WORKDIR — 工作目錄
```dockerfile
WORKDIR /app
# 之後的指令都在 /app 下執行
```

---

### COPY vs ADD
| 功能 | COPY | ADD |
|------|------|-----|
| 複製本地檔案 | ✅ | ✅ |
| 自動解壓 tar | ❌ | ✅ |
| **推薦程度** | **優先使用** | 僅在需要時 |

---

## CMD vs ENTRYPOINT（面試必考！）

```dockerfile
# CMD：預設指令，可以被覆蓋
CMD ["node", "server.js"]
```
```bash
docker run myapp              # 執行 node server.js
docker run myapp node test.js # CMD 被覆蓋 → node test.js
```

```dockerfile
# ENTRYPOINT：一定會執行，不易被覆蓋
ENTRYPOINT ["node"]
CMD ["server.js"]             # 提供預設參數
```
```bash
docker run myapp              # 執行 node server.js
docker run myapp test.js      # 執行 node test.js（只覆蓋 CMD）
```

> 記憶：**CMD 可覆蓋，ENTRYPOINT 不易覆蓋**

---

## Image Layer 與 Cache

```
┌──────────────────────┐
│ CMD ["node", "..."]  │  ← Layer 5
├──────────────────────┤
│ COPY . .             │  ← Layer 4（程式碼）
├──────────────────────┤
│ RUN npm install      │  ← Layer 3（依賴）
├──────────────────────┤
│ COPY package*.json   │  ← Layer 2
├──────────────────────┤
│ FROM node:20-alpine  │  ← Layer 1（base）
└──────────────────────┘
```

**原則**：不常變的放前面 → 善用 cache → 加速 build

---

## ❌ vs ✅ Cache 最佳實踐

```dockerfile
# ❌ 不好：每次改程式碼都要重新 npm install
COPY . .
RUN npm install
```

```dockerfile
# ✅ 好：只有 package.json 變了才重新 install
COPY package*.json ./
RUN npm install
COPY . .
```

> 改一行程式碼，不需要重裝所有套件！

---

## Multi-stage Build

大幅**減少 Image 大小**：

```dockerfile
# ===== Stage 1：Build =====
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# ===== Stage 2：Production =====
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

| | 沒有 Multi-stage | 有 Multi-stage |
|---|---|---|
| 大小 | ~1.2GB | **~200MB** |

---

## .dockerignore

跟 `.gitignore` 類似，排除不需要的檔案：

```
node_modules
.git
.env
Dockerfile
docker-compose.yml
README.md
.vscode
```

好處：
- **加速 build**（減少 build context）
- **安全**（避免 `.env` 進入 Image）

---

## 🖥️ Demo 2：Build 自己的 Image

```bash
# 建立一個簡單的 web app
mkdir my-web && cd my-web

# 寫 Dockerfile
cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
EOF

# Build！
docker build -t my-web-app .

# 執行
docker run -d -p 9090:80 --name demo-web my-web-app
# 開瀏覽器 → http://localhost:9090
```

---

<!-- _class: section-divider -->

# ☕ 休息 10 分鐘
## 回來後：Container 操作、Volume、Network、Compose

---

<!-- _class: section-divider -->

# 三、Container 操作與資料管理
## 環境變數、Port Mapping、Volume

---

## 環境變數

```bash
# 用 -e 傳遞
docker run -e DB_HOST=localhost -e DB_PORT=5432 postgres

# 用 env file 傳遞
docker run --env-file app.env postgres
```

```bash
# app.env
DB_HOST=localhost
DB_PORT=5432
DB_PASSWORD=secret123
```

> 實務上用來傳資料庫密碼、環境設定等

---

## Port Mapping

```
  Host（你的電腦）           Container
 ┌──────────────┐         ┌──────────────┐
 │              │         │              │
 │  Port 8080 ──┼─────────┼── Port 80   │
 │              │         │              │
 └──────────────┘         └──────────────┘
```

```bash
# 格式：-p <host_port>:<container_port>
docker run -d -p 8080:80 nginx

# 多個 port
docker run -d -p 8080:80 -p 8443:443 nginx
```

---

## 資料持久化：為什麼需要？

Container 被**刪除後**，裡面的資料就**消失了**。

如果跑的是資料庫呢？ → 需要 **Volume**！

### 三種方式

| 方式 | 管理者 | 適用場景 |
|------|--------|---------|
| **Named Volume** | Docker 管理 | 資料庫、持久儲存 |
| **Bind Mount** | 你指定路徑 | 開發時同步程式碼 |
| **tmpfs** | 記憶體 | 暫存敏感資料 |

---

## Named Volume vs Bind Mount

```bash
# Named Volume — Docker 管理的儲存空間
docker run -v my-data:/var/lib/mysql mysql:8.0

# Bind Mount — 你指定 Host 上的路徑
docker run -v $(pwd)/src:/app/src node:20
```

> 類比：
> **Named Volume** = 把資料交給 Docker 保管 📦
> **Bind Mount** = 把你的資料夾「共享」給 Container 📂

---

## 🖥️ Demo 3：Volume 持久化

```bash
# ❌ 沒有 Volume → 砍掉 Container，資料消失
docker run -d --name db-no-vol -e MYSQL_ROOT_PASSWORD=pass mysql:8.0
# （寫入資料 → 刪除 Container → 重建 → 資料不見了）

# ✅ 有 Volume → 砍掉 Container，資料還在
docker run -d --name db-with-vol \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=pass mysql:8.0
# （寫入資料 → 刪除 Container → 重建並掛同一個 Volume → 資料還在！）
```

---

## Container 偵錯工具

```bash
# 查看 log
docker logs <container>
docker logs -f <container>         # 即時追蹤
docker logs --tail 100 <container> # 最後 100 行

# 進入 Container
docker exec -it <container> bash

# 查看詳細資訊
docker inspect <container>

# 即時資源監控
docker stats
```

---

## 🧠 互動 2：情境題

**同事把 MySQL Container 刪掉重建，客戶資料全消失了。**
**問題出在哪？怎麼預防？**

---

## Docker Network

多個 Container 要互相通訊？→ 需要 **Network**

```
┌─────────────────────────────────────┐
│          Docker Network             │
│                                     │
│  ┌──────────┐     ┌──────────┐     │
│  │  Web App  │────►│ Database │     │
│  │ (Node.js) │     │ (MySQL)  │     │
│  └──────────┘     └──────────┘     │
│                                     │
└─────────────────────────────────────┘
```

---

## 預設 bridge vs 自訂 bridge

| 功能 | 預設 bridge | 自訂 bridge |
|------|------------|------------|
| DNS 解析（用名稱互連） | ❌ 只能用 IP | ✅ 可用名稱 |
| 隔離性 | 所有 Container 混在一起 | 只有加入的才能通訊 |

```bash
# 建立自訂 network
docker network create my-network

# Container 加入 network
docker run -d --name app --network my-network my-app
docker run -d --name db  --network my-network mysql:8.0

# app 可以用 "db" 這個名稱連到 MySQL ✅
```

> **結論**：永遠使用自訂 bridge network！

---

<!-- _class: section-divider -->

# 四、Docker Compose
## 一個 YAML 搞定多容器應用

---

## 為什麼需要 Compose？

不用 Compose：
```bash
docker network create app-net
docker run -d --name db --network app-net -e MYSQL_ROOT_PASSWORD=pass mysql:8.0
docker run -d --name app --network app-net -p 3000:3000 -e DB_HOST=db my-app
docker run -d --name nginx --network app-net -p 80:80 my-nginx
# 每次都要打這麼多指令... 😩
```

用 Compose：
```bash
docker compose up -d    # 一行搞定！ 🎉
```

---

## docker-compose.yml 結構

```yaml
services:
  web:                          # Service 名稱 = hostname
    image: nginx:alpine
    ports:
      - "8080:80"

  app:
    build: ./app                # 從 Dockerfile build
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=db
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:                      # 宣告 Named Volume
```

---

## 常用指令

```bash
# 啟動所有 service（背景）
docker compose up -d

# 啟動並重新 build
docker compose up -d --build

# 查看狀態
docker compose ps

# 查看 log
docker compose logs -f

# 進入 Container
docker compose exec app bash

# 停止並移除
docker compose down

# 停止並移除 Volume（⚠️ 資料會消失）
docker compose down -v
```

---

## depends_on 的陷阱（面試常考！）

```yaml
services:
  app:
    depends_on:
      - db          # ⚠️ 只保證 db Container 先啟動
                    # 不保證 MySQL 已經 ready！
```

正確做法 → 搭配 **healthcheck**：

```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy   # ✅ 等 db 真的 ready

  db:
    image: mysql:8.0
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 3
```

---

## 環境變數管理

```yaml
# docker-compose.yml
services:
  db:
    image: postgres:${POSTGRES_VERSION:-16}   # 從 .env 讀取
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
  app:
    env_file:
      - app.env                               # 傳給 Container
```

```env
# .env（Compose 自動讀取）
POSTGRES_VERSION=16
DB_PASSWORD=secret
```

> `.env` 放在 `docker-compose.yml` 同一層會**自動被讀取**

---

## Network 在 Compose 中

```yaml
services:
  frontend:
    networks: [frontend-net]

  backend:
    networks: [frontend-net, backend-net]   # 兩邊都能通

  db:
    networks: [backend-net]                 # frontend 連不到 db

networks:
  frontend-net:
  backend-net:
```

> Compose 預設已自動建立一個 network，通常不需要特別設定

---

## 🖥️ Demo 4：Compose 起 WordPress

```yaml
# docker-compose.yml
services:
  wordpress:
    image: wordpress:latest
    ports: ["8080:80"]
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
    depends_on:
      db: { condition: service_healthy }

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: wordpress
    volumes: [db-data:/var/lib/mysql]
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]

volumes:
  db-data:
```

```bash
docker compose up -d    # 開啟 http://localhost:8080 🎉
```

---

## 🧠 互動 3：是非判斷

| # | 敘述 | 對 or 錯？ |
|---|------|-----------|
| 1 | `docker compose up -d` 會自動建立 network | ？ |
| 2 | `depends_on` 可以保證 MySQL 已 ready | ？ |
| 3 | `docker compose down` 會刪除 Volume | ？ |
| 4 | Container 之間可以用 service 名稱互連 | ？ |
| 5 | `.env` 只能用 `env_file` 才能讀取 | ？ |

---

## 互動 3：答案

| # | 答案 | 說明 |
|---|------|------|
| 1 | ✅ 對 | Compose 自動建立 default network |
| 2 | ❌ 錯 | 只保證啟動順序，要搭配 healthcheck |
| 3 | ❌ 錯 | 要加 `-v` 才會刪 Volume |
| 4 | ✅ 對 | Compose 自動提供 DNS 解析 |
| 5 | ❌ 錯 | `.env` 在同層會自動被讀取 |

> **第 2 題是面試超常考題！**

---

<!-- _class: section-divider -->

# 五、面試重點整理
## 帶走這些就夠了

---

## 面試必問 Top 10

| # | 問題 | 一句話回答 |
|---|------|-----------|
| 1 | Container vs VM？ | Container 共享 kernel，更輕更快 |
| 2 | Image vs Container？ | Image 是模板，Container 是執行實例 |
| 3 | CMD vs ENTRYPOINT？ | CMD 可覆蓋，ENTRYPOINT 不易覆蓋 |
| 4 | COPY vs ADD？ | COPY 更單純，優先使用 |
| 5 | Volume vs Bind Mount？ | Volume 由 Docker 管理，Bind Mount 你指定 |
| 6 | 預設 vs 自訂 bridge？ | 自訂才有 DNS 解析 |
| 7 | depends_on 保證什麼？ | 只保證啟動順序，不保證 ready |
| 8 | 如何減少 Image 大小？ | alpine + multi-stage + 合併 layer |
| 9 | Docker Compose 用途？ | 一個 YAML 管理多容器應用 |
| 10 | Docker 安全最佳實踐？ | 非 root + 特定 tag + 掃描漏洞 |

---

## 學習路徑

```
Docker 基礎
    │
    ├── Container & Image 基本操作     ← 今天學了 ✅
    ├── Dockerfile 撰寫               ← 今天學了 ✅
    ├── Volume & Network              ← 今天學了 ✅
    └── Docker Compose                ← 今天學了 ✅
         │
         └── 進階主題
              ├── Security（非 root、Image 掃描）
              ├── CI/CD 整合（GitHub Actions）
              └── Container Orchestration
                   └── Kubernetes（業界主流）
```

---

## 推薦資源

| 資源 | 說明 |
|------|------|
| [Docker 官方文件](https://docs.docker.com/) | 最權威的參考資料 |
| [Docker Hub](https://hub.docker.com/) | 查找官方 Image |
| [Play with Docker](https://labs.play-with-docker.com/) | 免費線上練習環境 |
| [Docker Getting Started](https://docs.docker.com/get-started/) | 官方入門教學 |

---

<!-- _class: lead -->

# 🐳 Q&A 時間

## 有任何問題歡迎提問！

> Docker 最好的學習方式就是**動手做**。
> 今天的教材回去一定要自己跑一遍。
