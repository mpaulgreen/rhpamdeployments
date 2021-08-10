- Create a folder 01-createCustomKierServerImage
- Copy GetTasksCustomAPI-1.0.jar into the above new folder
- Install Virtualbox
- Install vagrant
- Install podman ```brew install podman```
- Create a Vagrantfile

```
Vagrant.configure("2") do |config|
  config.vm.box = "generic/fedora33"
  config.vm.hostname = "fedora33"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end
  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"

  config.vm.provision "shell", inline: <<-SHELL
    rm -rf /home/vagrant/app
    mkdir -p /home/vagrant/app
  SHELL
  config.vm.provision "file", source: "Dockerfile", destination: "/tmp/"
  config.vm.provision "file", source: "GetTasksCustomAPI-1.0.jar", destination: "/tmp/"

  config.vm.provision "shell", inline: <<-SHELL
    cp /tmp/Dockerfile /tmp/GetTasksCustomAPI-1.0.jar /home/vagrant/app/
    chown -R vagrant /home/vagrant/app/
    yum install -y podman
    groupadd -f -r podman

    usermod -aG podman $SUDO_USER
    podman login -u='11009103|mpaul-3scale' \
    -p=eyJhbGciOiJSUzUxMiJ9.eyJzdWIiOiI5YWFiY2UyZGI0MGY0MDk2YThmNWY4MzUyN2M0NWFjMSJ9.Ius_tSHx_W2QfW5WIxyX4lRsGolJkegNKgbtk0nfmZ2tnAZ82kqI18pkMUMHkZCqiXj9sSn0DI7Vfc4hhns7yr5IecW1CW56XfZu9nA9GZS060kSvqFhcCrq6fDZamEEvZ7W5sq8Eu5D2eOA0KJ6ovm463f-UjjchkK-u4pAxPI_p0_NovmKq8y5SRC82BT0-k57DT_CA4Oa329hRaqsDH4pkSdI-mHpMkwrCjHYsBYsmlQaYfOsi6GeaohpdW4yAGjnoV0lZVZb_R3fkhp0tsvY9_h0uPJV5OTsR2Z3bUsCjGUjScrwzA2KjgqerdwnsAYahOgVCCX2oM9_CgTxHKat-XiZcNZQm_x9WduqPw2srGccIXUqZjJBysg2YYL8hznTdhUrTIiieJGG2ssZt9qG2Na0bpnHXW5VxacIEilUXkyaUX04W-310l6fgFuKa5JTW0_6dTs7OKNxp3trZc2mgMaQ3cIv5ScYvOr7iLmsO-pjjM2TJQ_wahPRgjVR7l5Ui7aZdoqhnxeLjPvg-0hWAvzEtTS5v03Sjj6l66GRxm40xnH0VBDpEHGcj4LTsm92-xJuHgoShGSrFm3yn4w2rtdgLQVrkANXtMDt88hA68noCA5h6N0HNr8c0twfMxcjeTw7Vi49xg7bX86dTRWVEeD-gRLeuY-d1dpwIjA registry.redhat.io
  SHELL

  config.vm.provision "podman" do |d|
    d.build_image "/home/vagrant/app",
      args: "-t quay.io/ecosystem-appeng/rhpam-kieserver-rhel8-custom:7.9.0"
  end
  config.vm.provision "shell", inline: <<-SHELL
    podman login -u="mpaulgreen" \
    -p="Andromeda@22" quay.io
    podman push quay.io/ecosystem-appeng/rhpam-kieserver-rhel8-custom:7.9.0
  SHELL
end
```

- Configire Vagrant to use current configuration
```
export VAGRANT_CWD=$PWD
export CONTAINER_HOST=ssh://vagrant@127.0.0.1:2222/run/podman/podman.sock
export CONTAINER_SSHKEY=$PWD/.vagrant/machines/default/virtualbox/private_key
```
- Startup vagrant ```vagrant up```
- ssh to the rhel box ```vagrant ssh```
- ```sudo podman images```