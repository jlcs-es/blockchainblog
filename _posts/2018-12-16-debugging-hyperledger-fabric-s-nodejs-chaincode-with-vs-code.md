---
layout: post
published: true
title: Debugging Hyperledger Fabric's NodeJS chaincode with VS Code
date: '2018-12-16'
background: /img/posts/fabriclovesvscode.png
---
## Development network

To test our chaincode we don't need a complex fabric network, only an endorsing peer, an orderer and optionally a client to query the endorser.

## Test chaincode

This is a simple [Typescript](https://www.typescriptlang.org/) chaincode, which compiles to pure JavaScript, but adds nice tools within VS Code, like autocompletion. Really recommended when learning to use the Fabric SDK.

<script src="https://gist.github.com/jlcs-es/64e1446838602a4369ecb0af525a2add.js"></script>

## VS Code debugger

Next, we need to write a [launch configuration for VS Code](https://code.visualstudio.com/Docs/editor/debugging), which tells it how to run our chaincode.

## Launch it all

## Automated script
