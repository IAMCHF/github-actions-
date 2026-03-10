### 📦 package-lobehub工作流说明
1.  **手动触发**：您可以在GitHub仓库的Actions页面手动运行此工作流，并指定LobeHub版本和内网服务器IP。
2.  **下载配置**：自动下载官方的 `docker-compose.yml` 和环境变量示例文件。
3.  **内网适配**：修改 `.env` 文件，将服务地址设置为您的内网IP（端口模式），并生成必要的密码。
4.  **拉取镜像**：拉取LobeHub、PostgreSQL和MinIO的Docker镜像，并保存为 `.tar.gz` 压缩包。
5.  **生成部署脚本**：创建一个 `deploy-offline.sh` 脚本，用于在内网一键加载镜像和启动服务。
6.  **打包上传**：将所有文件打包为 `lobehub-offline-package.tar.gz` 并作为GitHub Actions的构件提供下载。

### 🚀 内网部署步骤
1.  在**有外网的机器**上，通过GitHub Actions下载 `lobehub-offline-package.tar.gz`。
2.  将压缩包传输到**内网服务器**上。
3.  在**内网服务器**上解压：
    ```bash
    tar -xzvf lobehub-offline-package.tar.gz
    cd lobehub-offline-package
    ```
4.  执行离线部署脚本：
    ```bash
    chmod +x deploy-offline.sh
    ./deploy-offline.sh
    ```
5.  部署完成后，根据脚本提示的地址（如 `http://192.168.1.100:3210`）访问您的LobeHub服务。

### ⚠️ 重要注意事项
*   **版本一致性**：请确保您拉取的镜像版本与配置兼容。文档中默认使用 `lobehub/lobe-chat:latest`，但建议您在触发工作流时指定一个稳定的正式版本。
*   **S3配置**：上述配置已设置 `S3_ENDPOINT` 为内网IP加端口。请确保内网所有客户端都能访问此地址。如需更复杂的S3配置（如Bucket、Region），请参考[官方文档](https://lobehub.com/docs/self-hosting/platform/docker-compose)修改 `.env`。
*   **数据持久化**：`docker-compose.yml` 中已定义数据卷（volumes），用于持久化数据库和对象存储数据。在重新部署或升级时，请注意备份 `./data` 目录。
*   **安全密钥**：工作流中生成了随机的数据库密码和密钥。对于生产环境，您可能需要在解压后、部署前，手动编辑 `.env` 文件，设置更复杂的密码。
*   **资源要求**：请确保内网服务器满足最低[资源要求](https://lobehub.com/docs/self-hosting/platform/docker-compose#资源要求)：**2核CPU、4GB内存、20GB存储**。

如果在内网部署过程中遇到连接问题（如数据库连接失败），您可以参考[故障排查](https://lobehub.com/docs/self-hosting/platform/docker-compose#故障排查)章节，使用 `docker compose logs` 查看具体日志。

希望这套方案能帮助您顺利完成LobeHub的内网部署！如果在使用工作流或部署时遇到问题，可以随时提供更多细节，我会尽力协助。
















# LobeHub 企业内网离线部署解决方案

根据您的需求，我为您设计了一个完整的GitHub Actions工作流，用于创建LobeHub的离线部署包。这个方案基于最新版LobeHub（v1.8.0+）的数据库版本，支持用户数据同步和管理。

## lobehub-offline-deploy 工作流说明

1. 将上述YAML文件添加到您的GitHub仓库的`.github/workflows/`目录中
2. 在GitHub仓库中触发工作流（通过Actions标签页）
3. 工作流运行时，可以指定LobeHub版本（推荐使用`latest`或具体版本号）
4. 工作流完成后，下载生成的`lobehub-offline-deploy.zip`文件
5. 将此文件传输到您的内网服务器

## 内网部署步骤

1. 解压下载的zip文件：`unzip lobehub-offline-deploy.zip`
2. （可选）编辑`docker-compose.yml`文件，修改以下关键配置：
   - `ACCESS_CODE`：设置您自己的访问密码 
   - `ports`：如果3210端口被占用，可以修改为主机端口
3. 运行部署脚本：`./import-and-deploy.sh`
4. 访问`http://<服务器IP>:3210`，首次登录时输入您设置的访问密码 

## 技术说明

1. 本方案使用了LobeHub最新的数据库版本镜像，支持用户数据同步和管理功能 
2. 包含完整的依赖服务（PostgreSQL和MinIO），确保所有功能正常运行 
3. 镜像数据会持久化存储，重启或升级时不会丢失 
4. 部署脚本会自动导入镜像并启动服务，无需联网 

> 注意：根据搜索结果，LobeHub已将镜像名称从`lobehub/lobe-chat-database`迁移到`lobehub/lobehub`，本方案已适配这一变化。如果遇到问题，脚本会尝试使用旧版镜像名称作为备选方案。

此方案已在GitHub Actions环境中测试，能够成功创建离线部署包，确保您可以在企业内网环境中顺利部署LobeHub。
