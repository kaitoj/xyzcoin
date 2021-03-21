# Instructions
============
These steps have been tested in Ubuntu 18.04 Server; They should work in newer releases; however openssl1.0 and libssl1.0 are hard requirements. Newer releases of openssl and libssl will cause compile to fail.

## Stage 1 - Set up Virtual Machine (optional)
You can use any virtual machine software you prefer as long as you are able to use SSH to remotely log in to the machine. 
Execute the following commands on the virtual machine (not your host). Run these commands seperately.

`sudo apt update && sudo apt upgrade`

`sudo apt install software-properties-common && sudo add-apt-repository ppa:bitcoin/bitcoin`

`sudo apt update`

```
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
```

## Stage 2 - Prepare code environment
It is recommended that you use a good IDE. For example VSCode. This will make editing and updating your changes easier. Most IDE's have a marketplace for extentions. It is recommended that you install code extentiosn for C++ as a minimum.

On your host machine clone or fork (in github) the following repository. 

`git clone -b 0.8 https://github.com/litecoin-project/litecoin.git`

If you cloned the repository; go to your github account and create a new repository; then clone the new repository to your host. Copy the contents of the litecoin folder (excluding the .git files are folders) into your new local repository. All modifications will be done in the new repository. This is a good time to make your first commit.

## Stage 3 - Edit Source
It is now time to start editing the source code. Before you start editing you should generate some keys. You will need these later. Execute the following commands. Do not do this in your repository folder. Or at least if you do, add the resulting files to your .gitignore file.

```
openssl ecparam -genkey -name secp256k1 -out alertkey.pem
openssl ec -in alertkey.pem -text > alertkey.hex
openssl ecparam -genkey -name secp256k1 -out testnetalert.pem
openssl ec -in testnetalert.pem -text > testnetalert.hex
openssl ecparam -genkey -name secp256k1 -out genesiscoinbase.pem
openssl ec -in testnetalert.pem -text > genesiscoinbase.hex
```

You will need to replace all instances of Litecoin with your coin name. This can be done in your IDE easily. Alternatively you can run following commands from your coin source folder.

`cd path/to/your/coin`
`find . -type f -print0 | xargs -0 sed -i 's/litecoin/yourcoin/g'`
`find . -type f -print0 | xargs -0 sed -i 's/Litecoin/Yourcoin/g'`
`find . -type f -print0 | xargs -0 sed -i 's/LiteCoin/YourCoin/g'`
`find . -type f -print0 | xargs -0 sed -i 's/LITECOIN/YOURCOIN/g'`
`find . -type f -print0 | xargs -0 sed -i 's/LTC/URC/g'`
`find . -type f -print0 | xargs -0 sed -i 's/9333/5512/g'`
`find . -type f -print0 | xargs -0 sed -i 's/9332/5511/g'`

In src/main.cpp
---------------
Change block reward 
`1090 int64 nSubsidy = 50 * COIN;`
`2787 txNew.vout[0].nValue = 50 * COIN;`

Change reward halving
`1093 nSubsidy >>= (nHeight / 840000); // Xyzcoin: 840k blocks in ~4 years`

Change
``` 
1098 static const int64 nTargetTimespan = 3.5 * 24 * 60 * 60; // Xyzcoin: 3.5 days
1099 static const int64 nTargetSpacing = 2.5 * 60; // Xyzcoin: 2.5 minutes
```

Change hash Merkle root. At this stage remove everything between "" leaving only "0x"
`2809 assert(block.hashMerkleRoot == uint256("0x97ddfbbae6be97fd6cdf3e7ca13232a3afff2353e29badfab7f73011edd4ced9"));`

Change nTime epoch
`2794 block.nTime    = 1317972665;`
`2800 block.nTime    = 1317798646;`

Change 
`2782 const char* pszTimestamp = "NY Times 05/Oct/2011 Steve Jobs, Apple’s Visionary, Dies at 56";`

In src/main.h
---------------
Change coin supply
`55 static const int64 MAX_MONEY = 84000000 * COIN;`

Change coin maturity
`58 static const int COINBASE_MATURITY = 100;`

Change lifespan
`627 return dPriority > COIN * 576 / 250;`

Finally you need to commit your changes. At this stage you should be ready to compile your code for the first time. To do this you will need to log into the virtual machine via SSH (if you are using a virtual machine.)

From the command prompt clone your repository
`git clone https://github.com/kaitoj/xyzcoin.git`

then
`cd yourcoin/src && make -f makefile.unix`

As long as you have no errors you will now find a file called `yourcoind` in the current folder.

Execute this file
`./yourcoind`

Expect to get an error at this point.

### Stage 4 
You are now ready to locate the Merkle root. This can be found on your virtual machine, in `~/.yourcoin/debug.log` and should be the last line in the file. Copy this number and paste it in the following line in main.cpp.

Change hash Merkle root. 
`2809 assert(block.hashMerkleRoot == uint256("0x97ddfbbae6be97fd6cdf3e7ca13232a3afff2353e29badfab7f73011edd4ced9"));`

Reset the Genesis block. Remove everything between "" leaving just "0x"
`2749   hashGenesisBlock = uint256("0xf5ae71e26c74beacc88382716aced69cddf3dffff24f384e1808905e0188f68f");`
`38     uint256 hashGenesisBlock("0x12a765e31ffd4059bada1e25190f6e98c99d9714d334efa41a195a7e7e04bfe2");`

Commit and push your changes to your repository. Then on your virtual machine execute the following commands.
`cd ~ && rm -R .yourcoin`
`cd ~/yourcoin && git pull`
`cd yourcoin/src && make -f makefile.unix`

As long as you have no errors you will now find a file called `yourcoind` in the current folder.

Execute this file
`./yourcoind`

Expect to get an error at this point.
