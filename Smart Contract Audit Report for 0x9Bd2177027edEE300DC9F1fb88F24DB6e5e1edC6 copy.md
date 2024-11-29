# Smart Contract Audit Report for 0x9Bd2177027edEE300DC9F1fb88F24DB6e5e1edC6

## Overview
- **Contract Address**: [0x9Bd2177027edEE300DC9F1fb88F24DB6e5e1edC6](https://etherscan.io/address/0x9Bd2177027edEE300DC9F1fb88F24DB6e5e1edC6)
- **Audited Functions**: `transfer()`, `approve()`.

## Issues Identified

### Issue 1: [High]
**Description:** 
Some tokens do not revert the transaction when the transfer or transferFrom fails and returns False. Hence we must check the return value after calling the transfer or transferFrom function.

**Proof of Concept:** 
The contract uses OpenZeppelin's ERC20 implementation which doesn't revert on failure for transfer or transferFrom by default

**Recommended Mitigation:**
- The SafeERC20 library from OpenZeppelin already provides methods like safeTransfer and safeTransferFrom which revert if the transfer fails.
- If the contract is meant to be used in an environment where you can't modify the external calls, you might consider overriding the transfer and transferFrom functions in your version of the ERC20 token to include the revert logic.

### Issue 2: [Low]
**Description:** 
Events are inheritable members of contracts. When you call them, they cause the arguments to be stored in the transaction’s log a special data structure in the blockchain.
These logs are associated with the address of the contract which can then be used by developers and auditors to keep track of the transactions.
The contract was found to be missing these events on the function which would make it difficult or impossible to track these transactions off-chain.

**Proof of Concept:** 
Although ERC20 has an Approval event, adding a custom event can include additional details or be more descriptive for specific use cases.

**Recommended Mitigation:**
- transfer, transferFrom, and approve are overridden to emit custom events alongside the standard events from ERC20. This allows for additional data logging (bytes data in TransferEvent could be used to pass extra transaction details).
transferOwnership and confirmTransferOwnership are also overridden to emit custom events that provide clear tracking of ownership changes.


## Additional Notes

- **Gas Optimizations**: 
  - Plus equals (+=) costs more gas than addition operator. The same thing happens with minus equals (-=)
  -	The require() and revert() functions take an input string to show errors if the validation fails.This strings inside these functions that are longer than 32 bytes require at least one additional MSTORE, along with additional overhead for computing memory offset, and other parameters.
  - Public constant variables cost more gas because the EVM automatically creates getter functions for them and adds entries to the method ID table. The values can be read from the source code instead. 
  - String constants are used for error messages, which consume more gas than using custom errors. 
  - The contract was found to be doing comparisons using inequalities inside the if statement.When inside the if statements, non-strict inequalities (>=, <=) are usually cheaper than the strict equalities (>, <)
  - The contract was found to be performing comparisons using inequalities inside the require statement. When inside the require statements, non-strict inequalities (>=, <=) are usually costlier than strict equalities (>, <).


## Conclusion

The audit of the smart contract at 0x9Bd2177027edEE300DC9F1fb88F24DB6e5e1edC6 revealed a high-severity issue related to the lack of transaction reversion upon transfer failures, which could compromise security. Immediate implementation of revert logic via SafeERC20 or custom modifications is advised. Additionally, the contract presents opportunities for gas optimization, which should be considered to improve efficiency. Addressing these findings will bolster the contract's security and performance. 

---

*Report generated on 28-11-2024*  
*Auditor: FelixP*