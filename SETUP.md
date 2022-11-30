https://notes.ethereum.org/@protolambda/merge-devnet-setup-guide

https://github.com/protolambda/merge-genesis-tools

- update genesis time to something like `date +%s`
  - decimal in `config.yaml`
  - hexadecimal in `genesis.json`
- initialize execution layer chain `geth init --datadir /tmp/holexec execution/genesis.json`
- get genesis hash from execution layer `eth.getBlockByNumber(0)`
  - update genesis hash in `config.yaml` and `deposit_contract_block.txt`
- generate a `mnemonic.yaml`

- launch execution-layer client `geth --networkid 11166111 --datadir /tmp/holexec`

https://github.com/protolambda/eth2-testnet-genesis

https://github.com/ethereum/staking-deposit-cli
./deposit existing-mnemonic


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
  --execution.urls "http://127.0.0.1:8551" \
  --jwt-secret "/tmp/holesovice/execution/0/geth/jwtsecret" \
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
  --execution.urls "http://127.0.0.1:8551" \
  --jwt-secret "/tmp/holesovice/execution/0/geth/jwtsecret" \
  --dataDir "/tmp/holesovice/consensus/4" \
  --paramsFile "/home/user/.opt/eth-clients/holesovice/consensus/config.yaml" \
  --genesisStateFile "/home/user/.opt/eth-clients/holesovice/consensus/genesis.ssz" \
  --enr.ip 127.0.0.1 \
  --rest.port 9996 \
  --port 9400 \
  --network.connectToDiscv5Bootnodes true \
  --bootnodes "enr:-Ku4QFQtoQ8Os6ePsGnaVhQHVOJpG1AdD8lI5eVin-LVXhDIY5GtGfZhJfWiN9sx59y9rt5kKEdA3hlVd5NcbVr97gkFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQL8-KCPND2kixBj-wDqGG33ySHYJflbz7XkGL5JsVxJnYN0Y3CCIyg,enr:-Ku4QNOQz2TatOuWhuFvuzfXpDsPrv4An-P5P60SwG3nR-pWdsj2G0Is7M5lJ0cqfGFrg0QLEb1Bj5l3u6gvo4PmFDEFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQPZxnhiMNp6Kwt0xg29s8bcCYsWheIyQ3GKamZ4lnipLIN0Y3CCI4w,enr:-Ku4QHFo3nbJPxWY0je6z34UPfma9xqP9os0ML3GIgUCnsyhRGC84B0BvHOOuxEVBEzKPAoCv7hbMkGGkbCPqMhMclUFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQP9MEV7lchABvza09gZKSuSb1pEv99SlFjrHSQujh7lb4N0Y3CCI_A,enr:-Ku4QHJ3K_HhJWuLid32jpxwZipds0-YgtX8IW43BvJqUQutJ6il5EIQY71qS9T_vHNst5Y_Bm0OOdh84xnxVF4hsLoFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQMmaG17UMfVGG8NAuAmMzFd88xWstoO_gHMOO5RnaWNi4N0Y3CCJFQ,enr:-Ku4QD_lKVwo52Y9JOrZibU9I6qh-QOiA213HAELfLHfnB4reWQhZq0YVS1rsf1GU7afqjD7UvKfWQTi6LsdjGNkMSwFh2F0dG5ldHOIAAAAAAAAAACEZXRoMpDfrT8vAhZhEf__________gmlkgnY0gmlwhH8AAAGJc2VjcDI1NmsxoQJsajVPzBxl8CS8W2Jj7CKpxojG5t85yrQ32WuMD1h4eoN0Y3CCJLg"

curl http://localhost:9996/eth/v1/node/identity
