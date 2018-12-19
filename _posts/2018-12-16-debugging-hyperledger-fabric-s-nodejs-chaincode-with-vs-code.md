---
layout: post
published: true
title: Debugging Hyperledger Fabric's NodeJS chaincode with VS Code
date: '2018-12-16'
background: /img/posts/fabriclovesvscode.png
---
## How does debugging work in Hyperledger Fabric

In a normal deployment, when an endorser peer wants to run the chaincode, it uses a docker container to isolate the execution of the chaincode. This isolated container and the peer node communicate through the bridge network between them, therefore, the invocation arguments and the response travel through the wire.

```
+------------------------------------------------+
|  Peer Node                                     |
|                                                |
|         +-------------+       +-------------+  |
|         |             |       |             |  |
|         | Chaincode A |       | Chaincode B |  |
|         |             |       |             |  |
|         | container   |       | container   |  |
|         |             |       |             |  |
|         +-------------+       +-------------+  |
|                                                |
+------------------------------------------------+
```

This modularity where the peer doesn't execute the code, but instead another machine does (in this case a docker container), provides the tools for development debugging: instead of creating an isolated docker container, it uses any other machine that can run NodeJS and communicate with the peer.

In our case, we will run the peer node inside a docker container, and then run the chaincode within the host's NodeJS. The peer's docker container will expose the default port 7052 so the NodeJS process can communicate with it.

```
+---------------------------------------------------+
| Host machine (e.g. Ubuntu 18.04)                  |
|                                                   |
|    +-[Docker]------+                              |
|    |               |                              |
|    |  Peer node    |                              |
|    |               |                              |
|    |               |          +-[NodeJS]------+   |
|    |               |          |               |   |
|    |          7052 +----------+ Chaincode     |   |
|    |               |          |               |   |
|    +---------------+          +---------------+   |
|                                                   |
+---------------------------------------------------+
```

Now that the chaincode is running in our host's NodeJS, the VS Code debugger can connect to it, providing all the debugging tools, while the peer endorses the resulting transaction as in a real scenario.


## Development network

To test our chaincode we don't need a complex fabric network, only the endorsing peer, an orderer and optionally a client to query the endorser.

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
                   |  NodeJS   |
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

The key aspect is that the peer node is started with the option **`--peer-chaincodedev=true`**.

If you want to test your chaincode against the CouchDB database, the following docker-compose file sets up this simple network, but with a CouchDB database for the ledger:

<script src="https://gist.github.com/jlcs-es/163bf8cb09922a43befd90d6e5a86074.js"></script>


## Test chaincode

Instead of using one of the many demos available in the fabric-samples project, I will use a basic template from where you can start you own development.

This is a simple [Typescript](https://www.typescriptlang.org/) chaincode, which compiles to pure JavaScript, but adds nice tools within VS Code, like autocompletion. Really recommended when learning to use the Fabric SDK.

The `chaincode.ts` file is the main one. It imports the chaincode implementation and starts a peer chaincode process with it. The `shim.start()` function will wait until a peer node connects to it, in our case, the `peer-dev` node from the docker-compose network above.

The `mycc.ts` file naively implements the `ChaincodeInterface` with Typescript types and console output.

<script src="https://gist.github.com/jlcs-es/65eb47f459747828c6fe72cab9356c80.js"></script>

To compile the Typescript chaincode you can create a [VS Code task](https://code.visualstudio.com/Docs/editor/tasks) that automatically builds it after every save of the files:

<script src="https://gist.github.com/jlcs-es/56e55302897334bcc104e410fe631cde.js"></script>

## VS Code debugger

Next, we need to write a [launch configuration for VS Code](https://code.visualstudio.com/Docs/editor/debugging), which tells it how to run our chaincode in our local NodeJS installation and how to connect to the NodeJS debugger.

This launch configuration takes the compiled Typescript code, runs the `chaincode.js` main file with the **option `--peer.address localhost:7052`, which tells the `shim.start(...)` function where the peer node is**:

<script src="https://gist.github.com/jlcs-es/8006f329a17ff9ad2458a43f445b8dc1.js"></script>


## Launch it all

Now that we have our environment all set up, we can start debugging our code.

1.	Launch the blockchain network
	
    `docker-compose up`
    
2.	Launch the VS Code debugger

	In the left panel, go to the debugging tab and press the green play button next to the name of our launch configuration:

	![Start debugger button](/img/posts/launchdebugger.jpg){:width="50%" style="margin-left: 25%;"}
    
    At this point the `shim.start(...)` function will connect to the running peer at the exposed 7052 port. The peer, in development mode, now knows where to contact for chaincode invocations.
    
3.	_Install_ and instantiate the chaincode

	Although the chaincode already is _installed_ in your machine, the peer node still requires these steps to register the chaincode and then being able to call it.
    
    You can do it from the client container. First start an interactive terminal within the client container:
    
    `$ docker exec -it cli-dev bash`
    
    And then, run the install and instantiate commands:
    
    ```
    # peer chaincode install -l node -n mycc -p /opt/gopath/src/chaincodedev/chaincode -v v0
	# peer chaincode instantiate -o orderer:7050 -l node -n mycc -v v0 -c '{"Args":[]}' -C myc
    ```
    
4.	Make changes to the chaincode

    Once the chaincode has been instantiated, you don't need to redo the previous steps. Once you make a new change to the chaincode, the task will rebuild it (or you can compile it with `tsc`), and you only need to relaunch the debugger
    
    ![Restart debugger button](/img/posts/debugbar.jpg){:width="50%" style="margin-left: 25%;"}


## Automated script


<script src="https://gist.github.com/jlcs-es/b211d181f736cba89080bf0bd6d9c8da.js"></script>
