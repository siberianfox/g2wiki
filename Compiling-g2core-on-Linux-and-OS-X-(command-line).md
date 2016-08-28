# What's Needed

* Linux (Ubuntu-based)
  ```bash
  sudo apt-get install git-core make
  ```

* OS X - If you don't already have Xcode command line tools installed (it doesn't hurt to run it again):
  ```bash
  xcode-select --install
  ```

# Cloning the repo

  * See [Getting Started with G2core → Cloning the Repo](https://github.com/synthetos/g2_private/wiki/Getting-Started-with-G2core#cloning-the-repo) if you haven't already.

# Build the sources:

```
cd g2/g2core
make CONFIG=TestV9
```

* See [Adding and Revising Boards](Adding-and-Revising-Boards) for explanation of the available `make` options.

# Flashing and Debugging

- See [[Flashing g2core with Linux]].