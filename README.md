<p align="center"><a href="https://github.com/eth-clients/goerli">Goerli (2019)</a> | <a href="https://github.com/eth-clients/sepolia">Sepolia (2021)</a> | <strong>Holešky (2023)</strong></p>
<p align="center"><img src="./assets/holesovice.png" /></p>

# Holešky (Holešovice Testnet)

Meet `--holesky`, the first long-standing, merged-from-genesis, public Ethereum testnet. Holešky will replace <a href="https://github.com/eth-clients/goerli">Goerli</a> as a staking, infrastructure and protocol-developer testnet in 2023. For testing decentralized applications, smart contracts, and other EVM functionality, please use <a href="https://github.com/eth-clients/sepolia">Sepolia</a>!

### Metadata

This is _work in progress_. The final config will be shared prior to launch date.

* Status: _Planned_ ([#3](https://github.com/eth-clients/holesky/issues/3))
* Flag: `--holesky`
* Flavor: Permissionless (Proof-of-Stake), _to replace Goerli_
* Launch Date: **Sept/15, 2023, 14:00 UTC**
  * Epoch time: 1694786400
  * LTS: Dec/2027, EOL: Dec/2028
* Execution Version: Cancun _(TBD)_
* Consensus Version: Deneb _(TBD)_
* Merge: At Genesis (`0`)
* Shapella Epoch: `10`
  * Epoch time: `1694790240` (64 minutes after genesis)
* Network ID: `17000`
* Chain ID: `17000`
* Genesis Hash: `0xff9006..64f23d`
* EVM Version: London
  * Homestead: 0 (block number)
  * Byzantium: 0 (block number)
  * Constantinople: 0 (block number)
  * Petersfork: 0 (block number)
  * Istanbul: 0 (block number)
  * Berlin: 0 (block number)
  * MuirGlacier: 0 (block number)
  * London: 0 (block number)
  * Shanghai: 1694790240 (unix time)
  * Cancun: _(TBD)_
* Bootnodes Execution layer:
  * `enode://ac906289e4b7f12df423d654c5a962b6ebe5b3a74cc9e06292a85221f9a64a6f1cfdd6b714ed6dacef51578f92b34c60ee91e9ede9c7f8fadc4d347326d95e2b@146.190.13.128:30303`
  * `enode://a3435a0155a3e837c02f5e7f5662a2f1fbc25b48e4dc232016e1c51b544cb5b4510ef633ea3278c0e970fa8ad8141e2d4d0f9f95456c537ff05fdf9b31c15072@178.128.136.233:30303`
* Bootnodes Consensus layer:
  * `enr:-KG4QMH842KsJOZAHxI98VJcf8oPr1U8Ylyp2Tb-sNAPniWSCaxIS4F9gc3lGOnROEok7g5qrOm8WgJTl2WXx8MhMmIMhGV0aDKQqX6DZjABcAAKAAAAAAAAAIJpZIJ2NIJpcISygIjpiXNlY3AyNTZrMaECvQMvoDF46BfJgvAbbv1hwpNu9VQBXRIpHS_B8zmkZmmDdGNwgiMog3VkcIIjKA`
  * `enr:-Ly4QDU8tZeygxz1gEeAD4EKe4H_8gg-IanpTY6h8A1YGPv5BPNvCMD77zjHUk_iF1pfG_8DC6jYWbIOD1k5kF-LaG4Bh2F0dG5ldHOIAAAAAAAAAACEZXRoMpCpfoNmMAFwAAoAAAAAAAAAgmlkgnY0gmlwhJK-DYCJc2VjcDI1NmsxoQN4bUae9DwIcq_56DNztksQYXeddTDKRonI5qI3YhN4SohzeW5jbmV0cwCDdGNwgiMog3VkcIIjKA`
* Deposit contract address: `0x4242424242424242424242424242424242424242`

### Resources
* Ethstats: [https://ethstats.holesky.ethpandaops.io](https://ethstats.holesky.ethpandaops.io)
* Homepage: [https://holesky.ethpandaops.io](https://holesky.ethpandaops.io)
* Latest Proposal: [Ethereum Magicians - Predictable Ethereum Testnet Lifecycle](https://ethereum-magicians.org/t/proposal-predictable-ethereum-testnet-lifecycle/11575)
* Some Context: [Ethereum Magicians - Paths out of the Goerli supply mess](https://ethereum-magicians.org/t/testnet-workgroup-paths-out-of-the-goerli-supply-mess/11453)
