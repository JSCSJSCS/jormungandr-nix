# A nix-shell for Jormungandr

To drop into a shell with all configuraton files generated for you:
```
nix-shell https://github.com/input-output-hk/jormungandr-nix/archive/71dfe428f38f9942e1cb540d2e3ef1546d6ec54d.tar.gz -A shell
```

Once in the shell run `run-jormungandr` to start jormungandr.

To regenerate the config (in case of incompatible change in jormungandr after updating the commit), run `generate-config`

You can tweak the blockchain configuration through nix-shell parameters, eg.:
```
nix-shell --arg faucetAmounts "[ 100000 1234444 34556 ]" \
          --arg numberOfStakePools 2 \
          --argstr block0_consensus bft \
          --argstr storage "/tmp/jormungandr-storage" \
          https://github.com/input-output-hk/jormungandr-nix/archive/master.tar.gz \
          --run run-jormungandr
```

## Available parameters

| Parameter  | Default value | Description
| ------------- | ------------- | ------------- |
| `--argstr package` | `jormungandrMaster` | Jormungandr package to use. Use `jormungandr` for last stable release. `jormungandrMaster` for last master branch build. |
| `--argstr rootDir` | `/tmp` | Parent directory of the working directory (generated). |
| `--argstr block0_consensus` | `genesis`  | Consensus algorithm initialy used. `bft` or `genesis` |
| `--arg faucetAmounts`  | `[ 1000000000 ]` | List of amounts (space separated) in Lovelace that will be attributed to faucet addresses in block 0. |
| `--arg numberOfStakePools` | `0` if `bft`, `1` if `genesis` | Number of stake pools initialy registered. Each faucet will own on of the stake pool (hence `numberOfStakePools` must be ≤ `faucetAmounts` length). |
| `--arg numberOfLeaders` | `1` | Number of BFT leaders (keys will be generated). |
| `--arg block0_date` | `1550822014` (February 22, 2019) | the official start time of the blockchain, in seconds since UNIX EPOCH |
| `--arg isProduction` | `false` | if `true` (meant for production) use `production` for discrimination otherwise use `test`. |
| `--arg slots_per_epoch` | `60` | Number of slots in each epoch |
| `--arg slot_duration` | `10` | The slot duration, in seconds, is the time between the creation of 2 blocks |
| `--arg epoch_stability_depth` | `10` | The number of blocks (*10) per epoch |
| `--arg bft_slots_ratio` | `0` | Genesis praos parameter D |
| `--arg consensus_genesis_praos_active_slot_coeff` | `0.1` | Genesis praos active slot coefficient. Determines minimum stake required to try becoming slot leader, must be in range (0,1] |
| `--arg max_number_of_transactions_per_block` | `255` | This is the max number of messages allowed in a given Block |
| `--arg linear_fees_constant` | `10` | parameter in fee calculation [1] |
| `--arg linear_fees_coefficient` | `0` | parameter in fee calculation [1] |
| `--arg linear_fees_certificate` | `0` | parameter in fee calculation [1] |
| `--arg kes_update_speed` | `43200` (12 hours) | The speed to update the KES Key in seconds |
| `--argstr storage` | `./storage` | path to the storage. |
| `--argstr rest_listen` | `127.0.0.1:8443` | listen address of the rest endpoint |
| `--argstr rest_prefix` | `api` | rest api prefix |
| `--arg logger_verbosity` | `1` | logger verbosity. 0: warning, 1: info, 2: debug, 3 and above: trace. |
| `--argstr logger_format` | `plain` | log output format - `plain` or `json`. |
| `--argstr logger_output` | `gelf` | log output - `stderr`, `gelf` (graylog), `syslog` (unix only) or `journald` |
| `--argstr logger_backend` | `monitoring.stakepool.cardano-testnet.iohkdev.io:12201` if `gelf` | Graylog server to ouput the log to, default to iohk cardano-testnet graylog server (for debug purposes). |
| `--argstr logs_id` | generated uuid | Uniquely identify the logs of this node on the Graylog server. Please comunicate this id when filling issues. |
| `--argstr public_address` | `/ip4/127.0.0.1/tcp/8299` |  the address to listen from and accept connection from. This is the public address that will be distributed to other peers of the network that may find interest into participating to the blockchain dissemination with the node. |
| `--argstr  trusted_peers` | none | comma seperated list of of nodes to connect to in order to bootstrap the p2p topology (and bootstrap our local blockchain). Eg. `/ip4/104.24.28.11/tcp/8299,/ip4/104.24.29.11/tcp/8299` |
| `--argstr topics_of_interests` | `messages=high,blocks=high` if pools are registered, `messages=low,blocks=normal` otherwise | the different topics (comma separated) we are interested to hear about: - messages: notify other peers this node is interested about Transactions, typical setting for a non mining node: "low", for a stakepool: "high"; - blocks: notify other peers this node is interested about new Blocs, typical settings for a non mining node: "normal", for a stakepool: "high"; |

[1]: fee(num_bytes, has_certificate) = constant + num_bytes * coefficient + has_certificate * certificate