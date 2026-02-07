## Vagrant

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
