* Install the prerequisites:

```
sudo apt-get install git-core gcc make
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

**WARNING!** Do not use *-j* option when running make. It will break everything! See #11 for the details.
The output is *./bin/gShield/gShield_flash.bin*. 

After that, you will likely want to flash the firmware. 
- [[Flashing G2 with Linux]].
