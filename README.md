## `Initiating the Contract`

Import threse contracts and inherit from "ICrossTalkApplication" contract for smart contract compatibility.

pragma solidity >=0.8.0 <0.9.0;

```sh
import "evm-gateway-contract/contracts/ICrossTalkApplication.sol";
import "evm-gateway-contract/contracts/Utils.sol";
import "@routerprotocol/crosstalk-utils/contracts/CrossTalkUtils.sol";

contract PingPong is ICrossTalkApplication {
}
```
