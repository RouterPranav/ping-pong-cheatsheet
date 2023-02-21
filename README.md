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
