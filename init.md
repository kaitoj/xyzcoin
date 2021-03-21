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

## Stage 4 
You are now ready to locate the Merkle root. This can be found on your virtual machine, in `~/.yourcoin/debug.log` and should be the last line in the file. Copy this number and paste it in the following line in main.cpp.

In src/main.cpp
---------------
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
`./yourcoind -testnet`

Expect to get an error at this point. As weith the Merkle hash, yo will need to locate the genesis block hash. This can be found on your virtual machine, in `~/.yourcoin/testnet3/debug.log`. At or near the end of the debug log file you should find three lines like the following;

```
2021-03-21 14:05:19 block.nTime = 1616329319
2021-03-21 14:05:19 block.nNonce = 2087006351
2021-03-21 14:05:19 block.GetHash = 2b379f242af728d7ad18df7d8b7f6d7ee0a987780fcdab56356c001ec0be6082
```
In src/main.cpp
---------------
Change the testnet nOnce and genesis block using the nNonce and the GetHash from the debug log file
`2801 block.nNonce   = 385270584;`
`2749 hashGenesisBlock = uint256("0x3acbbcc500475f31853c8912fda7fdcef59976bec5ba4ca5ab91da04ceab0048");`

Repeat the previous steps to obtain the main net nOnce and genesis hash.
`./yourcoind`

Expect to get an error at this point. As with the Merkle hash, you will need to locate the genesis block hash. This can be found on your virtual machine, in `~/.yourcoin/debug.log`. At or near the end of the debug log file you should find three lines like the following;

```
2021-03-21 14:05:19 block.nTime = 1616329319
2021-03-21 14:05:19 block.nNonce = 2087006351
2021-03-21 14:05:19 block.GetHash = 2b379f242af728d7ad18df7d8b7f6d7ee0a987780fcdab56356c001ec0be6082
```

In src/main.cpp
---------------
Change the testnet nOnce and genesis block using the nNonce and the GetHash from the debug log file
`2796 block.nNonce   = 385270584;`
`38 uint256 hashGenesisBlock("0x2b379f242af728d7ad18df7d8b7f6d7ee0a987780fcdab56356c001ec0be6082");`

## Stage 5

In src/checkpoints.cpp
----------------------

Change checkpoint map. Delete lines 39 - 54; change 1500 to 0 and replace the hash with the main net genesis hash created in Stage 4
```
37 boost::assign::map_list_of
        (  1500, uint256("0x841a2965955dd288cfa707a755d05a54e45f8bd476835ec9af4402a2b59a2967"))
        (  4032, uint256("0x9ce90e427198fc0ef05e5905ce3503725b80e26afd35a987965fd7e3d9cf0846"))
        (  8064, uint256("0xeb984353fc5190f210651f150c40b8a4bab9eeeff0b729fcb3987da694430d70"))
        ( 16128, uint256("0x602edf1859b7f9a6af809f1d9b0e6cb66fdc1d4d9dcd7a4bec03e12a1ccd153d"))
        ( 23420, uint256("0xd80fdf9ca81afd0bd2b2a90ac3a9fe547da58f2530ec874e978fce0b5101b507"))
        ( 50000, uint256("0x69dc37eb029b68f075a5012dcc0419c127672adb4f3a32882b2b3e71d07a20a6"))
        ( 80000, uint256("0x4fcb7c02f676a300503f49c764a89955a8f920b46a8cbecb4867182ecdb2e90a"))
        (120000, uint256("0xbd9d26924f05f6daa7f0155f32828ec89e8e29cee9e7121b026a7a3552ac6131"))
        (161500, uint256("0xdbe89880474f4bb4f75c227c77ba1cdc024991123b28b8418dbbf7798471ff43"))
        (179620, uint256("0x2ad9c65c990ac00426d18e446e0fd7be2ffa69e9a7dcb28358a50b2b78b9f709"))
        (240000, uint256("0x7140d1c4b4c2157ca217ee7636f24c9c73db39c4590c4e6eab2e3ea1555088aa"))
        (383640, uint256("0x2b6809f094a9215bafc65eb3f110a35127a34be94b7d0590a096c3f126c6f364"))
        (409004, uint256("0x487518d663d9f1fa08611d9395ad74d982b667fbdc0e77e9cf39b4f1355908a3"))
        (456000, uint256("0xbf34f71cc6366cd487930d06be22f897e34ca6a40501ac7d401be32456372004"))
        (541794, uint256("0x1cbccbe6920e7c258bbce1f26211084efb19764aa3224bec3f4320d77d6a2fd2"))
        (585010, uint256("0xea9ea06840de20a18a66acb07c9102ee6374ad2cbafc71794e576354fea5df2d"))
        (638902, uint256("0x15238656e8ec63d28de29a8c75fcf3a5819afc953dcd9cc45cecc53baec74f38"))
55      ;
```

Change timestamp to match the block.nTime in src/main.cpp
`42 1616329319, // * UNIX timestamp of last checkpoint block`

Change transactions to 0
`43 4896865,    // * total number of transactions between genesis and last checkpoint`

Change transactions after checkoint (optional)
`45 7000.0     // * estimated number of transactions per day after checkpoint`

Change block height to 0 and replace the hash with the testnet genesis block hash
`50 (   546, uint256("0xa0fea99a6897f531600c8ae53367b126824fd6a847b2b2b73817a95b8e27e602"))`

Change line 53 to the testnet block.nTime and line 54 to 0
```
52 static const CCheckpointData dataTestnet = {
        &mapCheckpointsTestnet,
        1365458829,
        547,
        576
57    };
```

This is a good time to commit and push all the changes you have made so far.