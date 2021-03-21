#Instructions
============
These steps have been tested in Ubuntu 18.04 Server; They should work in newer releases; however openssl1.0 and libssl1.0 are hard requirements. Newer releases of openssl and libssl will cause compile to fail.

##Stage 1 - Set up Virtual Machine (optional)
-------
You can use any virtual machine software you prefer as long as you are able to use SSH to remotely log in to the machine. 
Execute the following commands on the virtual machine (not your host).

`sudo apt update && sudo apt upgrade`

`sudo apt install software-properties-common && sudo add-apt-repository ppa:bitcoin/bitcoin`

`sudo apt update`

```
{
sudo apt install -y \
    git \
    build-essential \
    gcc \
    make \
    perl \
    dkms \
    libtool \
    autotools-dev \
    automake \
    pkg-config \
    libssl-dev \
    libevent-dev \
    bsdmainutils \
    libboost-system-dev \
    libboost-filesystem-dev \
    libboost-chrono-dev \
    libboost-program-options-dev \
    libboost-test-dev \
    libboost-thread-dev \
    libboost-all-dev \
    libdb4.8-dev \
    libdb4.8++-dev \
    libminiupnpc-dev \
    libzmq3-dev \
    libqt5gui5 \
    libqt5core5a \
    libqt5dbus5 \
    qttools5-dev \
    qttools5-dev-tools \
    libprotobuf-dev \
    protobuf-compiler \ 
    libqt4-dev \
    libprotobuf-dev \
    protobuf-compiler \
    openssl1.0 \
    libssl1.0-dev
}
```
 
git clone -b 0.8 https://github.com/litecoin-project/litecoin.git
 
find . -type f -print0 | xargs -0 sed -i 's/litecoin/technocoin/g'
find . -type f -print0 | xargs -0 sed -i 's/Litecoin/Technocoin/g'
find . -type f -print0 | xargs -0 sed -i 's/LiteCoin/TechnoCoin/g'
find . -type f -print0 | xargs -0 sed -i 's/LITECOIN/TECHNOCOIN/g'
find . -type f -print0 | xargs -0 sed -i 's/LTC/VISION/g'
find . -type f -print0 | xargs -0 sed -i 's/9333/2333/g'
find . -type f -print0 | xargs -0 sed -i 's/9332/2332/g'
 
openssl ecparam -genkey -name secp256k1 -out alertkey.pem
openssl ec -in alertkey.pem -text > alertkey.hex
openssl ecparam -genkey -name secp256k1 -out testnetalert.pem
openssl ec -in testnetalert.pem -text > testnetalert.hex
openssl ecparam -genkey -name secp256k1 -out genesiscoinbase.pem
openssl ec -in testnetalert.pem -text > genesiscoinbase.hex