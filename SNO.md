# Installing SNO node by using Agent Based installer.
Download binaries and extract the binaries
```bash
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.18.9/openshift-client-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.18.9/openshift-install-linux.tar.gz
tar -xvf openshift-client-linux.tar.gz -C /usr/bin
tar -xvf openshift-install-linux.tar.gz -C /usr/bin
```
Install nmstate dependency
```bash
sudo dnf install /usr/bin/nmstatectl -y
```
Create a folder
```bash
mkdir ~/agent-based;cd ~/agent-based
```
Create the install-config.yaml
```bash
apiVersion: v1
baseDomain: ab.example.com
metadata:
  name: sno
controlPlane:
  name: master
  replicas: 1
  architecture: amd64
compute: 
- name: worker
  replicas: 0 
platform:
  none: {}
pullSecret: ''
sshKey: |
  ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAwfGoU592TWMgtB1INQhViagxPWh8dT5uAQjTiX0P karan@192.168.1.29
```
Create agent-config.yaml
```bash
apiVersion: v1alpha1
kind: AgentConfig
metadata:
  name: sno-cluster
rendezvousIP: 192.168.122.149
hosts:
  - hostname: master-1
    interfaces:
      - name: enp1s0
        macAddress: 52:54:00:05:d9:68
    rootDeviceHints:
      deviceName: /dev/vda
    networkConfig:
      interfaces:
        - name: enp1s0
          type: ethernet
          state: up
          mac-address: 52:54:00:05:d9:68
          ipv4:
            enabled: true
            address:
              - ip: 192.168.122.149
                prefix-length: 23
            dhcp: false
      dns-resolver:
        config:
          server:
            - 192.168.122.1
      routes:
        config:
          - destination: 0.0.0.0/0
            next-hop-address: 192.168.122.1
            next-hop-interface: enp1s0
            table-id: 254
```
Create image
```bash
cd ~/agent-based
openshift-install agent create image
```
Check the created agent ISO agent.x86_64.iso and attach the Node and boot the Node.
```
ll ~/agent-based/agent.x86_64.iso
```
To check the installation progress of node, run below on the node where you generate the agent ISO
```bash
openshift-install --dir ~/agent-based agent wait-for bootstrap-complete --log-level=info
```
Or SSH the Node and then check the live status of assisted service with journalctl
```
ssh core@192.168.122.149
journalctl -u assisted-service.service
```
