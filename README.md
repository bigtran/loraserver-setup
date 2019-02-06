# LoRa Server 安装配置

这个repo提供了  [LoRa Server](https://www.loraserver.io/) 安装配置用的 [Ansible](https://www.ansible.com) 操作手册，包括项目所用到的一些依赖。通过 [Vagrant](https://www.vagrant.com) 文件，LoRa Server 还可以在本地部署（比如 [VirtualBox](https://www.virtualbox.org))。

包括：

* 配置防火墙策略 firewall rules (iptables)
* 配置MQTT代理、连接凭据 Mosquitto (MQTT broker) + connection credentials
* 配置 Redis
* 配置 PostgreSQL，创建数据库和用户
* 安装配置 [LoRa Gateway Bridge](https://www.loraserver.io/lora-gateway-bridge/)
* 安装配置 NS 网络服务器 [LoRa Server](https://www.loraserver.io/loraserver/)
* 安装配置 AS 应用服务器 [LoRa App Server](https://www.loraserver.io/lora-app-server/)
* 安装配置 GS 地理服务器 [LoRa Geo Server](https://www.loraserver.io/lora-geo-server/)
* 从 [Let's Encrypt](https://letsencrypt.org) 获取 HTTPS 证书

## Vagrant (使用VirtualBox的本地环境)

通过包含的 `Vagrantfile` ，可以生成一个包含了最新 LoRa Server 的 Ubuntu 16.04 的虚拟机。
并且能转发以下端口到你的主机：

* `8080`: LoRa App Server 应用服务器的界面和接口
* `1700`: packet-forwarder 数据的UDP监听
* `1883`: Mosquitto MQTT
* `1884`: Mosquitto Websockets

Note: 使用Vagrant时，不需要安装 Ansible（自动安装到Vagrant机器）。

### 环境要求 Requirements

配置LoRa Server 网络服务器环境的时候，确保

* 安装了最新的[Vagrant](https://www.vagrantup.com) 。
* 安装了 [VirtualBox](https://www.virtualbox.org)，以及 [VirtualBox Extension Pack](https://www.virtualbox.org/wiki/Downloads).

### 起步 Getting started

1. 更新 `roles/loraserver/templates/loraserver.toml`，使得`network_server.band.name` 匹配 LoRaWAN 频段。
   根据所选择的频段，同时更新 `network_server.network_settings`下的其他设置。

2. 在本repo的根目录下，执行指令：
    
    ```bash
    vagrant up
    ```

    这样就可以引入 Vagrant box，安装所有关联文件。

3. 配置基站（LoRa Gateway），让其NS指向你本机的IP，端口为 `1700`。

4. 打开浏览器，访问 http://localhost:8080/ 。（因为用的是自我签名的证书，浏览器可能会提示证书不被信任，不影响测试）。

5. 更新 Vagrant 环境（例如：更新配置文件、安装包，执行如下指令）：

    ```bash
    vagrant provision
    ```

6. 其他有用指令：

   ```bash
   # stop the vagrant machine
   vagrant halt 

   # restart the vagrant machine
   vagrant reload

   # ssh into the vagrant machine
   vagrant ssh

   # destroy the vagrant machine
   vagrant destroy
   ```

## 远程部署

本手册在 [DigitalOcean.com](https://m.do.co/c/6cd86e9f1cb8) 上测试过，bare-metal, AWS, ...应该也没问题。

### 环境要求 Requirements

运行Ansible Playbook的电脑，需要安装 Ansible 2.1+. 
也可以用pip 或 brew 安装。
* pip (`pip install ansible`)
* Homebrew (OS X) (`brew install ansible`)
详情具体参考 [Ansible installation guide](http://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

本 Ansible playbook在下列环境下测试过:

* Debian
    * Stretch (9.x)

* Ubuntu
    * Xenial (16.04.x LTS)
    * Bionic (18.04.x LTS)

### 配置 Configuration

1. 创建一个 Ubuntu 16.04.x 虚拟机，并且保证能安装了Ansible的主机可以远程访问 (例如： `ssh user@ip`)。

2. 如果需要配置 LetsEncrypt，需要为目标主机配置一条DNS记录，等待 DNS记录能被解析到你的IP。
   不用LetsEncrypt (
   `accept_letsencrypt_tos: False` in `single_server.yml`)，可以跳过。

3. 复制本repo的  `inventory.example` 到 `inventory`，并且用第二步配置的域名替换 `example.com`。

4. 复制本repo的 `group_vars/single_server.example.yml` 到 `group_vars/single_server.yml`，按需修改配置。

5. 更新LoRa Gateway Bridge, LoRa App Server and LoRa Server 配置文件：

   * `roles/lora-gateway-bridge/templates/lora-gateway-bridge.toml`
   * `roles/lora-app-server/templates/lora-app-server.toml`
   * `roles/loraserver/templates/loraserver.toml`

更多参考，详见:

* https://www.loraserver.io/lora-gateway-bridge/
* https://www.loraserver.io/loraserver/
* https://www.loraserver.io/lora-app-server/

### 提前准备 Provisioning

在本机运行以下指令，以部署 LoRa Server 到目标主机，以及升级到最新版本，更新最新配置：

```bash
ansible-playbook -i inventory full_deploy.yml
```

安装完成后，管理后台可以通过以下链接访问 
http://yourdomain.com:8080/.


## 修改记录 Changelog (playbook changes)

### 2018-10-30

* LoRa App Server default configuration now uses http (no TLS certificate).

### 2018-09-17

* Updated playbook to support Ubuntu 18.04.x, 16.04.x and Debian Stretch (9.x).
* `postgresql` package is always installed from distribution repository.
* `mosquitto` package is always installed from distribution repository.
* Added installation of LoRa Geo Server service.

### 2018-07-30

* Change to LoRa Server v2 apt repository.

### 2018-04-22

* Remove `mosquitto-auth-plug` setup (which was causing a lot of issues)
* Update configuration `.yml` files under `group_vars` and `host_vars` (for Vagrant)

### 2018-02-22

* Include LoRa Gateway Bridge, LoRa App Server and LoRa Server configuration
  files as templates. See **Configuration**.

### 2017-12-16

* `auth_opt_aclquery` query of the mosquitto-auth-plug has been updated
  as application users have been deprecated.

### 2017-07-26

* `GW_SERVER_JWT_SECRET` configuration option has been added to the example
  configuration file, which will be mandatory for
  [LoRa Server](https://docs.loraserver.io/) 0.20.0.

* Port `8002` (used by [LoRa Gateway Config](https://docs.loraserver.io/lora-gateway-config/))
  has been added as public accessible port in the example configuration.

* `letsencrypt` cli has been changed to `certbot` cli (as per installation
  instructions documented at https://certbot.eff.org).

### 2017-06-20

* Mosquitto authentication / authorization has been added (using
  [mosquitto-auth-plug](https://github.com/jpmens/mosquitto-auth-plug)).
  The `loraserver_hosts.example.yml` has been updated with example
  configuration. Note that anonymous connections will be rejected. This allows
  users to connect to the MQTT broker using their LoRa App Server credentials.

### 2017-03-28

* PostgreSQL 9.6 will now be installed from the [PostgreSQL deb repository](https://www.postgresql.org/download/).
  In case you're upgrading, make sure to migrate your data.

* Mosquitto will be now be installed from either the Mosquitto PPA or
  the [Mosquitto deb repository](https://mosquitto.org/download/).
