<p align="center"><a href="https://github.com/eth-clients/goerli">Goerli (2019)</a> | <a href="https://github.com/eth-clients/sepolia">Sepolia (2021)</a> | <strong>Holešky (2023)</strong></p>
<p align="center"><img src="./assets/holesovice.png" /></p>

# Holešky (Holešovice Testnet)

Meet `--holesky`, the first long-standing, merged-from-genesis, public Ethereum testnet. Holešky will replace <a href="https://github.com/eth-clients/goerli">Goerli</a> as a staking, infrastructure and protocol-developer testnet in 2023. For testing decentralized applications, smart contracts, and other EVM functionality, please use <a href="https://github.com/eth-clients/sepolia">Sepolia</a>!

### Metadata

This is _work in progress_. The final config will be shared prior to launch date.

* Status: _Planned_ ([#3](https://github.com/eth-clients/holesky/issues/3))
* Flag: `--holesky`
* Flavor: Permissionless (Proof-of-Stake), _to replace Goerli_
* Launch Date: **Sept/28, 2023, 12:00 UTC**
  * Linux Epoch time: 1695902400
  * LTS: Dec/2027, EOL: Dec/2028
* Execution Version: Cancun _(TBD)_
* Consensus Version: Deneb _(TBD)_
* Merge: At Genesis (`0`)
* Shapella Epoch: `256`
  * Epoch time: `1696000704` (**Sept/29, 2023, 15:18:24 UTC**)
* Network ID: `17000`
* Chain ID: `17000`
* Genesis block on EL:
```json
 {
    "baseFeePerGas": "0x3b9aca00",
    "difficulty": "0x1",
    "extraData": "0x",
    "gasLimit": "0x17d7840",
    "gasUsed": "0x0",
    "hash": "0xb5f7f912443c940f21fd611f12828d75b534364ed9e95ca4e307729a4661bde4",
    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "miner": "0x0000000000000000000000000000000000000000",
    "mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "nonce": "0x0000000000001234",
    "number": "0x0",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "receiptsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
    "size": "0x202",
    "stateRoot": "0x69d8c9d72f6fa4ad42d4702b433707212f90db395eb54dc20bc85de253788783",
    "timestamp": "0x65156994",
    "totalDifficulty": "0x1",
    "transactions": [],
    "transactionsRoot": "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
    "uncles": []
  }
```
* EVM Version: London
  * Homestead: 0 (block number)
  * Byzantium: 0 (block number)
  * Constantinople: 0 (block number)
  * Petersfork: 0 (block number)
  * Istanbul: 0 (block number)
  * Berlin: 0 (block number)
  * MuirGlacier: 0 (block number)
  * London: 0 (block number)
  * Shanghai: 1696000704 (unix time)
  * Cancun: _(TBD)_
* Bootnodes Execution layer:
  * `enode://ac906289e4b7f12df423d654c5a962b6ebe5b3a74cc9e06292a85221f9a64a6f1cfdd6b714ed6dacef51578f92b34c60ee91e9ede9c7f8fadc4d347326d95e2b@146.190.13.128:30303`
  * `enode://a3435a0155a3e837c02f5e7f5662a2f1fbc25b48e4dc232016e1c51b544cb5b4510ef633ea3278c0e970fa8ad8141e2d4d0f9f95456c537ff05fdf9b31c15072@178.128.136.233:30303`
* Bootnodes Consensus layer:
  * // EF
  * `enr:-Ku4QFo-9q73SspYI8cac_4kTX7yF800VXqJW4Lj3HkIkb5CMqFLxciNHePmMt4XdJzHvhrCC5ADI4D_GkAsxGJRLnQBh2F0dG5ldHOIAAAAAAAAAACEZXRoMpAhnTT-AQFwAP__________gmlkgnY0gmlwhLKAiOmJc2VjcDI1NmsxoQORcM6e19T1T9gi7jxEZjk_sjVLGFscUNqAY9obgZaxbIN1ZHCCIyk`
  * `enr:-Ku4QPG7F72mbKx3gEQEx07wpYYusGDh-ni6SNkLvOS-hhN-BxIggN7tKlmalb0L5JPoAfqD-akTZ-gX06hFeBEz4WoBh2F0dG5ldHOIAAAAAAAAAACEZXRoMpAhnTT-AQFwAP__________gmlkgnY0gmlwhJK-DYCJc2VjcDI1NmsxoQKLVXFOhp2uX6jeT0DvvDpPcU8FWMjQdR4wMuORMhpX24N1ZHCCIyk`
  * `enr:-LK4QPxe-mDiSOtEB_Y82ozvxn9aQM07Ui8A-vQHNgYGMMthfsfOabaaTHhhJHFCBQQVRjBww_A5bM1rf8MlkJU_l68Eh2F0dG5ldHOIAADAAAAAAACEZXRoMpBpt9l0BAFwAAABAAAAAAAAgmlkgnY0gmlwhLKAiOmJc2VjcDI1NmsxoQJu6T9pclPObAzEVQ53DpVQqjadmVxdTLL-J3h9NFoCeIN0Y3CCIyiDdWRwgiMo`
  * `enr:-Ly4QGbOw4xNel5EhmDsJJ-QhC9XycWtsetnWoZ0uRy381GHdHsNHJiCwDTOkb3S1Ade0SFQkWJX_pgb3g8Jfh93rvMBh2F0dG5ldHOIAAAAAAAAAACEZXRoMpBpt9l0BAFwAAABAAAAAAAAgmlkgnY0gmlwhJK-DYCJc2VjcDI1NmsxoQOxKv9sv3zKF8GDewgFGGHKP5HCZZpPpTrwl9eXKAWGxIhzeW5jbmV0cwCDdGNwgiMog3VkcIIjKA`
  * // TEKU
  * `enr:-LS4QG0uV4qvcpJ-HFDJRGBmnlD3TJo7yc4jwK8iP7iKaTlfQ5kZvIDspLMJhk7j9KapuL9yyHaZmwTEZqr10k9XumyCEcmHYXR0bmV0c4gAAAAABgAAAIRldGgykGm32XQEAXAAAAEAAAAAAACCaWSCdjSCaXCErK4j-YlzZWNwMjU2azGhAgfWRBEJlb7gAhXIB5ePmjj2b8io0UpEenq1Kl9cxStJg3RjcIIjKIN1ZHCCIyg`
  * // Sigma Prime
  * `enr:-Le4QLoE1wFHSlGcm48a9ZESb_MRLqPPu6G0vHqu4MaUcQNDHS69tsy-zkN0K6pglyzX8m24mkb-LtBcbjAYdP1uxm4BhGV0aDKQabfZdAQBcAAAAQAAAAAAAIJpZIJ2NIJpcIQ5gR6Wg2lwNpAgAUHQBwEQAAAAAAAAADR-iXNlY3AyNTZrMaEDPMSNdcL92uNIyCsS177Z6KTXlbZakQqxv3aQcWawNXeDdWRwgiMohHVkcDaCI4I`
* Deposit contract address: `0x4242424242424242424242424242424242424242`

### Resources
* Block Explorer (EL+CL) [https://holesky.beaconcha.in/](https://holesky.beaconcha.in/)
* Ethstats: [https://ethstats.holesky.ethpandaops.io](https://ethstats.holesky.ethpandaops.io)
* Homepage: [https://holesky.ethpandaops.io](https://holesky.ethpandaops.io)
* Latest Proposal: [Ethereum Magicians - Predictable Ethereum Testnet Lifecycle](https://ethereum-magicians.org/t/proposal-predictable-ethereum-testnet-lifecycle/11575)
* Some Context: [Ethereum Magicians - Paths out of the Goerli supply mess](https://ethereum-magicians.org/t/testnet-workgroup-paths-out-of-the-goerli-supply-mess/11453)
