# rpi-microk8s
Requirements:

Mac computer for flashing
2x Raspberry Pi 4 8GB (4GB/2GB may work)

1. flash ubuntu server 22.04.2 LTS 64 bit

	set hostname
	enable SSH
	`ssh keygen` if you don't have an SSH key yet
	Allow public -key authentication only
	check "set username and password"
	use terminus

2. `sudo apt update && sudo apt upgrade && sudo apt install zsh neovim`
then reboot `sudo reboot`

3.
then get oh-my-zsh `sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`

`sudo nvim /boot/firmware/cmdline.txt`

add `cgroup_enable=memory cgroup_memory=1` to each pi

`sudo reboot`

4.
now install microk8s

`sudo snap install microk8s --classic`

enable microk8s group services on each pi:

`sudo usermod -a -G microk8s <user>`
`newgrp microk8s`

enable services one at a time:
`microk8s enable dns`
`microk8s enable dashboard`
`microk8s enable storage`
`microk8s enable ingress`

on master node: `microk8s add-node`

on master node, update the hosts before adding with the --worker command

`sudo nvim /etc/cloud/templates/hosts.debian.tmpl`

like:

`192.168.0.123 kubeNodeName` (output of `$ hostname`)

then copy to `/etc/hosts` on master node. Otherwise worker nodes cannot connect to the master node.

token is only good for a single attempt - run `microk8s add-node` again and for each pi

5.
now get to kubernetes dashboard:

expose the dashboard over nodeport to access the dashboard outside the cluster:

`microk8s.kubectl -n kube-system edit service kubernetes-dashboard`

change `type: ClusterIP` to `type: NodePort` and save the file. 

then `$ microk8s kubectl get all --all-namespaces` and check the port after 443:<port>

then go to `https://<lan IP>:port` and get the dashboard

now setup KubeConfig so you dont have to get token every time

First get gh cli to clone other OSS configs:

```
type -p curl >/dev/null || (sudo apt update && sudo apt install curl -y)
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```


