# Docker Ubuntu Install
```
# 卸载老版本软件
sudo apt-get remove docker docker-engine docker.io containerd runc

# 安装依赖包
sudo apt-get install ca-certificates curl gnupg lsb-release

# 增加Docker官方GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 设置稳定版仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装docker引擎
sudo apt-get install docker-ce docker-ce-cli containerd.io

# 当前用户加入docker组
sudo groupadd docker
sudo usermod -aG docker xxUser
sudo systemctl restart docker
su root
su ervin
sudo passwd

# 验证docker
docker -v
```