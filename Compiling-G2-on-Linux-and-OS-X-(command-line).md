* Install the prerequisites:

```
sudo apt-get install git-core gcc make
```

For a 64bit Linux you might also need lib32z1 because the gcc-arm-none-eabi package in /Tools is a 32bit binary:

```
sudo apt-get install lib32z1
```

* Checkout TinyG2 source
as of now, *master* branch is very outdated. There's no stable version available, but the brave souls may try *edge* branch.

```
mkdir -p ~/src/github.com/synthetos
cd ~/src/github.com/synthetos
git clone https://github.com/synthetos/g2 g2
cd ./g2
git checkout edge
```

To clone the edge branch directly you can run the following command:

```
git clone -b edge git@github.com:synthetos/g2.git
```

* Build the sources:

```
cd g2/TinyG2
make
```

If you get the error which breaks at ``include <type_traits>`` you probably use an incompatible version of ``gcc-arm-none-eabi``. At this time, to compile G2 it's best to use the gcc-arm-none-eabi binaries which are downloaded in the Tools directory or downgrade to 4.9.1.

Do not use *-j* option when running make. It will break everything. See #11 for the details.
The output is *./bin/gShield/gShield_flash.bin*. 

After compiling, you will likely want to flash the firmware. 
- [[Flashing G2 with Linux]].
- [[Flashing G2 with OSX]].
