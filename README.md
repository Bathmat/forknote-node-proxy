# forknote-node-proxy

## Important note!

During testing, each pool needed to have `"default": true` in config.json to allow each version of util to work correctly.

## Setup Instructions

Based on a clean Ubuntu 16.04 LTS minimal install

## Deployment via Installer

1. Create a user 'nodeproxy' and assign a password (or add an SSH key. If you prefer that, you should already know how to do it)

```bash
useradd -d /home/nodeproxy -m -s /bin/bash nodeproxy
passwd nodeproxy
```

2. Add your user to `/etc/sudoers`, this must be done so the script can sudo up and do it's job.  We suggest passwordless sudo.  Suggested line: `<USER> ALL=(ALL) NOPASSWD:ALL`.  Our sample builds use: `nodeproxy ALL=(ALL) NOPASSWD:ALL`

```bash
echo "nodeproxy ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

3. Log in as the **NON-ROOT USER** you just created and run the [deploy script](https://raw.githubusercontent.com/Bathmat/forknote-node-proxy/master/install.sh).  This is very important!  This script will install the proxy to whatever user it's running under!

```bash
curl -L https://raw.githubusercontent.com/Bathmat/forknote-node-proxy/master/install.sh | bash
```

3. Once it's complete, copy `example_config.json` to `config.json` and edit as desired.
4. Run: `source ~/.bashrc`  This will activate NVM and get things working for the following pm2 steps.
8. Once you're happy with the settings, go ahead and start all the proxy daemon, commands follow.

```shell
cd ~/forknote-node-proxy/
pm2 start proxy.js --name=proxy --log-date-format="YYYY-MM-DD HH:mm Z"
pm2 save
```
You can check the status of your proxy by either issuing

```
pm2 logs proxy
```

or using the pm2 monitor

```
pm2 monit
```

## Known Issues

VMs with 512Mb or less RAM will need some swap space in order to compile the C extensions for node.  Bignum and the CN libraries can chew through some serious memory during compile.  In regards to this, one of our users has put together a guide for T2.Micro servers: https://docs.google.com/document/d/1m8E4_pDwKuFo0TnWJaO13LDHqOmbL6YrzyR6FvzqGgU (Credit goes to MayDay30 for his work with this!)

If not running on an Ubuntu 16.04 system, please make sure your kernel is at least 3.2 or higher, as older versions will not work for this.

Many smaller VMs come with ulimits set very low. We suggest looking into setting the ulimit higher. In particular, `nofile` (Number of files open) needs to be raised for high-usage instances.

If your system doesn't have AES-NI, then it will throw an error during the node-multi-hashing install, as this requires AES-NI.  If this is the case, go ahead and change the following line:
"multi-hashing": "git+https://github.com/Snipa22/node-multi-hashing-aesni.git",
to:
"multi-hashing": "git://github.com/clintar/node-multi-hashing.git#Nan-2.0",

In your `packages.json`, do a `npm install`, and it should pass.


## Performance

The proxy gains a massive boost over a basic pool by accepting that the majority of the hashes submitted _will not_ be valid (does not exceed the required difficulty of the pool).  Due to this, the proxy doesn't bother with attempting to validate the hash state nor value until the share difficulty exceeds the pool difficulty.

In testing, we've seen AWS t2.micro instances take upwards of 2k connections, while t2.small taking 6k.  The proxy is extremely light weight, and while there are more features on the way, it's our goal to keep the proxy as light weight as possible.

## Configuration Guidelines

Please check the [wiki](https://github.com/Snipa22/xmr-node-proxy/wiki/config_review) for information on configuration

## Installation/Configuration Assistance

Contact Bathmat if you have any issues.

## Known Working Pools

* [Bathmat's TurtleCoin Pool](http://turtle.bathmatminingpools.com)
* [Bathmat's LeviarCoin Pool](http://leviar.bathmatminingpools.com)
* [Bathmat's BitCoal Pool](http://coal.bathmatminingpools.com)
