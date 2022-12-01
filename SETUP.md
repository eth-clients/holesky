# How to setup a post-merge Testnet

Field notes from setting up a public Ethereum testnet that is _"merged from genesis."_ It has been compiled from many bits and pieces on the web with main pioneering work done by Protolambda and Parithosh.

### Execution-Layer Configuration

Creating a new genesis is as simple as running `geth dumpgenesis` and subsequently modifying fields as needed for the testnet. Main cornerstones:

- set all protocol upgrades to block `0`
- set merge block and TTD to `0` and set the TTD passed flag to `true`
- remove the `ethash` parameter altogether
- remove the existing `alloc` as well and create a custom allocation as per our needs
- adjust the genesis parameters as we wish (mainly `timestamp`, `extraData`, `nonce`)
  - current unix time: `date +%s`
  - note that all fields needs to be hexadecimal
- lastly, set a new `chainID` for replay protection

Now all that is left is the deposit contract. Since we _merge from genesis_, this has to be part of the genesis state.

- take the vanilla deposit contract from the consensus specs: [deposit_contract.sol](https://github.com/ethereum/consensus-specs/blob/dev/solidity_deposit_contract/deposit_contract.sol)
- get the solidity compiler version 0.6.11 and get the binary of the runtime part
- create an empty deposit tree (see [proto's notes](https://github.com/protolambda/merge-genesis-tools/blob/master/README.md#deposit-contract) below)

> - To embed the run-time code of the contract, not the deployment code
>     - Use: `solc --bin-runtime`
> - Initialize the initial storage, normally initialized by the contract constructor
>     - Reverse engineered from contract storage debugger (remix IDE)
>     - Verified with `compute_initial_storage`
>     - Matching the storage layout: `solc --storage-layout`
> - We embed an empty deposit tree, there are no missing deposit events.
>
> _[...]_
>
> To generate the storage layout data:
> ```
> solc --optimize --optimize-runs 5000000 --metadata-literal --bin-runtime --storage-layout deposit_contract.sol
> ```
>
> To compute the storage key-value pairs:
> ```
> (venv) python compute_initial_storage.py
> ```

Finally, add the binary and storage tuples to the alloc of the genesis file. Once set, initialize as many execution clients as desired.

```shell
geth init \
  --datadir "/tmp/holesovice/execution/0" \
  /home/user/.opt/eth-clients/holesovice/execution/genesis.json

geth init \
  --datadir "/tmp/holesovice/execution/1" \
  /home/user/.opt/eth-clients/holesovice/execution/genesis.json
```

Run an execution node and get the genesis hash from it, e.g., attach to it and query `eth.getBlockByNumber(0)`.

```shell
geth \
  --networkid 11166111 \
  --datadir "/tmp/holesovice/execution/0"
```

Note down the genesis hash for the consensus layer later and also get the node's enode to connect other execution-layer clients if desired.

```shell
geth \
  --networkid 11166111 \
  --datadir "/tmp/holesovice/execution/1" \
  --authrpc.port 8651 \
  --port 31303 \
  --bootnodes "enode://d5704c15a444908cacdea75c25139530024c248b05815e7e260c415bea2381465fa7cd4f78d778700e37a2be9fe480642d308e74a704bcf146bf7cb53732ab83@127.0.0.1:30303"
```

Persist the bootnodes in a text file for later use if necessary, e.g., `bootnodes.txt`.

### Consensus-Layer Configuration

To create a new consensus-layer configuration, simply get the latest [mainnet.yaml](https://github.com/ethereum/consensus-specs/blob/dev/configs/mainnet.yaml) config from the consensus specs.

- come up with a unique `CONFIG_NAME`, i.e., the name of our testnet
- set TTD to `0` and the terminal hash to the genesis hash we previously extracted
- set the minimum genesis time to the genesis time of the execution layer (or later)
- set the fork versions to something custom to not conflict with mainnet, e.g., `0x00166111` for genesis, `0x01166111` for altair, and so on
- set the deposit chain and network ID to the previously created execution-layer network and match the deposit contract address to the one in the genesis

In addition to the `config.yaml` for the consensus layer, we also create auxiliary files for the deposit contract:

- `deploy_block.txt` containing the block number `0`
- `deposit_contract_block.txt` containing the execution genesis hash
- `deposit_contract.txt` containing the address from the genesis alloc

Lastly, before generating the genesis, we need to create validators that will kick off the chain right from the start.

- generate mnemonics (BIP0039) for everyone who is going to run genesis validators
  - store them in a `mnemonics.yaml`
  - see these notes for formatting: [protolambda/eth2-testnet-genesis#mnemonics](https://github.com/protolambda/eth2-testnet-genesis#mnemonics)
- lastly, use the [eth2 testnet genesis](https://github.com/protolambda/eth2-testnet-genesis) tool to generate the `genesis.ssz` state for a `merge` configuration

```shell
eth2-testnet-genesis merge \
  --config "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --eth1-config "/home/user/.opt/eth-clients/holesovice/execution/genesis.json" \
  --mnemonics "/home/user/.opt/eth-clients/holesovice/consensus/mnemonic.yaml" \
  --state-output "/home/user/.opt/eth-clients/holesovice/consensus/genesis.ssz" \
  --tranches-dir "/home/user/.opt/eth-clients/holesovice/consensus/tranches"
```

The genesis state will contain the initial genesis validators and there is no need to conduct actual deposits. To add more validators later, simply use the deposit contract at any time.

That's all for the configuration. We can run beacon nodes now and allow them to network before genesis. Note, that the network needs at least two peered beacon nodes, otherwise the nodes will act as if they are offline and refuse to author blocks.

```shell
lodestar beacon \
  --suggestedFeeRecipient "0xCaA29806044A08E533963b2e573C1230A2cd9a2d" \
  --execution.urls "http://127.0.0.1:8551" \
  --jwt-secret "/tmp/holesovice/execution/0/geth/jwtsecret" \
  --dataDir "/tmp/holesovice/consensus/0" \
  --paramsFile "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --genesisStateFile "/home/user/.opt/eth-clients/holesovice/consensus/genesis.ssz" \
  --enr.ip 127.0.0.1
```

Note, we are setting the ENR IP to whatever we need it _(here: local)_ so that we can use the ENR as initial bootnode record. Get it from the identity endpoint:

```shell
curl http://localhost:9596/eth/v1/node/identity
```

All ENRs can be persisted in a file such as `bootnodes.txt` that we can pass to the beacon-chain clients later to bootstrap the networking.

```shell
lodestar beacon \
  --suggestedFeeRecipient "0xCaA29806044A08E533963b2e573C1230A2cd9a2d" \
  --execution.urls "http://127.0.0.1:8551" \
  --jwt-secret "/tmp/holesovice/execution/0/geth/jwtsecret" \
  --dataDir "/tmp/holesovice/consensus/1" \
  --paramsFile "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --genesisStateFile "/home/user/.opt/eth-clients/holesovice/consensus/genesis.ssz" \
  --enr.ip 127.0.0.1 \
  --rest.port 9696 \
  --port 9100 \
  --network.connectToDiscv5Bootnodes true \
  --bootnodes "enr:-Ku4QFQtoQ8Os6ePsGnaVhQHVOJpG1AdD8lI5eVin-LVXhDIY5GtGfZhJfWiN9sx59y9rt5kKEdA3hlVd5NcbVr97gkFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQL8-KCPND2kixBj-wDqGG33ySHYJflbz7XkGL5JsVxJnYN0Y3CCIyg,enr:-Ku4QNOQz2TatOuWhuFvuzfXpDsPrv4An-P5P60SwG3nR-pWdsj2G0Is7M5lJ0cqfGFrg0QLEb1Bj5l3u6gvo4PmFDEFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQPZxnhiMNp6Kwt0xg29s8bcCYsWheIyQ3GKamZ4lnipLIN0Y3CCI4w,enr:-Ku4QHFo3nbJPxWY0je6z34UPfma9xqP9os0ML3GIgUCnsyhRGC84B0BvHOOuxEVBEzKPAoCv7hbMkGGkbCPqMhMclUFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQP9MEV7lchABvza09gZKSuSb1pEv99SlFjrHSQujh7lb4N0Y3CCI_A,enr:-Ku4QHJ3K_HhJWuLid32jpxwZipds0-YgtX8IW43BvJqUQutJ6il5EIQY71qS9T_vHNst5Y_Bm0OOdh84xnxVF4hsLoFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQMmaG17UMfVGG8NAuAmMzFd88xWstoO_gHMOO5RnaWNi4N0Y3CCJFQ,enr:-Ku4QD_lKVwo52Y9JOrZibU9I6qh-QOiA213HAELfLHfnB4reWQhZq0YVS1rsf1GU7afqjD7UvKfWQTi6LsdjGNkMSwFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQJsajVPzBxl8CS8W2Jj7CKpxojG5t85yrQ32WuMD1h4eoN0Y3CCJLg"
```

We do this for as many beacon nodes as we want. Note that we need to enable discv5 bootnodes, otherwise our clients will not peer.

### Launch the Network

We have both execution-layer and consensus-layer clients running already. We can easily make sure they are running correctly by reading the logs. If they fail to authenticate or connect, there will be errors.

However, to actually kick off the network, we need to ensure the validators are loaded and connected prior to genesis. If we fail to generate a chain within the first 32 slots the clients will remain forever in a _syncing_ state, potentially to prevent long-range attacks.

- use the [deposit CLI](https://github.com/ethereum/staking-deposit-cli) to generate validator keypairs
- run `./deposit existing-mnemonic` and follow the instructions

This generates deposit data and the actual keypairs. The deposit data can be discarded as we don't need actual deposits. The keys can be imported to our validator client. Make sure to also provide a password.

```shell
lodestar validator \
  --dataDir "/tmp/holesovice/consensus/0" \
  --suggestedFeeRecipient "0xCaA29806044A08E533963b2e573C1230A2cd9a2d" \
  --graffiti "YOLO HOLESOVICE LODESTAR" \
  --paramsFile "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --importKeystores "/home/user/.opt/eth-clients/holesovice/consensus/validator_keys" \
  --importKeystoresPassword "/home/user/.opt/eth-clients/holesovice/consensus/validator_keys/password.txt"
```

Once the keys and passwords are imported, the validator client will wait to take on its duties. Once the minimum genesis time (and genesis delay) are passed, it will propose blocks and attestations.

That's our new testnet.

# Resources

Credit goes to Gajinder, Parithosh, and Protolamda for the valuable pointers, tools, and documentation.

- [github/protolambda/merge-genesis-tools](https://github.com/protolambda/merge-genesis-tools)
- [github/ethereum/consensus-specs](https://github.com/ethereum/consensus-specs)
- [notes/protolambda/merge-devnet-setup-guide](https://notes.ethereum.org/@protolambda/merge-devnet-setup-guide)
- [github/ethereumjs/consensus-deployment-ansible](https://github.com/ethereumjs/consensus-deployment-ansible)
- [github/protolambda/eth2-testnet-genesis](https://github.com/protolambda/eth2-testnet-genesis)
- [github/ethereum/staking-deposit-cli](https://github.com/ethereum/staking-deposit-cli)
- [github/ChainSafe/lodestar](https://github.com/ChainSafe/lodestar)

# Temporary Local Testnet Notes

```shell
geth init \
  --datadir "/tmp/holesovice/execution/0" \
  /home/user/.opt/eth-clients/holesovice/execution/genesis.json

geth init \
  --datadir "/tmp/holesovice/execution/1" \
  /home/user/.opt/eth-clients/holesovice/execution/genesis.json

geth \
  --networkid 11166111 \
  --datadir "/tmp/holesovice/execution/0" \
  --bootnodes "enode://d5704c15a444908cacdea75c25139530024c248b05815e7e260c415bea2381465fa7cd4f78d778700e37a2be9fe480642d308e74a704bcf146bf7cb53732ab83@127.0.0.1:30303,enode://eb271cb12a0f227265e79322fb3e8fc9e87e5401d1781f328b2002584dcfc200686f76644c614f629fcea10ca81b06c5ceb357da5aadd62934e8c26fc39944b3@127.0.0.1:31303"

geth attach "/tmp/holesovice/execution/0/geth.ipc"

eth.getBlockByNumber(0)

geth \
  --networkid 11166111 \
  --datadir "/tmp/holesovice/execution/1" \
  --authrpc.port 8651 \
  --port 31303 \
  --bootnodes "enode://d5704c15a444908cacdea75c25139530024c248b05815e7e260c415bea2381465fa7cd4f78d778700e37a2be9fe480642d308e74a704bcf146bf7cb53732ab83@127.0.0.1:30303,enode://eb271cb12a0f227265e79322fb3e8fc9e87e5401d1781f328b2002584dcfc200686f76644c614f629fcea10ca81b06c5ceb357da5aadd62934e8c26fc39944b3@127.0.0.1:31303"

eth2-testnet-genesis merge \
  --config "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --eth1-config "/home/user/.opt/eth-clients/holesovice/execution/genesis.json" \
  --mnemonics "/home/user/.opt/eth-clients/holesovice/consensus/mnemonic.yaml" \
  --state-output "/home/user/.opt/eth-clients/holesovice/consensus/genesis.ssz" \
  --tranches-dir "/home/user/.opt/eth-clients/holesovice/consensus/tranches"

lodestar beacon \
  --suggestedFeeRecipient "0xCaA29806044A08E533963b2e573C1230A2cd9a2d" \
  --execution.urls "http://127.0.0.1:8551" \
  --jwt-secret "/tmp/holesovice/execution/0/geth/jwtsecret" \
  --dataDir "/tmp/holesovice/consensus/0" \
  --paramsFile "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --genesisStateFile "/home/user/.opt/eth-clients/holesovice/consensus/genesis.ssz" \
  --enr.ip 127.0.0.1

lodestar validator \
  --dataDir "/tmp/holesovice/consensus/0" \
  --suggestedFeeRecipient "0xCaA29806044A08E533963b2e573C1230A2cd9a2d" \
  --graffiti "YOLO HOLESOVICE LODESTAR" \
  --paramsFile "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --importKeystores "/home/user/.opt/eth-clients/holesovice/consensus/validator_keys" \
  --importKeystoresPassword "/home/user/.opt/eth-clients/holesovice/consensus/validator_keys/password.txt"

curl http://localhost:9596/eth/v1/node/identity

lodestar beacon \
  --suggestedFeeRecipient "0xCaA29806044A08E533963b2e573C1230A2cd9a2d" \
  --execution.urls "http://127.0.0.1:8551" \
  --jwt-secret "/tmp/holesovice/execution/0/geth/jwtsecret" \
  --dataDir "/tmp/holesovice/consensus/1" \
  --paramsFile "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --genesisStateFile "/home/user/.opt/eth-clients/holesovice/consensus/genesis.ssz" \
  --enr.ip 127.0.0.1 \
  --rest.port 9696 \
  --port 9100 \
  --network.connectToDiscv5Bootnodes true \
  --bootnodes "enr:-Ku4QFQtoQ8Os6ePsGnaVhQHVOJpG1AdD8lI5eVin-LVXhDIY5GtGfZhJfWiN9sx59y9rt5kKEdA3hlVd5NcbVr97gkFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQL8-KCPND2kixBj-wDqGG33ySHYJflbz7XkGL5JsVxJnYN0Y3CCIyg,enr:-Ku4QNOQz2TatOuWhuFvuzfXpDsPrv4An-P5P60SwG3nR-pWdsj2G0Is7M5lJ0cqfGFrg0QLEb1Bj5l3u6gvo4PmFDEFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQPZxnhiMNp6Kwt0xg29s8bcCYsWheIyQ3GKamZ4lnipLIN0Y3CCI4w,enr:-Ku4QHFo3nbJPxWY0je6z34UPfma9xqP9os0ML3GIgUCnsyhRGC84B0BvHOOuxEVBEzKPAoCv7hbMkGGkbCPqMhMclUFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQP9MEV7lchABvza09gZKSuSb1pEv99SlFjrHSQujh7lb4N0Y3CCI_A,enr:-Ku4QHJ3K_HhJWuLid32jpxwZipds0-YgtX8IW43BvJqUQutJ6il5EIQY71qS9T_vHNst5Y_Bm0OOdh84xnxVF4hsLoFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQMmaG17UMfVGG8NAuAmMzFd88xWstoO_gHMOO5RnaWNi4N0Y3CCJFQ,enr:-Ku4QD_lKVwo52Y9JOrZibU9I6qh-QOiA213HAELfLHfnB4reWQhZq0YVS1rsf1GU7afqjD7UvKfWQTi6LsdjGNkMSwFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQJsajVPzBxl8CS8W2Jj7CKpxojG5t85yrQ32WuMD1h4eoN0Y3CCJLg"

curl http://localhost:9696/eth/v1/node/identity

lodestar beacon \
  --suggestedFeeRecipient "0xCaA29806044A08E533963b2e573C1230A2cd9a2d" \
  --execution.urls "http://127.0.0.1:8651" \
  --jwt-secret "/tmp/holesovice/execution/1/geth/jwtsecret" \
  --dataDir "/tmp/holesovice/consensus/2" \
  --paramsFile "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --genesisStateFile "/home/user/.opt/eth-clients/holesovice/consensus/genesis.ssz" \
  --enr.ip 127.0.0.1 \
  --rest.port 9796 \
  --port 9200 \
  --network.connectToDiscv5Bootnodes true \
  --bootnodes "enr:-Ku4QFQtoQ8Os6ePsGnaVhQHVOJpG1AdD8lI5eVin-LVXhDIY5GtGfZhJfWiN9sx59y9rt5kKEdA3hlVd5NcbVr97gkFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQL8-KCPND2kixBj-wDqGG33ySHYJflbz7XkGL5JsVxJnYN0Y3CCIyg,enr:-Ku4QNOQz2TatOuWhuFvuzfXpDsPrv4An-P5P60SwG3nR-pWdsj2G0Is7M5lJ0cqfGFrg0QLEb1Bj5l3u6gvo4PmFDEFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQPZxnhiMNp6Kwt0xg29s8bcCYsWheIyQ3GKamZ4lnipLIN0Y3CCI4w,enr:-Ku4QHFo3nbJPxWY0je6z34UPfma9xqP9os0ML3GIgUCnsyhRGC84B0BvHOOuxEVBEzKPAoCv7hbMkGGkbCPqMhMclUFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQP9MEV7lchABvza09gZKSuSb1pEv99SlFjrHSQujh7lb4N0Y3CCI_A,enr:-Ku4QHJ3K_HhJWuLid32jpxwZipds0-YgtX8IW43BvJqUQutJ6il5EIQY71qS9T_vHNst5Y_Bm0OOdh84xnxVF4hsLoFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQMmaG17UMfVGG8NAuAmMzFd88xWstoO_gHMOO5RnaWNi4N0Y3CCJFQ,enr:-Ku4QD_lKVwo52Y9JOrZibU9I6qh-QOiA213HAELfLHfnB4reWQhZq0YVS1rsf1GU7afqjD7UvKfWQTi6LsdjGNkMSwFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQJsajVPzBxl8CS8W2Jj7CKpxojG5t85yrQ32WuMD1h4eoN0Y3CCJLg"

curl http://localhost:9796/eth/v1/node/identity

lodestar beacon \
  --suggestedFeeRecipient "0xCaA29806044A08E533963b2e573C1230A2cd9a2d" \
  --execution.urls "http://127.0.0.1:8551" \
  --jwt-secret "/tmp/holesovice/execution/0/geth/jwtsecret" \
  --dataDir "/tmp/holesovice/consensus/3" \
  --paramsFile "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --genesisStateFile "/home/user/.opt/eth-clients/holesovice/consensus/genesis.ssz" \
  --enr.ip 127.0.0.1 \
  --rest.port 9896 \
  --port 9300 \
  --network.connectToDiscv5Bootnodes true \
  --bootnodes "enr:-Ku4QFQtoQ8Os6ePsGnaVhQHVOJpG1AdD8lI5eVin-LVXhDIY5GtGfZhJfWiN9sx59y9rt5kKEdA3hlVd5NcbVr97gkFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQL8-KCPND2kixBj-wDqGG33ySHYJflbz7XkGL5JsVxJnYN0Y3CCIyg,enr:-Ku4QNOQz2TatOuWhuFvuzfXpDsPrv4An-P5P60SwG3nR-pWdsj2G0Is7M5lJ0cqfGFrg0QLEb1Bj5l3u6gvo4PmFDEFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQPZxnhiMNp6Kwt0xg29s8bcCYsWheIyQ3GKamZ4lnipLIN0Y3CCI4w,enr:-Ku4QHFo3nbJPxWY0je6z34UPfma9xqP9os0ML3GIgUCnsyhRGC84B0BvHOOuxEVBEzKPAoCv7hbMkGGkbCPqMhMclUFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQP9MEV7lchABvza09gZKSuSb1pEv99SlFjrHSQujh7lb4N0Y3CCI_A,enr:-Ku4QHJ3K_HhJWuLid32jpxwZipds0-YgtX8IW43BvJqUQutJ6il5EIQY71qS9T_vHNst5Y_Bm0OOdh84xnxVF4hsLoFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQMmaG17UMfVGG8NAuAmMzFd88xWstoO_gHMOO5RnaWNi4N0Y3CCJFQ,enr:-Ku4QD_lKVwo52Y9JOrZibU9I6qh-QOiA213HAELfLHfnB4reWQhZq0YVS1rsf1GU7afqjD7UvKfWQTi6LsdjGNkMSwFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQJsajVPzBxl8CS8W2Jj7CKpxojG5t85yrQ32WuMD1h4eoN0Y3CCJLg"

curl http://localhost:9896/eth/v1/node/identity

lodestar beacon \
  --suggestedFeeRecipient "0xCaA29806044A08E533963b2e573C1230A2cd9a2d" \
  --execution.urls "http://127.0.0.1:8651" \
  --jwt-secret "/tmp/holesovice/execution/1/geth/jwtsecret" \
  --dataDir "/tmp/holesovice/consensus/4" \
  --paramsFile "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --genesisStateFile "/home/user/.opt/eth-clients/holesovice/consensus/genesis.ssz" \
  --enr.ip 127.0.0.1 \
  --rest.port 9996 \
  --port 9400 \
  --network.connectToDiscv5Bootnodes true \
  --bootnodes "enr:-Ku4QFQtoQ8Os6ePsGnaVhQHVOJpG1AdD8lI5eVin-LVXhDIY5GtGfZhJfWiN9sx59y9rt5kKEdA3hlVd5NcbVr97gkFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQL8-KCPND2kixBj-wDqGG33ySHYJflbz7XkGL5JsVxJnYN0Y3CCIyg,enr:-Ku4QNOQz2TatOuWhuFvuzfXpDsPrv4An-P5P60SwG3nR-pWdsj2G0Is7M5lJ0cqfGFrg0QLEb1Bj5l3u6gvo4PmFDEFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQPZxnhiMNp6Kwt0xg29s8bcCYsWheIyQ3GKamZ4lnipLIN0Y3CCI4w,enr:-Ku4QHFo3nbJPxWY0je6z34UPfma9xqP9os0ML3GIgUCnsyhRGC84B0BvHOOuxEVBEzKPAoCv7hbMkGGkbCPqMhMclUFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQP9MEV7lchABvza09gZKSuSb1pEv99SlFjrHSQujh7lb4N0Y3CCI_A,enr:-Ku4QHJ3K_HhJWuLid32jpxwZipds0-YgtX8IW43BvJqUQutJ6il5EIQY71qS9T_vHNst5Y_Bm0OOdh84xnxVF4hsLoFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQMmaG17UMfVGG8NAuAmMzFd88xWstoO_gHMOO5RnaWNi4N0Y3CCJFQ,enr:-Ku4QD_lKVwo52Y9JOrZibU9I6qh-QOiA213HAELfLHfnB4reWQhZq0YVS1rsf1GU7afqjD7UvKfWQTi6LsdjGNkMSwFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQJsajVPzBxl8CS8W2Jj7CKpxojG5t85yrQ32WuMD1h4eoN0Y3CCJLg"

curl http://localhost:9996/eth/v1/node/identity
```
