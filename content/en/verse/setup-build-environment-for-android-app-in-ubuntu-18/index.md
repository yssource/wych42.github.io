+++
title = "Setup build environment for Android app in Ubuntu 18"
date = 2020-04-30T15:21:00+08:00
lastmod = 2020-05-17T18:48:33+08:00
tags = ["android", "ubuntu", "jdk"]
categories = ["Verse"]
draft = false
toc = true
+++

## Install sdk command-line tools {#install-sdk-command-line-tools}

```bash
sudo apt install unzip # ensure unzip installed
sudo mkdir -p /usr/local/android-sdk/tools
cd /tmp
wget https://dl.google.com/android/repository/commandlinetools-linux-6200805_latest.zip
unzip commandlinetools-linux-6200805_latest.zip
sudo mv tools/* /usr/local/android-sdk/tools/

# setup path, change ~/.bashrc to your shellrc
echo "export PATH=$PATH:/usr/local/android-sdk/tools/bin:/usr/local/android-sdk/platform-tools" >> ~/.bashrc
source ~/.bashrc
```


## Install oracle-jdk8 {#install-oracle-jdk8}

Skip this section if jdk8 is already set.

Oracle JDK8 will be installed via [ppa:kkeiichi/java](https://launchpad.net/~kkeiichi/+archive/ubuntu/java), jdk-8u241-linux-x64.tar.gz should be downloaded and uploaded to the server before continuing.

Download link: [Official Site](https://www.oracle.com/java/technologies/javase-jdk8-downloads.html) [DropboxBackup](https://www.dropbox.com/s/yth6qme9zmuv3ow/jdk-8u241-linux-x64.tar.gz?dl=0)(package install script will check file shasum)

```bash
sudo mkdir -p /var/cache/oracle-java8-installer-local
sudo cp jdk-8u241-linux-x64.tar.gz /var/cache/oracle-java8-installer-local/

sudo add-apt-repository ppa:kkeiichi/java
sudo atp update
sudo apt install oracle-java8-installer-local
sudo apt install oracle-java8-set-default-local # set default
```


## Install sdk {#install-sdk}

```bash
mkdir ~/.android
echo "export ANDROID_HOME=~/.android" >> ~/.bashrc
source ~/.bashrc

# accept linceses
yes | sdkmanager --sdk_root=$ANDROID_HOME --licenses
# download sdk
yes | sdkmanager --sdk_root=$ANDROID_HOME "platforms;android-28" "build-tools;28.0.3"
```