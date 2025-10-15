# Roundcube 独立安装与优化标准作业程序 (SOP) - DirectAdmin 环境

本文档为在 DirectAdmin 主机环境下，从零开始完整部署并优化 Roundcube Webmail 提供了一份详尽的、分步式的标准作业程序。该指南已包含所有常见问题的排错方案，以确保安装过程顺畅无阻。

---

## 阶段一：初始准备工作

### 步骤 1: 下载与放置文件
1.  **下载**: 从 [Roundcube 官方网站](https://roundcube.net/download) 获取最新的 **Complete (完整)** 安装包 (例如 `roundcubemail-1.6.11-complete.tar.gz`)。
2.  **上传**: 使用 DirectAdmin 的**文件管理器**，将 `.tar.gz` 文件上传到您 `mail` 子域名的根目录 (例如 `/home/user/domains/mail.your-domain.com/public_html/`)。
3.  **解压缩**: 使用文件管理器中的 "Extract" (解压缩) 功能。
4.  **整理文件**: 将解压后生成的新子文件夹 (例如 `roundcubemail-1.6.11/`) 中的**所有内容**，移动到 `mail` 子域名的根目录中。

### 步骤 2: 创建数据库
1.  在您的新主机 DirectAdmin 面板中，进入 **"MySQL 管理"**。
2.  创建一个全新的、专用的数据库和用户。
3.  安全地记录下最终的凭据信息。例如：
    - **主机名 (Hostname)**: `localhost`
    - **数据库名 (Database)**: `edu_mail`
    - **用户名 (Username)**: `edu_mail`
    - **密码 (Password)**: `6y32AYYhMHvDswwHRcEh`

---

## 阶段二：Web 安装与配置

### 步骤 3: 运行 Web 安装向导
1.  在浏览器中访问安装器 URL: `https://mail.your-new-domain.com/installer/`
2.  **环境检查**: 在第一页，确认所有必需项都显示为绿色的 "OK"，然后点击 **"Next"**。
3.  **配置页面**: 填写以下关键信息 (其他选项保持默认)：
    - **IMAP Host**: `ssl://mail.your-new-domain.com`
    - **SMTP Host**: `ssl://mail.your-new-domain.com`
    - **数据库信息**: 填入步骤 2 中记录的凭据。
    - **插件 (Plugins)**: 勾选 `archive`, `managesieve`, `password`, `vcard_attachments`, `zipdownload`。
4.  点击 **"CREATE CONFIG"**。

### 步骤 4: 安装器排错与最终化
创建配置后，您会进入 "Test config" (测试配置) 页面。请按以下顺序操作：

1.  **初始化数据库**:
    - 您会看到错误 `DB Schema: NOT OK (Database not initialized)` (数据库结构: 未初始化)。
    - **操作**: 直接点击 **`Initialize database` (初始化数据库)** 按钮一次。页面将会刷新。
    > **注意**: 如果您看到 `Table 'session' already exists` (数据表 'session' 已存在) 的错误，这表示初始化过程被中断。请进入 **phpMyAdmin**，选择您的数据库，**删除 (Drop)** 里面所有的表，然后回到安装页面再点击一次 `Initialize database` 按钮。

2.  **修复 MIME 类型映射**:
    - 您很可能会看到错误 `Mimetype to file extension mapping: NOT OK`。
    - **操作**:
        1.  从 [此链接](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types) 下载标准的 `mime.types` 文件。
        2.  将这个 `mime.types` 文件上传到 Roundcube 安装目录下的 `/config/` 文件夹中。
        3.  编辑主配置文件 `/config/config.inc.php`，在文件末尾添加以下代码行，**并确保使用您账户的正确绝对路径**：
            ```php
            $config['mime_types'] = '/home/eebu/domains/mail.your-new-domain.com/public_html/config/mime.types';
            ```

3.  **执行最终连接测试**:
    - 完成以上修复后，刷新页面。所有检查项现在都应为绿色。
    - 使用一个预先创建好的测试邮箱账户 (例如 `test@eebu.edu.pl`) 来完成 **Test SMTP config** 和 **Test IMAP config** 的测试。
    - 看到两个测试都返回 **OK** 后，安装即告成功。

4.  **关键安全步骤**:
    - 回到文件管理器，**彻底删除整个 `installer` 文件夹**。

---

## 阶段三：安装后优化

### 步骤 5: 配置密码修改插件
1.  导航到 `/plugins/password/` 目录。
2.  复制 `config.inc.php.dist` 并将副本重命名为 `config.inc.php`。
3.  使用以下代码**完全替换**新 `config.inc.php` 文件的内容，并填入您新主机的 DirectAdmin 凭据：
    ```php
    <?php
    // DirectAdmin 密码修改插件配置
    $config['password_driver'] = 'directadmin';
    $config['password_directadmin_host'] = 'ssl://uk10.neodns.info'; // 您的 DA 面板地址
    $config['password_directadmin_port'] = 2222;
    $config['password_directadmin_username'] = 'eebu'; // 您的 DA 用户名
    $config['password_directadmin_password'] = 'Abc123'; // 您的 DA 密码
    ?>
    ```
4.  登录 Roundcube，进入“设置”菜单测试密码修改功能。

### 步骤 6: 自定义外观
1.  **准备文件**: 创建您的 `logo.png` (例如 200x50px) 和 `favicon.ico` (32x32px) 文件。
2.  **上传文件**: 将 `logo.png` 和 `favicon.ico` 两个文件都上传到 Roundcube 的根目录 (例如 `/public_html/`)。
3.  **配置 Logo**: 编辑主配置文件 `/config/config.inc.php`，在文件末尾添加以下代码。请确保文件名与您上传的完全一致，**文件名前的 `/` 至关重要**。
    ```php
    // 设置自定义 Logo
    $config['skin_logo'] = '/logo.png';
    ```
4.  **验证**: 使用浏览器的隐私/无痕模式访问您的登录页面，检查新的 Logo 和 Favicon 是否生效。

---

**部署完成。** 您的新 Roundcube Webmail 现已完全安装、优化并品牌化。
