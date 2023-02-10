```sh
//SPDX-License-Identifier: UNLICENSED
pragma solidity >=0.8.0 <0.9.0;


import "evm-gateway-contract/contracts/IGateway.sol";
import "evm-gateway-contract/contracts/ICrossTalkApplication.sol";
import "evm-gateway-contract/contracts/Utils.sol";


contract PingPong is ICrossTalkApplication {
    uint64 public currentRequestId;
    mapping(uint64 => mapping(string => mapping(uint64 => string)))
        public pingFromSource;
    mapping(uint64 => string) public ackFromDestination;
    IGateway public gatewayContract;
    uint64 public destGasLimit;
    uint64 public ackGasLimit;
    error CustomError(string message);
    event PingFromSource(
        uint64 indexed srcChainType,
        string indexed srcChainId,
        uint64 indexed requestId,
        string message
    );
    event NewPing(uint64 indexed requestId);

    event ExecutionStatus(uint64 indexed eventIdentifier, bool isSuccess);
    event AckFromDestination(uint64 indexed requestId, string ackMessage);

    constructor(
        address payable gatewayAddress,
        uint64 _destGasLimit,
        uint64 _ackGasLimit
    ) {
        gatewayContract = IGateway(gatewayAddress);
        destGasLimit = _destGasLimit;
        ackGasLimit = _ackGasLimit;
    }

    
    function pingDestination(
        uint64 chainType,
        string memory chainId,
        uint64 destGasPrice,
        uint64 ackGasPrice,
        address destinationContractAddress,
        string memory str,
        uint64 expiryDurationInSeconds
    ) public payable {
        currentRequestId++;
       
        bytes memory payload = abi.encode(currentRequestId, str);
        
        uint64 expiryTimestamp = uint64(block.timestamp) +
            expiryDurationInSeconds;

       
        bytes[] memory addresses = new bytes[](1);
        addresses[0] = toBytes(destinationContractAddress);

      
        bytes[] memory payloads = new bytes[](1);
        payloads[0] = payload;

       
        _pingDestination(
            expiryTimestamp,
            destGasPrice,
            ackGasPrice,
            chainType,
            chainId,
            payloads,
            addresses
        );

        emit NewPing(currentRequestId);
    }

    function _pingDestination(
        uint64 expiryTimestamp,
        uint64 destGasPrice,
        uint64 ackGasPrice,
        uint64 chainType,
        string memory chainId,
        bytes[] memory payloads,
        bytes[] memory addresses
    ) internal {
        
        gatewayContract.requestToDest(
            expiryTimestamp,
            false,
            Utils.AckType.ACK_ON_SUCCESS,
            Utils.AckGasParams(ackGasLimit, ackGasPrice),
            Utils.DestinationChainParams(
                destGasLimit,
                destGasPrice,
                chainType,
                chainId
            ),
            Utils.ContractCalls(payloads, addresses)
        );
    }

    function handleRequestFromSource(
        bytes memory srcContractAddress,
        bytes memory payload,
        string memory srcChainId,
        uint64 srcChainType
    ) external override returns (bytes memory) {
        require(msg.sender == address(gatewayContract));

        (uint64 requestId, string memory sampleStr) = abi.decode(
            payload,
            (uint64, string)
        );

        if (
            keccak256(abi.encodePacked(sampleStr)) ==
            keccak256(abi.encodePacked(""))
        ) {
            revert CustomError("String should not be empty");
        }

        pingFromSource[srcChainType][srcChainId][requestId] = sampleStr;

        emit PingFromSource(srcChainType, srcChainId, requestId, sampleStr);

        return abi.encode(requestId, sampleStr);
    }

   
    function handleCrossTalkAck(
        uint64 eventIdentifier,
        bool[] memory execFlags,
        bytes[] memory execData
    ) external override {
        bytes memory _execData = abi.decode(execData[0], (bytes));

        (uint64 requestId, string memory ackMessage) = abi.decode(
            _execData,
            (uint64, string)
        );

        ackFromDestination[requestId] = ackMessage;

        emit ExecutionStatus(eventIdentifier, execFlags[0]);
        emit AckFromDestination(requestId, ackMessage);
    }

    
    function toBytes(address a) public pure returns (bytes memory b) {
        assembly {
            let m := mload(0x40)
            a := and(a, 0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF)
            mstore(
                add(m, 20),
                xor(0x140000000000000000000000000000000000000000, a)
            )
            mstore(0x40, add(m, 52))
            b := m
        }
    }
}
```
