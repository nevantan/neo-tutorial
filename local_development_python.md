# Local Development of Neo Smart Contracts in Python

In this tutorial, we will use the [neo-local](https://github.com/CityOfZion/neo-local) project to setup a private chain for local development and testing of Neo smart contracts. Using a private chain gives us complete control over our environment, allowing us to work in isolation without dealing with external testnets.

In order to follow along, you will need access to a Unix-like terminal and a text editor of some sort. I will be working in a VM, using nano for text editing:

Ubuntu 18.04 (Minimal Install)  
4GB RAM  
50GB Disk

Note that you will probably want at least 20GB of disk space to store your private chain.

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

Your version numbers may not match mine exactly, that's fine so long as you're running the latest versions available to you.

Linux users should also be able to run the 'hello-world' docker container without `sudo` if the post-install steps worked. Note that you may have to restart your computer if you get a 'permission denied' error of some kind:
```bash
$ docker run hello-world
```

### Setup neo-local
Finally, we'll need to setup neo-local by cloning the repository and mounting the pre-configured wallet file so that we have GAS to work with on our private network.

In your terminal, navigate to the directory you want to store your Neo-related work, then clone the repository. For those unfamiliar with git, this will create a directory called 'neo-local' with the files we need inside. Navigate into the neo-local directory.

```bash
$ git clone https://github.com/CityOfZion/neo-local.git
$ cd neo-local
```

Most of the files in here are related to the neo-local project itself and we will not be modifying those at all. We will only be dealing with the `wallets/` directory during setup in a moment. Most of our work will be in the `smart-contracts/` directory once we're up and running.

The wallet file we'll be using is a bit difficult to locate. You can find it on the [neo-privatenet Docker hub page](https://hub.docker.com/r/cityofzion/neo-privatenet/), or just grab it from [this direct link](https://s3.amazonaws.com/neo-experiments/neo-privnet.wallet). Place the downloaded wallet file in the `wallets/` directory of the neo-local repository. For reference, the password for this wallet is `coz`.

### Start the neo-local stack
Starting the stack is slightly different between operating systems. Both sets of commands will leave you in a neo-python command-line interface (CLI).

#### Windows ([wiki](https://github.com/CityOfZion/neo-local/wiki/Usage-(Windows)))
```bash
$ docker-compose up -d --build --remove-orphans
$ docker exec -it neo-python np-prompt -p -v
```

#### MacOS ([wiki](https://github.com/CityOfZion/neo-local/wiki/Usage-(Mac))) and Linux ([wiki](https://github.com/CityOfZion/neo-local/wiki/Usage-(Linux)))

Install `make` if you don't already have it:

```bash
$ sudo apt install make
```

Start the stack:
```bash
$ make start
```

### Opening the wallet
The last setup step before working with smart contracts is opening the wallet we copied in earlier. Docker is set up to mount the `wallets/` directory at root, so our wallet resides at `/wallets/neo-privnet.wallet`. The `help` command in neo-python will show a list of possible commands. The one we're looking for is `open wallet {path}`, which will then prompt for a password for the wallet (`coz`). The whole thing should look as follows:

```
neo> open wallet /wallets/neo-privnet.wallet
[password]> ***
Opened wallet at /wallets/neo-privnet.wallet
neo> 
```

## A Basic Smart Contract
Since the neo-local stack uses neo-python, we'll be writing a basic smart contract in Python. Create the file `plus_one.py` in the `smart-contracts/` directory and add the following code:

```python
def Main(num):
  return num + 1;
```

As you can see, the contract takes an input number and returns that number plus one. Much like the `wallets/` directory, `smart-contracts/` is also mounted at root, so the path to our contract will be `/smart-contracts/plus_one.py`.

Neo smart contracts run on the NeoVM (Neo Virtual Machine) and must first be converted to bytecode, similar to Java, before they can be deployed. The `build {path}` command in neo-python does this for us, generating a `.avm` file alongside the `.py` file provided as input. However, this file on its own doesn't do us much good. We will want to start with the `build..test` variant of the command:

```
neo> build /smart-contracts/plus_one.py test 02 02 False False False 5
```

Let's break the command down before we get to the output. The full signature for this command is:

```
neo> build {path/to/file.py} test {param_types} {return_type} {needs_storage} {needs_dynamic_invoke} {is_payable} [params]
```

The path to file is fairly self-explanatory. Types, however, are provided as a [ContractParameterType](http://docs.neo.org/en-us/sc/Parameter.html) where parameter and return types are each represented as a single byte:

| Type             | Byte |
|------------------|------|
| Signature        | 00   |
| Boolean          | 01   |
| Integer          | 02   |
| Hash160          | 03   |
| Hash256          | 04   |
| ByteArray        | 05   |
| PublicKey        | 06   |
| String           | 07   |
| Array            | 10   |
| InteropInterface | f0   |
| Void             | ff   |

In our case, the contract takes an integer (02) and also returns an integer (02). The next three arguments are properties that are set on the contract, we aren't going to worry about those for now. The last argument is filling in the data that the contract will actually be called using. Running this command in the neo-python prompt should output something like the following:

```
neo> build /smart-contracts/plus_one.py test 02 02 False False False 5
[I 180909 22:53:38 BuildNRun:50] Saved output to /smart-contracts/plus_one.avm
[I 180909 22:53:38 Invoke:586] Used 0.021 Gas

-----------------------------------------------------------
Calling /smart-contracts/plus_one.py with arguments ['5']
Test deploy invoke successful
Used total of 19 operations
Result [{'type': 'Integer', 'value': 6}]
Invoke TX gas cost: 0.0001
-------------------------------------------------------------

neo>
```

The first output line is confirmation that the bytecode was built and where it was saved. The rest of the output describes the setup and result of the contract. We called it with the value of '5', the invoke was successful, and we received the Integer '6' back. Looks like our contract works!

## Deploying a Smart Contract
Finally, once we've built and tested our smart contract, we need to deploy it to the network. This is actually a fairly simple process at this point, we just need to use the `import contract` command. 

```
neo> import contract /smart-contracts/plus_one.avm 02 02 False False False
```

You will be prompted to fill in a few fields such as the Contract Name and Version, Author Name and Email, etc. Once those have been filled, some metadata about the contract will be printed and you will be prompted for your wallet password. Entering this password will deploy the contract and charge you the requisite quantity of GAS to do so. Since this is a private network, you should be fine to go ahead and deploy - the test wallet has plenty of GAS to work with.

Once the deployment goes through, grab the "hash" from the metadata that was printed and run a `testinvoke` command (substituting your own contract hash):

```
testinvoke 0x2b46bfe08185fbda2cb8121d6a2fd1a1d228c586 8
```

You should see the output with the expected result:

```
----------------------------------------------------------------
Test invoke successful
Total operations: 19
Results [{'type': 'Integer', 'value': '9'}]
Invoke TX GAS cost: 0.0
Invoke TX fee: 0.0001
----------------------------------------------------------------
```

If you get a message about the contract not being found, you may need to wait for a minute or two for your contract to be mined into a block before it can be invoked. Once again, this will be a local invocation and entering your password will run the contract on the network for real, charging you GAS in the process.

## Next Steps
As of the time of writing, Neo documentation is fairly fragmented. However, some of the following sites provide good info on smart contract development:

- [neo-python docs](https://neo-python.readthedocs.io/en/latest/index.html) ([Smart Contracts section](https://neo-python.readthedocs.io/en/latest/neo/SmartContract/smartcontracts.html))

- [Neo Docs](http://docs.neo.org/en-us/index.html) ([Smart Contracts section](http://docs.neo.org/en-us/sc/introduction.html))

Additionally, [neon-js](https://github.com/CityOfZion/neon-js) can be used to interact with smart contracts from JavaScript environments.