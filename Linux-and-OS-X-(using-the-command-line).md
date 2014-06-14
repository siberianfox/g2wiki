1. Install prerequisites:

    ```sudo apt-get install git-core gcc make```

2. Checkout TinyG2 source:
    ```
    mkdir -p ~/src/github.com/synthetos
    cd ~/src/github.com/synthetos
    git clone https://github.com/synthetos/g2
    cd g2

3. Choose the right branch.
    Note: as of now, ```master``` branch is very outdated. There's no stable version available, but the brave souls can try ```edge``` branch.
    
    ```git checkout edge```

4. Build the sources:

    ```
    cd TinyG2
    make TinyG2.hex -j4
    ```

After that, you will likely want to flash the firmware. See [[Flashing G2 with Linux]].