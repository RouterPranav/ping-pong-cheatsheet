# 🚀 `CrossTalk Cheatsheet`

## `Router Chain`

The Router Protocol bridges the gap between various layer 1 and layer 2 blockchains to improve cross-chain liquidity migration in DeFi. It facilitates the seamless transfer of tokens and contract-level data between different networks. The protocol can also initiate operations on one chain and execute them on another. This enhances the overall interoperability and usability of the DeFi ecosystem.

## `Router CrossTalk`

CrossTalk is a cross-chain framework by Router Protocol that facilitates communication between contracts on different chains. It can be easily integrated into your development environment, allowing for cross-chain message passing without affecting other parts of your product.

## `High Level Architecture`

<img width="487" alt="image" src="https://user-images.githubusercontent.com/124175970/221258104-5c65fe99-cf06-4171-8343-f8a67862a9d6.png">

## `Gateway Contracts`

Gateway contracts are contracts which are pre-deployed on supported blockchains for cross-chain communication.The source chain's gateway contract communicates with the destination chain's gateway contract, enabling communication between application contracts deployed on different chains.

## `CrossTalk Functions`

requestToDest 

The requestToDest function in Router's Gateway contracts enables cross-chain message transmission. To execute a cross-chain request, users can call this function and pass the payload and required parameters from the source to the destination chain. 

handleRequestFromSource

To process a cross-chain request on the destination chain, users need to have a handleRequestFromSource function on their destination chain contracts. This function enables the destination chain to receive the payload sent from the source chain through Router's Gateway contracts.

handleCrossTalkAck

To receive an acknowledgment of their cross-chain requests on the source chain, users need to implement a handleCrossTalkAck function on their source chain contracts. This function enables the source chain to receive the acknowledgment sent from the destination chain through Router's Gateway contracts.

## `requestToDest parameters`

<img width="401" alt="image" src="https://user-images.githubusercontent.com/124175970/221265218-a3bb1d4e-a4dc-4090-9b55-5852feb9a8e9.png">

requestArgs 
A struct having these parameter:-

expiryTimestamp: The timestamp by which your cross-chain call will expire
isAtomicCalls: setting to True enables Atomicity, setting to false disables Atomicity
feePayer: This specifies the address on the Router chain from which the cross-chain fee will be deducted.ackType :The Router chain receives an acknowledgment from the destination chain after executing contract calls, which can be retrieved by users to perform subsequent operations.

ackType 
<img width="413" alt="image" src="https://user-images.githubusercontent.com/124175970/221267619-5dada4e7-5708-42c4-ae7f-9e8d5b796b6a.png">

ackType=NO_ACK: No acknowledgment received on the source chain.
ackType=ACK_ON_SUCCESS: Acknowledgment only received on the source chain for successful execution on destination chain.
ackType=ACK_ON_ERROR: Acknowledgment only received on the source chain for failed execution on destination chain.
ackType=ACK_ON_BOTH: Acknowledgment received on the source chain for both successful and failed execution on destination chain.

ackGasParams
<img width="312" alt="image" src="https://user-images.githubusercontent.com/124175970/221267962-1f3bfa39-e91c-4a40-b187-2faac11ecf7d.png">

Includes the gas limit and gas price required to execute the handleRequestFromSource function on the source chain when the acknowledgment is received

destinationChainParams
<img width="361" alt="image" src="https://user-images.githubusercontent.com/124175970/221268423-1764185b-efad-4932-b205-579cb54f3971.png">

gasLimit: Required gas limit for executing cross-chain requests on the destination chain.
gasPrice: Gas price to be used on the destination chain.
destChainType: Represents the type of chain ( 0 for EVM Chains)
destChainId: Chain ID of destination chain, in string format.

contractCalls
<img width="352" alt="image" src="https://user-images.githubusercontent.com/124175970/221268851-dcd28ed8-9641-4ad8-b550-88bd440d67bf.png">

contains an array of payloads and contract addresses to be sent to the destination chain, with the payloads sent to their corresponding contract addresses


## `handleRequestFromSource`

<img width="365" alt="image" src="https://user-images.githubusercontent.com/124175970/221269189-4c7e1072-7275-404a-acb7-773f31f62da2.png">

includes source contract address, payload, source chain ID, and source chain type

## `handleCrossTalkAck`

<img width="339" alt="image" src="https://user-images.githubusercontent.com/124175970/221270488-49d95c1c-af70-4ac4-849f-93d1e19d4334.png">

eventIdentifier

The nonce can be used to verify if a request was executed on the destination chain.

execFlags:-

Atomic : In the case where 3 payloads are sent and the second payload fails, it returns [true, false, false].

Non-Atomic : In the case where 3 payloads are sent and the second payload fails, it returns [true, false, true].

## `execData`

Atomic : In the case where 3 payloads are sent and the second payload fails, it returns [returnData, errorData, emptyData].

Non-Atomic : In the case where 3 payloads are sent and the second payload fails, it returns [returnData, errorData, returnData].









