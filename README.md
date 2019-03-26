```yaml
#cloud-config
hostname: servidor
manage_etc_hosts: true

users:
  - name: ops
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    shell: /bin/bash
    plain_text_passwd: felizcumpledocker
    lock_passwd: false
    ssh_pwauth: true
    chpasswd: { expire: false }
    groups: sudo

package_update: true
package_upgrade: false

packages:
  - ntp
  - nfs-common

locale: "en_US.UTF-8"

timezone: "America/La_Paz"

runcmd:
  - systemctl disable docker
  - systemctl stop docker
  - sysctl -w net.ipv6.conf.all.disable_ipv6=1
  - sysctl -w net.ipv6.conf.default.disable_ipv6=1
  - echo "Port 4444" >> /etc/ssh/sshd_config
  - systemctl restart sshd
  - curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--no-deploy traefik --no-deploy servicelb --cluster-secret=felizcumpledocker --write-kubeconfig /home/ops/kubeconfig" sh -
  - systemctl enable k3s
```

## Download Flash tool from Hypriot

```sh
curl -LO https://github.com/hypriot/flash/releases/download/2.3.0/flash
chmod +x flash
sudo mv flash /usr/local/bin/flash
```

## servidor-master

Connect your SD card

	flash --userdata ./user-data.yaml
	
Insert the SD card on the RPi and boot the device.

Wait around 5 minutes.

	scp -P 4444 ops@servidor.local:~/kubeconfig ./
	sed -i'' -e "s/localhost/servidor.local/g" kubeconfig
	export KUBECONFIG=$PWD/kubeconfig
	kubectl get pods -o wide --all-namespaces

## servidor-worker

	sed -i'' -e "s/localhost/servidor.local/g" servidor-worker.yaml
	flash --userdata ./servidor-worker.yaml
	


	