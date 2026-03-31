# Docker 教育訓練 — Demo 腳本

> 講師在現場示範時，照著以下步驟操作即可。
> 每個 Demo 都標註了預估時間和講解要點。

---

## Demo 1：基礎指令操作（15 min）

### 1-1. Hello World（3 min）

```bash
# 執行你的第一個 Container
docker run hello-world
```

**講解要點**：
- 指出輸出中的步驟：pull → create → start → 輸出 → 停止
- 「Docker 先在本地找 image，找不到就去 Docker Hub 下載」

### 1-2. 進入 Ubuntu Container（4 min）

```bash
# 啟動一個 Ubuntu 容器，進入互動模式
docker run -it ubuntu bash
```

在 Container 內部操作：
```bash
# 你現在是在 Container 裡面，不是你的電腦！
whoami                   # root
cat /etc/os-release      # 確認是 Ubuntu
ls /                     # 看看目錄結構
apt update               # 可以裝套件
exit                     # 離開
```

**講解要點**：
- 「你現在等於進到了一台迷你的 Linux 機器裡面」
- `-i` = interactive（保持輸入），`-t` = tty（分配終端機）
- exit 後 Container 就停止了

### 1-3. 背景執行 Nginx（4 min）

```bash
# 在背景啟動 Nginx web server
docker run -d -p 8080:80 --name my-nginx nginx:alpine
```

**講解要點**：
- `-d` = detach（背景執行）
- `-p 8080:80` = 「你電腦的 8080 port → Container 的 80 port」
- `--name` = 給 Container 一個好記的名字

```bash
# 開瀏覽器看 http://localhost:8080 → 看到 Nginx 歡迎頁面
```

### 1-4. 常用管理指令（4 min）

```bash
# 列出正在跑的 Container
docker ps

# 列出所有 Container（包含停止的）
docker ps -a

# 看 Container 的 log
docker logs my-nginx

# 進入正在跑的 Container
docker exec -it my-nginx sh
ls /usr/share/nginx/html/
cat /usr/share/nginx/html/index.html
exit

# 查看本地有哪些 Image
docker images

# 停止 Container
docker stop my-nginx

# 確認已停止
docker ps
docker ps -a    # 還在，但是 STATUS 是 Exited

# 移除 Container
docker rm my-nginx
```

**講解要點**：
- `docker ps` 只顯示 running 的，加 `-a` 才看到全部
- `docker exec` 可以「走進」正在跑的 Container
- 停止 ≠ 移除，stop 只是關機，rm 才是刪除

---

## Demo 2：Build 自己的 Image（10 min）

### 2-1. 準備檔案（2 min）

```bash
# 建立 Demo 目錄
mkdir -p ~/docker-demo/my-web && cd ~/docker-demo/my-web
```

建立 `index.html`：
```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Docker Demo</title>
    <style>
        body { font-family: Arial; text-align: center; padding-top: 100px; background: #1a1a2e; color: #eee; }
        h1 { font-size: 3em; }
        .container { background: #16213e; padding: 40px; border-radius: 10px; display: inline-block; }
        .emoji { font-size: 4em; }
    </style>
</head>
<body>
    <div class="container">
        <div class="emoji">🐳</div>
        <h1>Hello from Docker!</h1>
        <p>This page is running inside a Docker container.</p>
        <p>Built at: <strong>just now!</strong></p>
    </div>
</body>
</html>
EOF
```

建立 `Dockerfile`：
```bash
cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
EOF
```

建立 `.dockerignore`：
```bash
cat > .dockerignore << 'EOF'
Dockerfile
.dockerignore
EOF
```

### 2-2. Build Image（3 min）

```bash
# Build！
docker build -t my-web-app .
```

**講解要點**：
- `-t my-web-app` = 給 Image 取名字（tag）
- `.` = build context 是當前目錄
- 指出每一行輸出對應 Dockerfile 的哪個指令
- 「每一行指令就是一個 Layer」

```bash
# 看看 build 出來的 Image
docker images | head -5

# 看看 Image 的 Layer 歷史
docker history my-web-app
```

### 2-3. 執行並驗證（3 min）

```bash
# 用我們 build 的 Image 跑 Container
docker run -d -p 9090:80 --name demo-web my-web-app

# 開瀏覽器看 http://localhost:9090
```

**講解要點**：
- 「你剛才寫了一個 Dockerfile，Docker 幫你 build 成 Image，現在跑起來了」
- 「這就是 Docker 的核心流程：Dockerfile → Image → Container」

### 2-4. 展示 Cache 效果（2 min）

```bash
# 修改 index.html（改一個字）
sed -i 's/just now!/just now! (v2)/' index.html

# 重新 build — 注意速度差異，第一層用了 cache
docker build -t my-web-app .
```

**講解要點**：
- 指出 `CACHED` 字樣：「因為 FROM nginx:alpine 沒變，所以用 cache」
- 「這就是為什麼要把不常變的指令放前面」

```bash
# 清理
docker rm -f demo-web
```

---

## Demo 3：MySQL + Volume 持久化（10 min）

### 3-1. 沒有 Volume 的情況 — 資料消失（3 min）

```bash
# 啟動 MySQL（不用 Volume）
docker run -d \
  --name mysql-no-vol \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=testdb \
  mysql:8.0

# 等待 MySQL 啟動（約 15-20 秒）
echo "等 MySQL 啟動..."
sleep 20

# 寫入一筆資料
docker exec mysql-no-vol mysql -uroot -prootpass -e \
  "USE testdb; CREATE TABLE users (id INT, name VARCHAR(50)); INSERT INTO users VALUES (1, 'Alice');"

# 確認資料存在
docker exec mysql-no-vol mysql -uroot -prootpass -e "USE testdb; SELECT * FROM users;"
# → 看到 Alice ✅

# 砍掉 Container
docker rm -f mysql-no-vol

# 重新建立（同樣不用 Volume）
docker run -d \
  --name mysql-no-vol \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=testdb \
  mysql:8.0

sleep 20

# 查資料 → 資料不見了！
docker exec mysql-no-vol mysql -uroot -prootpass -e "USE testdb; SHOW TABLES;"
# → Empty set ❌
```

**講解要點**：
- 「Container 被刪掉，裡面的資料就跟著消失了」
- 「這就是為什麼我們需要 Volume」

```bash
docker rm -f mysql-no-vol
```

### 3-2. 有 Volume 的情況 — 資料保留（5 min）

```bash
# 建立一個 named volume
docker volume create mysql-data

# 啟動 MySQL（使用 Volume）
docker run -d \
  --name mysql-with-vol \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=testdb \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0

sleep 20

# 寫入資料
docker exec mysql-with-vol mysql -uroot -prootpass -e \
  "USE testdb; CREATE TABLE users (id INT, name VARCHAR(50)); INSERT INTO users VALUES (1, 'Alice'), (2, 'Bob');"

# 確認
docker exec mysql-with-vol mysql -uroot -prootpass -e "USE testdb; SELECT * FROM users;"
# → Alice, Bob ✅

# 砍掉 Container
docker rm -f mysql-with-vol

# 重新建立，掛載同一個 Volume
docker run -d \
  --name mysql-with-vol \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0

sleep 20

# 查資料 → 資料還在！
docker exec mysql-with-vol mysql -uroot -prootpass -e "USE testdb; SELECT * FROM users;"
# → Alice, Bob ✅✅
```

**講解要點**：
- 「Volume 是獨立於 Container 的儲存空間」
- 「Container 可以刪掉再建，只要掛載同一個 Volume，資料就在」
- 「所以生產環境的資料庫一定要用 Volume」

### 3-3. 清理（2 min）

```bash
# 查看 volume
docker volume ls

# 清理
docker rm -f mysql-with-vol
docker volume rm mysql-data
```

---

## Demo 4：Docker Compose 起 WordPress（15 min）

### 4-1. 準備檔案（3 min）

```bash
mkdir -p ~/docker-demo/wordpress && cd ~/docker-demo/wordpress
```

建立 `.env`：
```bash
cat > .env << 'EOF'
MYSQL_ROOT_PASSWORD=rootpass123
MYSQL_DATABASE=wordpress
MYSQL_USER=wp_user
MYSQL_PASSWORD=wp_secret
EOF
```

建立 `docker-compose.yml`：
```bash
cat > docker-compose.yml << 'EOF'
services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}
    volumes:
      - wp-content:/var/www/html/wp-content
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - db-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 3
    restart: unless-stopped

volumes:
  wp-content:
  db-data:
EOF
```

**講解要點**（邊寫邊講）：
- 「services 底下定義你要跑哪些 Container」
- 「wordpress service 用 8080 port，依賴 db」
- 「db service 用 healthcheck 確保 MySQL 真的 ready」
- 「`${MYSQL_USER}` 會從 .env 檔讀取變數」
- 「最下面的 volumes 區塊宣告要用的 named volume」

### 4-2. 啟動（5 min）

```bash
# 一行指令啟動所有服務
docker compose up -d
```

**講解要點**：
- 指出輸出中 network、volume 的自動建立
- 「Compose 自動幫你建了 network，所以 Container 之間可以用名稱互連」

```bash
# 查看狀態
docker compose ps

# 查看 log（等 db healthy）
docker compose logs -f db
# 看到 "ready for connections" 就 OK 了（Ctrl+C 退出）
```

### 4-3. 驗證（3 min）

```bash
# 開瀏覽器看 http://localhost:8080
# → 看到 WordPress 安裝畫面！
```

**講解要點**：
- 「只用了一個 YAML 檔案 + 一行指令，就架好了一個完整的 WordPress」
- 「這就是 Docker Compose 的威力」

```bash
# 更多常用指令示範
docker compose ps         # 查看狀態
docker compose logs wp    # 查看特定 service 的 log
docker compose exec db mysql -uroot -prootpass123 -e "SHOW DATABASES;"
```

### 4-4. 停止與清理（2 min）

```bash
# 停止所有服務
docker compose down

# 如果要連 Volume 一起清（資料會消失）
docker compose down -v
```

**講解要點**：
- `down` = 停止 + 移除 Container + 移除 Network
- `down -v` = 上述 + 移除 Volume（資料會消失，生產環境小心！）

### 4-5. 展示重啟後資料保留（2 min）

```bash
# 重新啟動（不加 -v）
docker compose up -d

# 等 db healthy
sleep 15
docker compose ps

# 開瀏覽器 http://localhost:8080 → 剛才設定的 WordPress 還在（因為 Volume）
```

```bash
# 最後清理
docker compose down -v
```

---

## Demo 備援方案

如果現場 Demo 出問題，可以用以下方式救場：

### 網路問題（無法 pull image）
- 使用預先 pull 好的 image
- 如果完全沒有 image，用截圖展示

### MySQL 啟動太慢
- sleep 時間拉長到 30 秒
- 或直接用 `docker compose logs -f db` 等到 ready

### Port 被佔用
```bash
# 查看哪個 port 被佔用
netstat -ano | findstr :8080

# 解法：改用其他 port
docker run -d -p 9999:80 --name my-nginx nginx:alpine
```

### Container 啟動後馬上停止
```bash
# 查看錯誤原因
docker logs <container-name>
```
