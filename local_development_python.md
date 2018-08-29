# Local Development of NEO Smart Contracts in Python

In this tutorial, we will use the [neo-local](https://github.com/CityOfZion/neo-local) project to setup a private chain for local development and testing of NEO smart contracts. Using a private chain gives us complete control over our environment, allowing us to work in isolation without dealing with external testnets.

In order to follow along, you will need access to a Unix-like terminal and either a programmer text editor or a Python IDE. I will be working from an Ubuntu 18.04 setup using [Sublime Text 3](https://www.sublimetext.com/) (other good editor options include [Atom](https://atom.io/) or [Visual Studio Code](https://code.visualstudio.com/)).

## Docker, Docker Compose, and neo-local
The neo-local project requires Docker to run, so the first order of business will be to get that installed. Docker is a container engine that can run pre-configured setups, which is exactly what neo-local uses it for. We will be using Docker Community Edition (Docker CE).

### Install Docker
You can find detailed installation instructions for your operating system of choice in the [Docker docs](https://docs.docker.com/install/). Here are direct links to the more common OS options:

[Windows](https://docs.docker.com/docker-for-windows/install/)

[MacOS](https://docs.docker.com/docker-for-mac/install/)

[Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/) ([Post-installtion steps for Linux](https://docs.docker.com/install/linux/linux-postinstall/) - follow these too)

### Install Docker Compose
We will also need Docker Compose. For Windows and MacOS, it should already be included in the package you installed in the previous step. Linux users will need to follow the steps outlined in the [compose section of the docs](https://docs.docker.com/compose/install/). Make sure you check the [GitHub releases page](https://github.com/docker/compose/releases) for the latest version number as instructed in the guide - don't just copy/paste the command without checking.

### Testing Docker
Just as a quick test, you should now be able to run the following commands (the lines beginning in '$') and see the corresponding output:
```bash
$ docker --version
Docker version 18.06.0-ce, build 0ffa825
$ docker-compose --version
docker-compose version 1.22.0, build f46880fe
```

Your version numbers may not match mine exactly, that's fine.

### Setup neo-local
Finally, we'll need to setup neo-local by cloning the repository and mounting the pre-configured wallet file so that we have GAS to work with on our private network.

In your terminal, navigate to the directory you want to store your NEO-related work, then clone the repository. For those unfamiliar with git, this will create a directory called 'neo-local' with the files we need inside. Navigate into the neo-local directory.

```bash
$ git clone https://github.com/CityOfZion/neo-local.git
$ cd neo-local
```

Most of the files in here are related to the neo-local project itself and we will not be modifying those at all. We will only be dealing with the `wallets` directory during setup in a moment. Most of our work will be in the `smart-contracts` directory once we're up and running.

The wallet file we'll be using is a little hidden, you can find it on the [neo-privatenet Docker hub page](https://hub.docker.com/r/cityofzion/neo-privatenet/), or just grab it from [this direct link](https://s3.amazonaws.com/neo-experiments/neo-privnet.wallet). Place the downloaded wallet file in the `wallets` directory of the neo-local repository. I'll mention it again later when we need it but, for reference, the password for this wallet is `coz`.

### Start the neo-local stack
Starting the stack is slightly different between operating systems. Both sets of commands will leave you in a neo-python command-line interface (CLI).

#### Windows ([neo-local wiki](https://github.com/CityOfZion/neo-local/wiki/Usage-(Windows)))
```bash
$ docker-compose up -d --build --remove-orphans
$ docker exec -it neo-python np-prompt -p -v
```

#### Mac ([neo-local wiki](https://github.com/CityOfZion/neo-local/wiki/Usage-(Mac))) and Linux ([neo-local wiki](https://github.com/CityOfZion/neo-local/wiki/Usage-(Linux)))

Install `make` if you don't already have it:

```bash
$ sudo apt install make
```

Start the stack:
```bash
$ make start
```

These commands should leave 