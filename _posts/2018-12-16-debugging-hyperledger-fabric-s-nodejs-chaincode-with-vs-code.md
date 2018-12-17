---
layout: post
published: true
title: Debugging Hyperledger Fabric's NodeJS chaincode with VS Code
date: '2018-12-16'
background: /img/posts/fabriclovesvscode.png
---
## Development network

To test our chaincode we don't need a complex fabric network, only an endorsing peer, an orderer and optionally a client to query the endorser.

```
                   +--------+       +-----------+
                   |        |       |           |
                   |  Peer  |       |  Orderer  |
+----------+       |        |       |           |
|  Client  +------->        +-------+           |
+----------+       +----+---+       +-----------+
                        |
                        |
                   +----v------+
                   |  VS Code  |
                   |  Debugger |
                   |           |
                   +-----------+

```

This post is based on the [fabric-samples](https://github.com/hyperledger/fabric-samples/tree/release-1.3/chaincode-docker-devmode) repository, where you can **download** some of the files for this demo. You will **need** the `msp/` directory, and the `myc.tx`, `orderer.block` and `script.sh` files, aside from the files in this post.

The directory sctructure you should have by the end is:

```
devmode
└───msp
    ├───admincerts
    ├───cacerts
    ├───keystore
    ├───signcerts
    ├───tlscacerts
    └───tlsintermediatecerts
    devmode.sh
    docker-compose.yaml
    installandinstantiate.sh
    myc.tx
    orderer.block
    script.sh
```

The _fabric-samples_ devmode demo sets up a network with the default LevelDB database, but I find more useful in my chaincode development the CouchDB option (it also depends in your use case).

If you want to test your chaincode against the CouchDB database, the following docker-compose file sets up this simple network, but with a CouchDB database for the ledger:

<script src="https://gist.github.com/jlcs-es/163bf8cb09922a43befd90d6e5a86074.js"></script>


## Test chaincode

Instead of using one of the many demos available in the fabric-samples project, I will use a basic template from where you can start you own development.

This is a simple [Typescript](https://www.typescriptlang.org/) chaincode, which compiles to pure JavaScript, but adds nice tools within VS Code, like autocompletion. Really recommended when learning to use the Fabric SDK.

The `chaincode.ts` file is the main one. It imports the chaincode implementation and starts a peer chaincode process with it. The `shim.start()` function will wait until a peer node connects to it, in our case, the `peer-dev` node from the docker-compose network above.

The `mycc.ts` file naively implements the `ChaincodeInterface` with Typescript types and console output.

<script src="https://gist.github.com/jlcs-es/65eb47f459747828c6fe72cab9356c80.js"></script>

## VS Code debugger

Next, we need to write a [launch configuration for VS Code](https://code.visualstudio.com/Docs/editor/debugging), which tells it how to run our chaincode.

## Launch it all

## Automated scripts

<script src="https://gist.github.com/jlcs-es/56e55302897334bcc104e410fe631cde.js"></script>

<script src="https://gist.github.com/jlcs-es/8006f329a17ff9ad2458a43f445b8dc1.js"></script>

<script src="https://gist.github.com/jlcs-es/b211d181f736cba89080bf0bd6d9c8da.js"></script>
