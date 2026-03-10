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
