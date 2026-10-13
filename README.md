# occmaps
全球灵异地点地图-开源
# 部署教程 · 全球灵异地点地图（开源版）

> 本文面向下载源码的用户，覆盖本地电脑（phpStudy/XAMPP/WAMP）、宝塔面板 VPS、Linux 命令行三种环境。
> 按教程操作**不会报错**；万一遇到问题，文末有「常见报错对照表」。

**版权归中国灵异网所有 · [talkghost.com](https://talkghost.com)**

---

## 〇、先回答：会不会报错？

- **按本教程步骤走：不会。** 首次访问网站会自动建表，无需手动导入 SQL。
- **唯一必改的配置**：数据库连接（`api/config.php`）。默认值已适配 XAMPP / WAMP / phpStudy（root + 空密码），本地环境通常不用改。
- **最容易踩的三个坑**（都已在本教程覆盖）：
  1. 数据库没创建 → `Unknown database 'occ_gs'`
  2. `config.php` 凭据没改成自己的 → `Access denied for user`
  3. Apache 没开 `mod_rewrite` → 500 错误

---

## 一、环境要求

| 项目 | 要求 |
|---|---|
| PHP | 7.4 或更高（8.x 兼容），需开启 `pdo_mysql` 扩展 |
| 数据库 | MySQL 5.7+ / MariaDB 10.3+ |
| Web 服务器 | Apache（需 mod_rewrite）或 Nginx |
| 目录权限 | `uploads/` 可写 |

---

## 二、本地电脑部署（phpStudy / XAMPP / WAMP）

以 phpStudy 为例，XAMPP/WAMP 同理：

1. **放源码**：解压后把 `occ.gs.cn` 文件夹放到网站根目录：
   - phpStudy：`D:\phpstudy_pro\WWW\occ.gs.cn\`
   - XAMPP：`C:\xampp\htdocs\occ.gs.cn\`
2. **启动**：打开面板，启动 MySQL 和 Apache/Nginx。
3. **建数据库**：面板里打开 phpMyAdmin（或命令行），执行：
   ```sql
   CREATE DATABASE occ_gs DEFAULT CHARSET utf8mb4;
   ```
4. **配置数据库**（通常可跳过）：编辑 `api/config.php`。默认 `root` + 空密码即 phpStudy/XAMPP/WAMP 的默认值；如果你的 MySQL root 有密码（phpStudy 部分版本为 `root`），改 `DB_PASS`。
5. **访问**：浏览器打开 `http://localhost/occ.gs.cn/`（或你配置的域名）。
6. **自检**：访问 `http://localhost/occ.gs.cn/api/health.php`，应返回：
   ```json
   {"code":0,"data":{"php":"8.1.0","pdo_mysql":true,"database":true,"places_table":true,...}}
   ```
   任何一项为 `false`，按下文对照表排查。

---

## 三、宝塔面板部署（VPS，最常用）

1. **安装环境**：宝塔面板 → 软件商店 → 安装 Nginx（或 Apache）+ MySQL + PHP 7.4+（PHP 设置里安装 `pdo_mysql` 扩展）。
2. **建站**：网站 → 添加站点，根目录指向解压后的 `occ.gs.cn` 目录；PHP 版本选刚装的。
3. **建数据库**：站点旁的「数据库」→ 创建数据库，记下库名、用户名、密码。
4. **配置**：编辑 `api/config.php`，把 `DB_NAME`/`DB_USER`/`DB_PASS` 改成刚创建的（`DB_HOST` 保持 `127.0.0.1`）。
5. **伪静态**（Nginx 用户）：站点设置 → 伪静态 → 选择 `thinkphp` 或直接粘贴：
   ```nginx
   location / {
       try_files $uri $uri/ =404;
   }
   ```
   （Apache 用户无需此步，根目录 `.htaccess` 自动生效。）
6. **目录权限**：文件 → 进入站点目录 → 勾选 `uploads` → 权限设为 `755`（属主为 `www`）。
7. **访问站点**，打开 `api/health.php` 自检（见上）。

---

## 四、Linux 命令行部署（Nginx + PHP-FPM + MySQL）

```bash
# 1. 放代码（以 /var/www 为例）
mkdir -p /var/www && cd /var/www
tar -xzf occ-gs-cn-opensource.tar.gz
cd occ.gs.cn

# 2. 建库和用户
mysql -u root -p <<'SQL'
CREATE DATABASE occ_gs DEFAULT CHARSET utf8mb4;
CREATE USER 'occ_gs'@'localhost' IDENTIFIED BY '换成强密码';
GRANT ALL PRIVILEGES ON occ_gs.* TO 'occ_gs'@'localhost';
FLUSH PRIVILEGES;
SQL

# 3. 配置（用编辑器修改 DB_NAME/DB_USER/DB_PASS）
vim api/config.php

# 4. 权限
chown -R www-data:www-data . && chmod -R 755 uploads

# 5. Nginx 站点配置（server 块内）
#    root /var/www/occ.gs.cn;
#    location / { try_files $uri $uri/ =404; }
#    location ~ \.php$ { include snippets/fastcgi-php.conf; fastcgi_pass unix:/run/php/php8.1-fpm.sock; }

systemctl reload nginx
```

---

## 五、初始化与管理后台

1. **自动建表**：数据库连接正常后，**首次访问任意页面即自动建表**（地点表、分类、设置等约 20 张），无需导入 SQL。
2. **管理员初始化**：打开 `admin/` 登录页，**随意输入任意用户名密码提交一次**——系统会创建初始管理员 `admin` 并把随机密码写入：
   ```
   api/initial_admin_password.txt
   ```
3. **登录**：用文件里的密码登录后台。
4. **必做安全操作**：
   - 立即在后台修改密码；
   - 删除 `api/initial_admin_password.txt`；
   - `api/config.php` 不要提交到公开仓库（已填真实凭据的话加 `.gitignore`）。

---

## 六、地图显示说明

- **不配任何 Key 也能看地图**：默认回退 Esri 国际街道 / OSM 底图（界面左下角有标注）。
- **配 Key 效果更好**：后台「地图设置」里填腾讯地图 / 天地图 / 高德 Key（高德 Key 需在开放平台申请，并配置域名白名单）。
- 前端右上角可切换「高德标准 / 天地图 / 暗色 / OSM」等底图，未配置的选项会自动隐藏。

---

## 七、常见报错对照表

| 报错/现象 | 原因 | 解决 |
|---|---|---|
| `SQLSTATE[HY000] [1045] Access denied for user 'xxx'@'localhost'` | `config.php` 数据库账号密码不对 | 改 `DB_USER`/`DB_PASS`；本地默认 `root` + 空密码 |
| `SQLSTATE[HY000] [1049] Unknown database 'occ_gs'` | 数据库没创建 | 执行 `CREATE DATABASE occ_gs DEFAULT CHARSET utf8mb4;` |
| `Base table or view doesn't exist: spooky_places` | 旧版源码包缺陷（新包已修复）；或数据库用户无建表权限 | 换用本教程随附的最新包；或给用户加 `CREATE/ALTER` 权限 |
| 500 Internal Server Error（一访问就 500） | Apache 未启用 mod_rewrite，`.htaccess` 无法解析 | 开启 `rewrite` 模块并重启；或临时删除根目录 `.htaccess`（仅损失目录斜杠跳转） |
| `api/health.php` 显示 `pdo_mysql:false` | PHP 未装 pdo_mysql | phpStudy：扩展里勾选；宝塔：PHP 设置→安装扩展；Linux：`apt install php-mysql` |
| `database:false` 但凭据正确 | MySQL 没启动 / 端口不对 / 库不存在 | 启动 MySQL；确认 `DB_PORT`（默认 3306） |
| 图片上传失败、`writable_uploads:false` | `uploads/` 不可写 | `chmod 755 uploads`（属主 `www`/`www-data`），必要时 `777` |
| `License Error: 底部版权声明已被篡改或删除` | 版权保护机制被触发（删改了 TG-CREDIT 标记/`credit-guard.js`/`api/license-guard.php`） | 从官方渠道重新获取源码；二次开发请保留版权标记（见 README） |
| Nginx 下 `/types/`、`/explore/` 404 | `.htaccess` 在 Nginx 不生效 | 按「三、5」配置 `try_files $uri $uri/ =404;` |
| 地图区域空白 | 未配地图 Key 且默认底图被墙/网络不通 | 后台配置腾讯/天地图 Key；或切换「OSM 全球」底图 |
| 登录成功马上掉登录 | Cookie 问题 | 确认站点域名访问一致（不要 localhost 和 127.0.0.1 混用）；HTTPS 站点检查证书 |

---

## 八、从旧版本升级

1. 备份数据库和 `uploads/`。
2. 用新包覆盖代码文件（`api/config.php` 保留你原来的配置）。
3. 访问一次任意页面，自动完成表结构迁移（只增字段，不丢数据）。

---

## 九、安全清单（上线前过一遍）

- [ ] 已修改管理员默认密码
- [ ] 已删除 `api/initial_admin_password.txt`
- [ ] `api/config.php` 未包含在公开仓库中（或只有默认占位值）
- [ ] `uploads/` 已禁止执行 PHP（包内 `.htaccess` 已处理，Nginx 用户需自行加 `location ~* ^/uploads/.*\.(php|py|sh)$ { deny all; }`）
- [ ] 保留了底部版权声明（技术机制会校验，删除会导致站点停止运行）
