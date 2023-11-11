![](https://github.com/EdwinLiavaa/How-To-Use-RPCs-on-COSMOS/blob/main/files/header.png?raw=true)
# How-To-Use-RPCs-on-COSMOS
## Introducing RPCs and how to use them specifically in the Cosmos Ecosystem:

In Web3, dApps (decentralized applications) need a way to communicate with blockchains. Without a means of communication, dApps won’t be able to access information and make transactions on the blockchain on which they operate. This connection between the blockchain and the dApps is facilitated by RPC nodes.

A remote procedure call (RPC) is a communication protocol used primarily between a client and a server.
## The 3 definitions of RPC that are often used interchangeably:

1. The RPC **runtime**: a runtime environment providing for communication facilities between
computers.
1. The RPC **exchange**: a set of request-and-response message exchanges between computers.
2. The RPC **message**: the single message from an RPC exchange

## Web2 v Web3 and the use of RPCs:

The fundamental existence of a Blockchain is driven by the notion that each client can run its own node, which communicates and maintains a copy of the latest “State” of the network. In contrast to Web2, applications communicate with back-end servers that connect to databases using for example ODBC or ADO to store and access data. In Web3, applications access blockchain data using nodes, and communication with these nodes are done through RPCs.

All in all, in order to access the “State”, the nodes expose RPC endpoints (network-specific address of a server process). The RPC nodes can be run locally or accessed remotely, but due to the impracticality of privately running and maintaining a node, blockchain data is typically accessed through node provider services or public endpoints supported by ecosystem grants.

## RPCs in the Cosmos Ecosystem:

In the Cosmos ecosystem, RPC nodes are required for applications to:

1. Read data from the blockchain and present it to the users.
2. Receive signed user transactions from wallets and propagate these to the rest of the peer-to-peer (P2P) network, to be used by the validators in building the blocks.
---
![](https://github.com/EdwinLiavaa/How-To-Use-RPCs-on-COSMOS/blob/main/files/RPCs1.png?raw=true)

## The 5 steps as sequence of events when making an RPC method call:

1. A client is interested in running a function on a remote server.
2. An RPC call is initiated by the client with all the relevant data (function name, parameters, etc.).
3. The call is sent to the target remote server which processes the request and executes the procedure.
4. The remote server sends a response to the client.
5. The client is now able to use the response in order to run its application logic

## The Tendermint RPC:

In the Cosmos ecosystem, developers use the Cosmos Software Development Kit (SDK) to build appchains. These appchains implement a consensus engine, Tendermint Core, which includes a Byzantine-Fault Tolerant consensus algorithm for ensuring that the same transactions are recorded on every machine in the same order.

In addition, every appchain requires nodes to expose RPC endpoints to serve client queries for blockchain data, as well as propogate transactions to other nodes in the network. On Cosmos, a commonly used endpoint is the Tendermint RPC endpoint, which is independent from the Cosmos SDK. Every node in the network can choose whether to expose their Tendermint RPC endpoints, which will be served by default on port 26657.

## Clients can use this endpoint to retrieve up-to-date “State” data in 3 different ways:

1. URI over HTTP:
```js
curl localhost:26657/block?height=5
```

2. JSON-RPC over HTTP (post request):
```js
curl --header "Content-Type: application/json" --request POST --data '{"method":"block", "params": ["5"], "id": 1}' localhost:266573
```

3. JSON-RPC over WebSocket (for async functions):
```js
ws ws://localhost:26657/websocket
> { "jsonrpc": "2.0", "method": "subscribe", "params": ["tm.event='NewBlock'"], "id": 1 }
```

Another common way to interact with a node on Cosmos is to use gRPC, a language agnostic, highperformance Remote Procedure Call (RPC) framework but there is a differences between gRPC and RPC.

Like Ethereum, the stateless and lightweight JSON-RPC remote procedure call (RPC) protocol is used for distributed client-server communication on the Cosmos ecosystem with JSON (RFC 4627) as its data format allowing:

1. Normal calls (Request — Response form).
2. Notifications (data sent to the server that does not require a response).
3. Multiple calls to be sent to the server which may be answered asynchronously.
---
![](https://github.com/EdwinLiavaa/How-To-Use-RPCs-on-COSMOS/blob/main/files/RPCs2.png?raw=true)

## Examples – How RPCs work using the Cosmos SDK Stargate in CosmJS compared to Ethereum:

## First Install CosmJS `^0.24.0`

```
npm install @cosmjs/proto-signing @cosmjs/stargate
# or
yarn add @cosmjs/proto-signing @cosmjs/stargate
```
## How to send your first Stargate transaction from JavaScript:

```js
import { DirectSecp256k1HdWallet } from "@cosmjs/proto-signing";
import { assertIsBroadcastTxSuccess, SigningStargateClient, StargateClient } from "@cosmjs/stargate";

const mnemonic = "surround miss nominee dream gap cross assault thank captain prosper drop duty group candy wealth weather scale put";
const wallet = await DirectSecp256k1HdWallet.fromMnemonic(mnemonic);
const [firstAccount] = await wallet.getAccounts();

const rpcEndpoint = "https://rpc.my_tendermint_rpc_node";
const client = await SigningStargateClient.connectWithSigner(rpcEndpoint, wallet);

const recipient = "cosmos1xv9tklw7d82sezh9haa573wufgy59vmwe6xxe5";
const amount = {
  denom: "ucosm",
  amount: "1234567",
};
const result = await client.sendTokens(firstAccount.address, recipient, [amount], "Have fun with your star coins");
assertIsBroadcastTxSuccess(result);
```

## How to send your custom message types:

```js
import { DirectSecp256k1HdWallet, Registry } from "@cosmjs/proto-signing";
import {
  assertIsBroadcastTxSuccess,
  SigningStargateClient,
  StargateClient,
} from "@cosmjs/stargate";

// A message type auto-generated from .proto files using ts-proto. @cosmjs/stargate ships some
// common types but don't rely on those being available. You need to set up your own code generator
// for the types you care about. How this is done should be documented, but is not yet:
// https://github.com/cosmos/cosmjs/issues/640
import { MsgDelegate } from "@cosmjs/stargate/build/codec/cosmos/staking/v1beta1/tx"; 

const mnemonic =
  "surround miss nominee dream gap cross assault thank captain prosper drop duty group candy wealth weather scale put";
const wallet = await DirectSecp256k1HdWallet.fromMnemonic(mnemonic);
const [firstAccount] = await wallet.getAccounts();

const rpcEndpoint = "https://rpc.my_tendermint_rpc_node";
const client = await SigningStargateClient.connectWithSigner(
  rpcEndpoint,
  wallet,
  { registry: registry }
);

const msg = MsgDelegate.create({
  delegatorAddress: alice.address0,
  validatorAddress: validator.validatorAddress,
  amount: {
    denom: "uatom",
    amount: "1234567",
  },
});
const msgAny = {
  typeUrl: msgDelegateTypeUrl,
  value: msg,
};
const fee = {
  amount: [
    {
      denom: "uatom",
      amount: "2000",
    },
  ],
  gas: "180000", // 180k
};
const memo = "Use your power wisely";
const result = await client.signAndBroadcast(
  firstAccount.address,
  [msgAny],
  fee,
  memo
);
assertIsBroadcastTxSuccess(result);
```
## How to get the balance address:

```js
import { Coin, createProtobufRpcClient, QueryClient } from "@cosmjs/stargate";
import { QueryClientImpl } from "cosmjs-types/cosmos/bank/v1beta1/query";
import { Tendermint34Client } from "@cosmjs/tendermint-rpc";

const getBalance = async (denom: string, address: string, rcp: string):
    Promise<Coin | undefined> => {
    try {
        const tendermint = await Tendermint34Client.connect(rcp);
        const queryClient = new QueryClient(tendermint);
        const rpcClient = createProtobufRpcClient(queryClient);
        const bankQueryService = new QueryClientImpl(rpcClient);
        const { balance } = await bankQueryService.Balance({
            address,
            denom,
        });
    return balance;
    } catch (error) {
        console.log(error);
    }
};
```
### How to get the balance:

```js
const balance = getBalance(
    "uatom",
    "cosmos1peydkky2dcj0n8mc9nl4rx0qwwmwph94mkp6we",
    "https://cosmos-mainnet-rpc.allthatnode.com:26657/"
);
```
## Summary:

The above examples focus on signing, but there are plenty of things you can build with CosmJS and only some of them require signing.

- Wallets (hot and cold; with and without a UI)
- Explorers
- Scrapers
- Key/Address generators
- dApps
- IBC relayers

## Sources:
- https://medium.com/lava-network/introducing-lava-a1597b244b9f
- https://medium.com/lava-network/a-developers-guide-to-accessing-blockchain-data-with-cosmos-rpc-examples-f86bc62e1f91
- https://ethereum.org/en/developers/docs/apis/json-rpc/
- https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-even6/fda52b23-608b-444e-a979-6b62a00e8f48
- https://www.blockmeadow.com/complete-ethereum-rpc-calls-list-with-examples/
- https://github.com/cosmos/cosmjs#using-custom-stargate-modules







