node-cryptonote-pool
====================

High performance Node.js (with native C addons) mining pool for CryptoNote based coins such as Bytecoin, Monero, QuazarCoin, HoneyPenny, etc..
Comes with lightweight example front-end script which uses the pool's AJAX API.



#### Table of Contents
* [Features](#features)
* [Community Support](#community--support)
* [Pools Using This Software](#pools-using-this-software)
* [Usage](#usage)
  * [Requirements](#requirements)
  * [Downloading & Installing](#1-downloading--installing)
  * [Configuration](#2-configuration)
  * [Configure Easyminer](#3-optional-configure-cryptonote-easy-miner-for-your-pool)
  * [Starting the Pool](#4-start-the-pool)
  * [Host the front-end](#5-host-the-front-end)
  * [Customizing your website](#6-customize-your-website)
  * [Upgrading](#upgrading)
* [Setting up Testnet](#setting-up-testnet)
* [JSON-RPC Commands from CLI](#json-rpc-commands-from-cli)
* [Monitoring Your Pool](#monitoring-your-pool)
* [Donations](#donations)
* [Credits](#credits)
* [License](#license)


#### Features

* TCP (stratum-like) protocol for server-push based jobs
  * Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Payment processing
  * Splintered transactions to deal with max transaction size
  * Minimum payment threshold before balance will be paid out
  * Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
* Detailed logging
* Ability to configure multiple ports - each with their own difficulty
* Variable difficulty / share limiter
* Share trust algorithm to reduce share validation hashing CPU load
* Clustering for vertical scaling
* Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
* Live stats API (using AJAX long polling with CORS)
  * Currency network/block difficulty
  * Current block height
  * Network hashrate
  * Pool hashrate
  * Each miners' individual stats (hashrate, shares submitted, pending balance, total paid, etc)
  * Blocks found (pending, confirmed, and orphaned)
* An easily extendable, responsive, light-weight front-end using API to display data
* Worker login validation (make sure miners are using proper wallet addresses for mining)


### Community / Support

* [CryptoNote Forum](https://forum.cryptonote.org/)
* [Bytecoin Github](https://github.com/amjuarez/bytecoin)
* [Monero Github](https://github.com/monero-project/bitmonero)
* [Monero Announcement Thread](https://bitcointalk.org/index.php?topic=583449.0)
* [Boolberry Forum](http://boolberry.com/forum/)
* [Boolberry Github](https://github.com/cryptozoidberg/boolberry)
* IRC (freenode)
  * Support / general discussion join #monero: https://webchat.freenode.net/?channels=#monero, https://webchat.freenode.net/?channels=#boolberry
  * Development discussion join #monero-dev: https://webchat.freenode.net/?channels=#monero-dev

#### Pools Using This Software

* http://pool.cryptoescrow.eu
* http://extremepool.org
* http://xminingpool.com
* http://xmr.poolto.be
* http://moneropool.com
* http://extremehash.com
* http://hashinvest.net
* http://moneropool.com.br
* http://monerominers.net
* http://monero.crypto-pool.fr
* http://cryptonotepool.org.uk
* http://minexmr.com
* http://kippo.eu
* http://coinmine.pl/xmr
* http://moneropool.org

Usage
===

#### Requirements
* Coin daemon(s) (find the coin's repo and build latest version from source)
* [Node.js](http://nodejs.org/) v0.10+ ([follow these installation instructions](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager))
* [Redis](http://redis.io/) key-value store v2.6+ ([follow these instructions](http://redis.io/topics/quickstart))
* libssl required for the node-multi-hashing module
  * For Ubuntu: `sudo apt-get install libssl-dev`


##### Seriously
Those are legitimate requirements. If you use old versions of Node.js or Redis that may come with your system package manager then you will have problems. Follow the linked instructions to get the last stable versions.


[**Redis security warning**](http://redis.io/topics/security): be sure firewall access to redis - an easy way is to
include `bind 127.0.0.1` in your `redis.conf` file. Also it's a good idea to learn about and understand software that
you are using - a good place to start with redis is [data persistence](http://redis.io/topics/persistence).


#### 1) Downloading & Installing


Clone the repository and run `npm update` for all the dependencies to be installed:

```bash
git clone https://github.com/clintar/node-cryptonote-pool.git pool
cd pool
npm update
```

#### 2) Configuration


*Warning for Cryptonote coins other than Monero:* this software may or may not work with any given cryptonote coin.
Be wary of altcoins that change the number of minimum coin units because you will have to reconfigure several config
values to account for those changes. Unless you're offering a bounty reward - do not open an issue asking for help
getting a coin other than Monero working with this software.


Copy the `config_example.json` file to `config.json` then overview each options and change any to match your preferred setup.


Explanation for each field:
```javascript
/* Used for storage in redis so multiple coins can share the same redis instance. */
"coin": "monero",

/* BBR ONLY - this name is hardcoded in testing for whether to perform 
boolberry-specific functionality in pool code. (To be implemented. Only works for 
bbr atm)*/
/* "coin": "boolberry", */

/* Used for front-end display */
"symbol": "MRO",

/* BBR ONLY - */
/* "symbol": "BBR", */
"logging": {

    "files": {

        /* Specifies the level of log output verbosity. This level and anything
           more severe will be logged. Options are: info, warn, or error. */
        "level": "info",

        /* Directory where to write log files. */
        "directory": "logs",

        /* How often (in seconds) to append/flush data to the log files. */
        "flushInterval": 5
    },

    "console": {
        "level": "info",
        /* Gives console output useful colors. If you direct that output to a log file
           then disable this feature to avoid nasty characters in the file. */
        "colors": true
    }
},

/* Modular Pool Server */
"poolServer": {
    "enabled": true,

    /* Set to "auto" by default which will spawn one process/fork/worker for each CPU
       core in your system. Each of these workers will run a separate instance of your
       pool(s), and the kernel will load balance miners using these forks. Optionally,
       the 'forks' field can be a number for how many forks will be spawned. */
    "clusterForks": "auto",

    /* Address where block rewards go, and miner payments come from. */
    "poolAddress": "4AsBy39rpUMTmgTUARGq2bFQWhDhdQNekK5v4uaLU699NPAnx9CubEJ82AkvD5ScoAZNYRwBxybayainhyThHAZWCdKmPYn"

    /* Poll RPC daemons for new blocks every this many milliseconds. */
    "blockRefreshInterval": 1000,

	/* BBR ONLY - Location to save out scratchpad that can be used for -k option to minerd. Needs
       to be a location that the daemon has write access to and where it is served by
       http. */
        "scratchpadFilePath": "/var/www/htdocs/node-cryptonote-pool-bbr/scratchpad.bin",

	/* BBR ONLY - How often the scratchpad is saved in seconds. */
        "scratchpadFileUpdateInterval": 14400000,

    /* How many seconds until we consider a miner disconnected. */
    "minerTimeout": 900,

    "ports": [
        {
            "port": 3333, //Port for mining apps to connect to
	    /* BBR - Note: These difficulties should be much higher for BBR */
            "difficulty": 500000, //Initial difficulty miners are set to
            "desc": "Low end hardware" //Description of port
        },
        {
            "port": 5555,
	    /* BBR - Note: These difficulties should be much higher for BBR */
            "difficulty": 1000000,
            "desc": "Mid range hardware"
        },
        {
            "port": 7777,
	    /* BBR - Note: These difficulties should be much higher for BBR */
            "difficulty": 2000000,
            "desc": "High end hardware"
        }
    ],

    /* Variable difficulty is a feature that will automatically adjust difficulty for
       individual miners based on their hashrate in order to lower networking and CPU
       overhead. */
    "varDiff": {
	    /* BBR - Note: These difficulties should be much higher for BBR */
        "minDiff": 300000, //Minimum difficulty
        "maxDiff": 100000,
        "targetTime": 100, //Try to get 1 share per this many seconds
        "retargetTime": 30, //Check to see if we should retarget every this many seconds
        "variancePercent": 30, //Allow time to very this % from target without retargeting
	/* BBR - Note: This should probably be much higher for BBR */
        "maxJump": 500000 //Limit diff percent increase/decrease in a single retargetting
    },

    /* Feature to trust share difficulties from miners which can
       significantly reduce CPU load. */
    "shareTrust": {
        "enabled": true,
        "min": 10, //Minimum percent probability for share hashing
        "stepDown": 3, //Increase trust probability % this much with each valid share
        "threshold": 10, //Amount of valid shares required before trusting begins
        "penalty": 30 //Upon breaking trust require this many valid share before trusting
    },

        /* Only used with old http protocol and only works with the simpleminer. */
        "longPolling": {
            "enabled": false,
            "timeout": 8500
        },

    /* If under low-diff share attack we can ban their IP to reduce system/network load. */
    "banning": {
        "enabled": true,
        "time": 600, //How many seconds to ban worker for
        "invalidPercent": 25, //What percent of invalid shares triggers ban
        "checkThreshold": 30 //Perform check when this many shares have been submitted
    }
},

/* Module that sends payments to miners according to their submitted shares. */
"payments": {
    "enabled": true,
    "interval": 600, //how often to run in seconds
    "maxAddresses": 50, //split up payments if sending to more than this many addresses
    "mixin": 3, //number of transactions yours is indistinguishable from
    "transferFee": 5000000000, //fee to pay for each transaction
    "minPayment": 100000000000, //miner balance required before sending payment
	"maxTransactionAmount": 0, //split transactions by this amount(to prevent "too big transaction" error)
    "denomination": 100000000000 //truncate to this precision and store remainder
},

/* Module that monitors the submitted block maturities and manages rounds. Confirmed
   blocks mark the end of a round where workers' balances are increased in proportion
   to their shares. */
"blockUnlocker": {
    "enabled": true,
    "interval": 30, //how often to check block statuses in seconds

    /* Block depth required for a block to unlocked/mature. Found in daemon source as
       the variable CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW */
    "depth": 60,
    "poolFee": 1.8, //1.8% pool fee (2% total fee total including donations)
    "devDonation": 0.1, //0.1% donation to send to pool dev - only works with Monero
    "coreDevDonation": 0.1 //0.1% donation to send to core devs - only works with Monero
},

/* AJAX API used for front-end website. */
"api": {
    "enabled": true,
    "hashrateWindow": 600, //how many second worth of shares used to estimate hash rate
    "updateInterval": 3, //gather stats and broadcast every this many seconds
    "port": 8117,
    "blocks": 30, //amount of blocks to send at a time
    "payments": 30, //amount of payments to send at a time
    "password": "test" //password required for admin stats
},

/* Coin daemon connection details. */
"daemon": {
    "host": "127.0.0.1",
        "port": 10102
},

/* Wallet daemon connection details. */
"wallet": {
    "host": "127.0.0.1",
        "port": 10103
},

/* Redis connection into. */
"redis": {
    "host": "127.0.0.1",
    "port": 6379
}
```
#### 3) Configure your pool's scratchpad

For normal work minerd need to be pointed to inital scratchpad file. Configure your pool with target folder/scratchpad.bin file and time interval for updating scratchpad file, and pool via daemon rpc will be regularly update shared scratchpad file. 

```javascript
/* Path to local scratchpad file that will be shared with http server */
"scratchpadFilePath": "/usr/share/nginx/html/scratchpad.bin",

/* Local scratchpad file update interval, milliseconds (4 hours by default) */
"scratchpadFileUpdateInterval": 14400000, 
```
scratchpadFilePath should point to file that is shared by your http server, and you have to provide a link to this file in your mining instructions (mining command-line params)

Rezip and upload to your server or a file host. Then change the `easyminerDownload` link in your `config.json` file to
point to your zip file.

#### 4) Start the pool

```bash
node init.js
```

The file `config.json` is used by default but a file can be specified using the `-config=file` command argument, for example:

```bash
node init.js -config=config_backup.json
```

This software contains four distinct modules:
* `pool` - Which opens ports for miners to connect and processes shares
* `api` - Used by the website to display network, pool and miners' data
* `unlocker` - Processes block candidates and increases miners' balances when blocks are unlocked
* `payments` - Sends out payments to miners according to their balances stored in redis


By default, running the `init.js` script will start up all four modules. You can optionally have the script start
only start a specific module by using the `-module=name` command argument, for example:

```bash
node init.js -module=api
```

[Example screenshot](http://i.imgur.com/SEgrI3b.png) of running the pool in single module mode with tmux.


#### 5) Host the front-end

Simply host the contents of the `website_example` directory on file server capable of serving simple static files.


Edit the variables in the `website_example/config.js` file to use your pool's specific configuration.
Variable explanations:

```javascript

/* Must point to the API setup in your config.json file. */
var api = "http://poolhost:8117";

/* Minimum units in a single coin, for Bytecoin its 100000000. */
var coinUnits = 1000000000000;

/* Pool server host to instruct your miners to point to.  */
var poolHost = "cryppit.com";

/* IRC Server and room used for embedded KiwiIRC chat. */
var irc = "irc.freenode.net/#boolberry";

/* Contact email address. */
var email = "support@cryppit.com";

/* Market stat display params from https://www.cryptonator.com/widget */
var cryptonatorWidget = ["BBR-BTC", "BBR-USD", "BBR-EUR", "BBR-GBP"];

/* Download link to cryptonote-easy-miner for Windows users. */
var easyminerDownload = "https://mega.co.nz/#!oURlDLKB!g-REMRhaABVmCqrj2dqJcuaGblsNp-k2qtkdCDiK5So";

/* Used for front-end block links. For other coins it can be changed, for example with
   Bytecoin you can use "https://minergate.com/blockchain/bcn/block/". */
var blockchainExplorer = "https://minergate.com/blockchain/bbr/block/";

/* Used by front-end transaction links. Change for other coins. */
var transactionExplorer = "https://minergate.com/blockchain/bbr/transaction/";

```

#### 6) Customize your website

The following files are included so that you can customize your pool website without having to make significant changes
to `index.html` or other front-end files thus reducing the difficulty of merging updates with your own changes:
* `custom.css` for creating your own pool style
* `custom.js` for changing the functionality of your pool website
```


#### 7) [Optional] Configure cryptonote-easy-miner for your pool
Your miners that are Windows users can use [cryptonote-easy-miner](https://mega.co.nz/#!oURlDLKB!g-REMRhaABVmCqrj2dqJcuaGblsNp-k2qtkdCDiK5So)
which will automatically generate their wallet address and stratup multiple threads of simpleminer. You can download
it and edit the `config.ini` file to point to your own pool.
Inside the `easyminer` folder, edit `config.init` to point to your pool details
```ini
pool_host=example.com
pool_port=5555
```
Then simply serve the files via nginx, Apache, Google Drive, or anything that can host static content.

#### Upgrading
When updating to the latest code its important to not only `git pull` the latest from this repo, but to also update
the Node.js modules, and any config files that may have been changed.
* Inside your pool directory (where the init.js script is) do `git pull` to get the latest code.
* Remove the dependencies by deleting the `node_modules` directory with `rm -r node_modules`.
* Run `npm update` to force updating/reinstalling of the dependencies.
* Compare your `config.json` to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.

### Setting up Testnet

No cryptonote based coins have a testnet mode (yet) but you can effectively create a testnet with the following steps:

* Open `/src/p2p/net_node.inl` and remove lines with `ADD_HARDCODED_SEED_NODE` to prevent it from connecting to mainnet (Monero example: http://git.io/0a12_Q)
* Build the coin from source
* You now need to run two instance of the daemon and connect them to each other (without a connection to another instance the daemon will not accept RPC requests)
  * Run first instance with `./coind --p2p-bind-port 28080 --allow-local-ip`
  * Run second instance with `./coind --p2p-bind-port 5011 --rpc-bind-port 5010 --add-peer 0.0.0.0:28080 --allow-local-ip`
* You should now have a local testnet setup. The ports can be changes as long as the second instance is pointed to the first instance, obviously

*Credit to surfer43 for these instructions*


### JSON-RPC Commands from CLI

Documentation for JSON-RPC commands can be found here:
* Daemon https://wiki.bytecoin.org/wiki/Daemon_JSON_RPC_API
* Wallet https://wiki.bytecoin.org/wiki/Wallet_JSON_RPC_API


Curl can be used to use the JSON-RPC commands from command-line. Here is an example of calling `getblockheaderbyheight` for block 100:

```bash
curl 127.0.0.1:18081/json_rpc -d '{"method":"getblockheaderbyheight","params":{"height":100}}'
```


### Monitoring Your Pool

* To inspect and make changes to redis I suggest using [redis-commander](https://github.com/joeferner/redis-commander)
* To monitor server load for CPU, Network, IO, etc - I suggest using [New Relic](http://newrelic.com/)
* To keep your pool node script running in background, logging to file, and automatically restarting if it crashes - I suggest using [forever](https://github.com/nodejitsu/forever)


Donations
---------
* BTC: `1667jMt7NTZDaC8WXAxtMYBR8DPWCVoU4d`
* MRO: `48Y4SoUJM5L3YXBEfNQ8bFNsvTNsqcH5Rgq8RF7BwpgvTBj2xr7CmWVanaw7L4U9MnZ4AG7U6Pn1pBhfQhFyFZ1rL1efL8z`

For Boolberry devs

* BBR: `1KfzJfoA2pbB6J2ee2JG7wYSqwKtdoqs97pVMdB471FXArr1ce52Wm1BCWdAv9JAxZTa7wcUkq2s695Nmn59HgZ6VVnSjfp`
* XMR/MRO: `41id8jHp2UiVuSKJcq9D78CQp9Ku2ecqvWL76kUCMDxzA5Q4rAbTHJSijWC33aPjMD92Dbs8XBG3yU3neWGFfmB57WNkZxb`
* BTC: `1LBvA9X7KToPPKJFL8E5qdePhUoYCJiEfW`

Credits
===

* [zone117x](//github.com/zone117x) - Co-Dev on [node-cryptonote-pool](https://github.com/zone117x/node-cryptonote-pool), did a ton of work on the pool
* [LucasJones](//github.com/LucasJones) - Co-dev on node-boolberry-pool project; did tons of debugging for binary structures and fixing them. Pool couldn't have been made without him.
* [surfer43](//github.com/iamasupernova) - Did lots of testing during development to help figure out bugs and get them fixed
* [wallet42](http://moneropool.com) - Funded development of payment denominating and min threshold feature
* [Wolf0](https://bitcointalk.org/index.php?action=profile;u=80740) - Helped try to deobfuscate some of the daemon code for getting a bug fixed
* [Tacotime](https://bitcointalk.org/index.php?action=profile;u=19270) - helping with figuring out certain problems and lead the bounty for this project's creation
* [clintar] (https://github.com/clintar/node-cryptonote-pool/) - Converted zonex117x's pool software to work with BBR based on LucusJones' work

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html
