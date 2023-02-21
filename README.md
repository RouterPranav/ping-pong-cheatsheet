## `Initiating the Contract`

Import threse external contracts and inherit from "ICrossTalkApplication" contract for smart contract compatibility.

```sh
pragma solidity >=0.8.0 <0.9.0;
import "evm-gateway-contract/contracts/ICrossTalkApplication.sol";
import "evm-gateway-contract/contracts/Utils.sol";
import "@routerprotocol/crosstalk-utils/contracts/CrossTalkUtils.sol";

contract PingPong is ICrossTalkApplication {
}
```

## `Creating State Variables and the Constructor`

Define state variables, events, custom errors for the contract, including a public variable to store the gateway contract address and a constructor to initialize the contract.

```sh
address public gatewayContract;
string public greeting;
uint64 public lastEventIdentifier;
uint64 public destGasLimit;
uint64 public ackGasLimit;

error CustomError(string message);
event ExecutionStatus(uint64 eventIdentifier, bool isSuccess);
event ReceivedSrcChainIdAndType(uint64 chainType, string chainID);

constructor(
	address payable gatewayAddress, 
	uint64 _destGasLimit, 
	uint64 _ackGasLimit
) {
  gatewayContract = gatewayAddress;
	destGasLimit = _destGasLimit;
	ackGasLimit = _ackGasLimit;
}
```


## `Sending a message to the destination chain`

Create pingDestination function with specified input parameters for initiating a cross-chain transaction with a destination contract on a different blockchain network.

```sh
  function pingDestination(
  uint64 chainType,
  string memory chainId,
  uint64 destGasPrice,
  uint64 ackGasPrice,
  bytes memory destinationContractAddress,
  string memory str,
  uint64 expiryDurationInSeconds
) public payable returns (uint64) {
  bytes memory payload = abi.encode(str);
  uint64 expiryTimestamp = uint64(block.timestamp) + expiryDurationInSeconds;
  Utils.DestinationChainParams memory destChainParams=
          Utils.DestinationChainParams(
	    destGasLimit, 
            destGasPrice, 
            chainType, 
            chainId
	);

  Utils.AckType ackType = Utils.AckType.ACK_ON_SUCCESS;
  Utils.AckGasParams memory ackGasParams = Utils.AckGasParams(ackGasLimit, ackGasPrice);
  
  CrossTalkUtils.singleRequestWithAcknowledgement(
      gatewayContract,
      expiryTimestamp,
      ackType,
      ackGasParams,
      destChainParams,
      destinationContractAddress,
      payload
  );
}
```
## `Handling a crosschain request`

create handleRequestFromSource function to handle a cross-chain request that arrives at the contract on the destination blockchain, by providing the required input parameters.

 ```sh
 function handleRequestFromSource(
  bytes memory srcContractAddress,
  bytes memory payload,
  string memory srcChainId,
  uint64 srcChainType
) external override returns (bytes memory) {
  require(msg.sender == gatewayContract);

  string memory sampleStr = abi.decode(payload, (string));

  if (
    keccak256(abi.encodePacked(sampleStr)) == keccak256(abi.encodePacked(""))
  ) {
    revert CustomError("String should not be empty");
  }
  greeting = sampleStr;
  return abi.encode(srcChainId, srcChainType);
}
```
## `Handling the acknowledgement received from destination chain`

createhandleCrossTalkAck function to handle acknowledgements sent by the destination chain to the source chain after a successful cross-chain communication. 

```sh
function handleCrossTalkAck(
    uint64 eventIdentifier,
    bool[] memory execFlags,
    bytes[] memory execData
  ) external view override {
    require(lastEventIdentifier == eventIdentifier);
		bytes memory _execData = abi.decode(execData[0], (bytes));

    (string memory chainID, uint64 chainType) = abi.decode(
      _execData,
      (string, uint64)
    );
		
    emit ExecutionStatus(eventIdentifier, execFlags[0]);
	
    emit ReceivedSrcChainIdAndType(chainType, chainID);
  }
  ```
