#########  remove duplicated SLOAD from ActivePool._rebalance function ######### 

#### Impact
in the ActivePool._rebalance function, in line 240, you create vars struct as memory, then in line 243 you load storage variable yieldingAmount[_collateral] and save it to the memory vars.currentAllocated. but again in line 263, you are using SLOAD to read the value of yieldingAmount[_collateral] from storage.

#### Findings:
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L240

#### Tools used
manually

######### Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate ######### 

#### Impact
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. 

yieldingPercentage and yieldingAmount and yieldGenerator and yieldClaimThreshold are used together in the _rebalance function once and can be placed in a single slot:

#### Findings:
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L44
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L45
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L46
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L47

#### Tools used
manually

Sample report approved by C4 judges team.
https://github.com/code-423n4/2022-01-sandclock-findings/issues/55

#########  require() or revert() statements that check input arguments should be at the top of the function ######### 

#### Impact
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

in the ActivePool._rebalance function, in line 257, we call vars.finalBalance = collAmount[_collateral].sub(_amountLeavingPool); but this will get revert if _amountLeavingPool is more than collAmount[_collateral].

So we need to first check that _amountLeavingPool  is <= collAmount[_collateral], in order to prevent wasting a Gcoldsload (2100 gas*) in _rebalance function that may ultimately revert in the case than _amountLeavingPool is more than collAmount[_collateral].

#### Findings:
https://github.com/code-423n4/2023-02-ethos/blob/73687f32b934c9d697b97745356cdf8a1f264955/Ethos-Core/contracts/ActivePool.sol#L257

#### Tools used
manually

Sample report approved by C4 judges team.
https://code4rena.com/reports/2022-11-non-fungible/#g-09-require-or-revert-statements-that-check-input-arguments-should-be-at-the-top-of-the-function