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

<script src="https://gist.github.com/jlcs-es/65eb47f459747828c6fe72cab9356c80.js"></script>

## VS Code debugger

Next, we need to write a [launch configuration for VS Code](https://code.visualstudio.com/Docs/editor/debugging), which tells it how to run our chaincode.

## Launch it all

## Automated scripts

<script src="https://gist.github.com/jlcs-es/56e55302897334bcc104e410fe631cde.js"></script>

<script src="https://gist.github.com/jlcs-es/8006f329a17ff9ad2458a43f445b8dc1.js"></script>