# brightpick-interview-task

## Installations

### VirtualBox

Install VirtualBox 7.2.6 following https://www.virtualbox.org/wiki/Downloads

### Vagrant

Download Vagrant by running the commands below on a debian based linux. Otherwise follow https://developer.hashicorp.com/vagrant/install#linux. and don't forget to install version 2.4.9.

```
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant=2.4.9-1
```

Verify the vagrant version running

```
vagrant -v
```

You should expect the following output

```
Vagrant 2.4.9
```

### KubeOne

Download the kubeone CLI from https://github.com/kubermatic/kubeone/releases/tag/v1.12.3

### TalosOS

Put aside the Talos OS on VirtualBox spawned by Vagrant due to the missing support for `config.vm.communicator = :none` https://github.com/hashicorp/vagrant/issues/13619
