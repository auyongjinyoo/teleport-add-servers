## Add existing servers to teleport

### Create a new access token

To add another node to the proxy server, we need to download and run the teleport client on a server. But first, we need to create a new token on the auth server.

```bash
docker-compose exec teleport tctl nodes add
```
Please keep you token and ca-pin for later use.

### Install Teleport on your server

There is many way to onstall Teleport on your server, you may refer to the teleport website `https://goteleport.com/docs/`. They provided very details on the installation guide. 

For my scenario, i will just download the RPM file and perform the installation on Alma Linux. You may need to ensure the same version is install with the teleport server.

```bash
wget https://get.gravitational.com/teleport-10.2.1-1.x86_64.rpm

yum localinstall teleport-10.2.1-1.x86_64.rpm
```
### Create a new config for your server

After completed the installation, it will require to create config file. You may using below command to create the configuration file. If there is existing `/etc/teleport.yaml` , you may remove it first. 

```bash
teleport configure -o file
```
The command will auto generated `/etc/teleport.yaml`, but we will need make changes in order for the nodes join the cluster.

**Example teleport.yml**:
```yml
version: v2
teleport:
  nodename: <auto generated>
  data_dir: /var/lib/teleport
  auth_token: <your-auth-token-from-earlier-step>
  auth_servers:
        - <teleport server ip>:3025
  log:
    output: stderr
    severity: INFO
    format:
      output: text
  ca_pin: <your-ca-pin-from-earlier-step>
  diag_addr: ""
auth_service:
  enabled: "no"
  listen_addr: 0.0.0.0:3025
  proxy_listener_mode: multiplex
ssh_service:
  enabled: "yes"
  listen_addr: 0.0.0.0:3022
  commands:
  - name: hostname
    command: [hostname]
    period: 1m0s
proxy_service:
  enabled: "no"
  https_keypairs: []
  acme: {}

```
### Start the new server

Now you may start the teleport using command.

```bash
teleport start
```
If everything setup correctly, you will able to see on your web ui have showing the new nodes. 

You can also simply include teleport to daemon start up using below command.

```bash
sudo systemctl enable –now teleport
```

*Note: 
- You may encounter not able to establish the connection during the startup, you need to ensure the auth server is accessible.
- You may also encounter when click connect on the UI, show error not able dial or connect <ip>:3022. You will need add this port in your firewall rules.


