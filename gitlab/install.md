# Install and configure Gitlab
In this Gitlab server and Gitlab Runner would be installed on Ubuntu server.
## 1. Setup Gitlab Server (ce)
- 1. Install dependencies
```bash
sudo apt update
sudo apt install ca-certificates curl openssh-server postfix tzdata perl curl
```
- 2. Install Gitlab
Refer to this [official documentation](https://docs.gitlab.com/install/package/ubuntu/), here step to install gitlab server
```bash
# Configure firewall
sudo systemctl enable --now ssh
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# Add the GitLab package repository
curl "https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh" | sudo bash

# Install the package
sudo EXTERNAL_URL="https://gitlab.example.com" apt install gitlab-ce
```
GitLab generates a random password and email address for the root administrator account stored in /etc/gitlab/initial_root_password for 24 hours. After 24 hours, this file is automatically removed for security reasons.

- 3. Reconfigure (Optional, if needed)
If you forget or need to change external_url after instalation, you can edit file configuration at **/etc/gitlab/gitlab.rb**
```bash
external_url 'http://YOUR_PUBLIC_IP_OR_REGISTERED_DOMAIN'
```

- 4. Rerunning gitlab server
```bash
sudo gitlab-ctl reconfigure
```

- 5. Access gitlab webserver
acces gitlab webserver by **http://YOUR_PUBLIC_IP_OR_REGISTERED_DOMAIN**, with user **root** and temporary password that you can see by execute this command
```bash
sudo cat /etc/gitlab/initial_root_password
```
*Note : This is temporary password, and the file will be automatic delete after 24 hours.

## 2. Setup Gitlab Runner
- Gitlab webserver based
    - Go to admin
    - CI/CD -> Runners
    - Create instance runner
    - Fill form (specially tags)
    - Create runner
    - Register runner (choose platform that will be runner, here i would use linux)

- Ubuntu server based (Gitlab runner server)
Refer to this [official documentation](https://docs.gitlab.com/runner/install/linux-repository), here step to install gitlab runner
1. Add the official GitLab repository :
```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
```
2. Install GitLab runner
```bash
sudo apt install gitlab-runner
```
3. Register the runner
```bash
gitlab-runner register  --url http://YOUR_PUBLIC_IP_OR_REGISTERED_DOMAIN  --token glrt-anytokengeneratedb.01.121j86j80
```
You will get this command after you Register runner in Gitlab webserver based