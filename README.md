This repository contains a set of scripts to perform airdrop functions for the Voi blockchain network.

All scripts are executed using `npm run` as detailed below. The programs are stored in the `/scripts` folder.

# Quick Start Instructions to Execute an Airdrop
1. Clone this repo:
```
git clone https://github.com/xarmian/voi_drops.git
```

2. Change to voi_drops folder and install dependencies:
```
cd voi_drops
npm i
```

3. Download the applicable reward file from https://xarmian.github.io/voi_rewards/

OR download the file using CURL with the following command:
```
curl -G -o all_rewards.csv "https://socksfirstgames.com/proposers/csv.php" \
    -d start="2023-09-25" \
    -d end="2023-10-01" \
    -d block_reward=12500000 \
    -d health_reward=10000000
```

4. Add mnemonic to .env file. Copy the file `.env.sample` to `.env`, edit the file, and change the line: 
```# MNEMONIC=```
to 
```MNEMONIC=your mnemonic goes here```

5. Run a test airdrop
```
npm run airdrop -- -a all_rewards.csv -g 16 -n "This is an airdrop note" -t
```

6. Looks good? Run for real:
```
npm run airdrop -- -a all_rewards.csv -g 16 -n "This is an airdrop note"
```

# Airdrop script - airdrop.js
Performs an airdrop of network tokens to the accounts in `<acctlist>`,
which is a CSV file containing three columns -- account address, account type, and atomic amount

Optional Parameters:
`-t` -- Test mode - output details of transactions to be sent, but don't send them
`-b <blacklist>` -- a CSV containing a list of accounts to exclude from the airdrop
`-g <group_size>` -- the number of accounts to include in each atomic transaction group
(note -- if an issue arrises with an address, such as it being invalid, all transactions
in the group will fail and be included in the errorList.csv generated by the script)
`-m <mnemonic>` -- mnemonic string (space-separated words) entered inside quotes (or use environment variable MNEMONIC)
`-n <note>` -- note to be included in each transaction

Usage: `npm run airdrop -- -a <acctlist> [-t] [-b <blacklist>] [-g <group_size>] [-m "mnemonic"] [-n "note"]`

# Epoch reward calculaton script - epoch_calc.js

Given a `START` and `END`, and an integer of atomic tokens
(`EPOCHREWARD`) to distribute evenly among block proposers, this script calculates the number of tokens
to distribute to each block proposer and writes it to `FILENAME`. This output file can then be used
as the `<acctList>` input to `airdrop.js` to airdrop the alloted tokens to each address.

Note: Due to the intensity and time required to read blocks from the indexer API,
this script will create a SQLite database file named `proposers.db`.
The database is used to store block information and acts as a local cache.

`START` and `END` may be block numbers or dates. If a date is provided, the script will automatically
locate the first block available on the `START` date and the last block available on the `END` date

Usage: `npm run epoch_calc -- -s START -e END -r EPOCHREWARD [-f FILENAME] [-b BLACKLIST]`  
Example: `npm run epoch_calc -- -s 240000 -e 250000 -r 2500000 -f rewards.csv -b blacklist.csv`
Example: `npm run epoch_calc -- -s 2023-09-25 -e 2023-10-01 -r 2500000 -f rewards.csv -b blacklist.csv`

# Epoch reward calculation from API - epoch_calc_api.js

Similar to `epoch_calc` but instead of using a local SQLite database, it will
use an API to retrieve block proposers.

Usage: `npm run epoch_calc_api -- -s START -e END -r EPOCHREWARD [-f FILENAME] [-b BLACKLIST]`  
Example: `npm run epoch_calc_api -- -s 240000 -e 250000 -r 2500000 -f rewards.csv -b blacklist.csv`
Example: `npm run epoch_calc_api -- -s 2023-09-25 -e 2023-10-01 -r 2500000 -f rewards.csv -b blacklist.csv`

# Account bucketing script - buckets.js

Usage: `npm run buckets`

NOTE: buckets.js is incomplete. 

# Block finder by timestamp - find_block.js

Given a timestamp in the format YYYY-MM-DDTHH:II:SS, find the block produced at the specified time,
or the next block produced after the specified time. For example, if block 123 is produced at 2023-08-30T07:59:59
and block 124 is produced at 2023-08-30T08:00:01, and the `TIMESTAMP` input is 2023-08-30T08:00:00 then the script
will return 124.

Note that a Z must be included in the time to use UTC time, otherwise the search will be performed based on the
local machine timezone.

This script will also accept a date in the format `YYYY-MM-DD` and will perform a search for the first and last
block produced between 12:00:00AM and 11:59:59PM GMT, and will output the command to run epoch_calc.js

Usage: `npm run find_block -- -t TIMESTAMP`  
Example: `npm run find_block -- -t 2023-09-19`

# Block scraper - block_follower.js

This utility script builds out a local SQLite database named `proposers.db` by pulling block data from API endpoints.
It stores the block number, proposer, and block timestamp in the database to be used by `epoch_calc.js` and the
proposers API.

Usage: `npm run block_follower -- [-f FILENAME]`

# Account Balance Snapshot Tool - balances.js

This script calculates the account balances of all accounts with a VOI or VIA balance.
By default the snapshot will be taken of the current block at the time the script is executed.
An optional -r parameter may be provided to specify the snapshot block.

Usage: `npm run balances -- -a [account] [-r round] [-b blacklist.csv]`

All parameters are optional:
- If no account is provided, script will iterate and snapshot balances for all accounts
- If no round is provided, script will snapshot the current round
- If no blacklist is provided, script will not filter out any accounts

If the -a parameter is used, the output will only be to the screen. If the -a parameter is not provided,
the current balances will be written to the file `balances.csv`.

# Environment Variables

Environment variables can be stored in a `.env` file in the root folder. A sample env file is located in the root
folder named `env.sample`. Uncomment a line and add a value to override the value used by this package.

Algod and Indexer clients will default to nodely's voi-testnet nodes, but can be configured with the environment variables:
- ALGOD_HOST
- ALGOD_PORT
- ALGOD_TOKEN
- INDEXER_HOST
- INDEXER_PORT
- INDEXER_TOKEN

Menonic to be used by Airdrop script:
- MNEMONIC
