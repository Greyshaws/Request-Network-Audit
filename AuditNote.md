# QUANTSTAMP HACKATHON

# PROJECT NAME: REQUEST NETWORK

## AUDITOR NAME: GRACIOUS IGWE


| Repository | Commit | 
| -------- | -------- |
| [requestNetwork](https://github.com/RequestNetwork/requestNetwork/blob/53ebc7a54a610ac28f616f87e1bda6f3d1e3265f/packages/smart-contracts/src/contracts/ERC20EscrowToPay.sol)     | e7a48e4    |

## List of spec/docs
* https://solidity-by-example.org/
* https://certificate.quantstamp.com/full/prysm
* https://support.request.finance/essentials/escrow
* https://secureum.substack.com/p/solidity-101
* https://eip2535diamonds.substack.com/p/understanding-delegatecall-and-how
* https://jeancvllr.medium.com/solidity-tutorial-all-about-structs-b3e7ca398b1e

## List of issues:

1. * **Severity**: High
   * **Title**: Low level calls
   * **Description**:  The 'delegate call' and 'call' functions in Solidity are low-level functions that are used to make a call to another contract, and they can be potentially dangerous to use if not handled properly. They are generally less safe than using a direct call to another contract, because it allows you to execute arbitrary code in the target contract without knowing the contract's interface or having any guarantees about the return value. This can lead to unexpected behavior if the return value is not handled properly, or if the target contract's code has changed since the calling contract was deployed.
   * **File**: ERC20EscrowToPay.sol
   * **Recommendation**:To avoid this issue, it's generally recommended to use direct calls to other contracts whenever possible. If you do need to use the delegatecall function, you should be very careful to properly validate the input parameters and handle the return value correctly to prevent potential vulnerabilities. In some cases, you may be able to use the Solidity library feature to achieve the same functionality as delegatecall in a safer way. 


---

2. * **Severity**: High
   * **Title**: Privileged roles and ownership
   * **Description**: In a smart contract, state variables such as owner can be used to grant certain addresses with special privileges or roles. For example, the owner variable might be used to grant the owner of the contract the ability to modify the contract's behavior or to perform certain actions that are not available to other users. However, this also means that the owner's private key is a critical asset that needs to be protected. If an attacker were to gain access to the owner's private key, they could potentially perform actions on behalf of the owner that could harm end-users of the contract.
   * **File**: ERC20EscrowToPay.sol
   * **Recommendation**: To mitigate this risk, it's important to follow best practices for securing private keys and to use a hardware wallet or other secure storage solution to protect them. It's also a good idea to regularly review the privileges granted to the owner and to consider whether they are necessary, and to consider using alternative mechanisms for granting privileges (such as multisignature contracts or contract upgrades) that may be more secure.

---

3. * **Severity**: High
   * **Title**: Block Timestamp manipulation
   * **Description**: The contract relies on timestamps to determine the state of certain variables, such as the emergencyClaimDate and unlockDate fields in the Request struct. However, timestamps in Ethereum are not guaranteed to be accurate and can be manipulated by miners. This could potentially allow an attacker to manipulate the state of the contract or cause it to behave unexpectedly.
   * **File**: ERC20EscrowToPay.sol
   * **Recommendation**: Use an alternative mechanism, such as block numbers, to determine the state of time-sensitive variables.

---


4. * **Severity**: High
   * **Title**: No protection against frontrunning
   * **Description**: Here, an attacker could potentially observe an incoming transaction and execute a conflicting transaction before it is processed, potentially manipulating the state of the contract or causing it to behave unexpectedly.
   *  **File**: ERC20EscrowToPay.sol
   *  **Exploit**: In the context of the 'freezeRequest' function, an attacker could try to call the function just before the payer in order to freeze the escrow and prevent the payer from being able to release the payment to the payee.
   *  **Recommendation**: This could be mitigated by using commit-reveal scheme, or submarine send.


---

5. * **Severity**: Medium
   * **Title**: Missing input validation for various parameters and addresses
   * **Description**: Some public functions, such as 'payEscrow' and 'freezeRequest', do not check whether the token address or fee Address is a valid ERC20 contract. This can allow an attacker to set an arbitrary address as the token address, potentially leading to funds being sent to the attacker or other unintended consequences.
   * **File**: ERC20EscrowToPay.sol
   * **Exploit**: `function payEscrow(
    address _tokenAddress,
    address _to,
    uint256 _amount,
    bytes memory _paymentRef,
    uint256 _feeAmount,
    address _feeAddress
  ) external IsNotInEscrow(_paymentRef) {
    if (_amount == 0 || _feeAmount == 0) revert('Zero Value');`
    
Consider the above function, there are input validations for the _amount and _feeAmount variables, but there's none for _to and _feeAddress. An attacker can enter in a zero address or malicious address, leading to the funds being stolen or other attacks.
   * **Recommendation**: Implement a check to ensure that the token address is a valid ERC20 contract before creating a request.


---

6. * **Severity**: Medium
   * **Title**: Error handling mechanism
   * **Description**: Using the 'assert' keyword in a smart contract can potentially be a security vulnerability because it can be used to halt the execution of the contract if the condition passed to it is not met.
   * **File**: ERC20EscrowToPay.sol
   * **Recommendation**: Use "assert(x)" if you never ever want x to be false, not in any circumstance (apart from a bug in your code). Use "require(x)" if x can be false, due to e.g. invalid input or a failing external component.


---


7.  * **Severity**: Low
    * **Title**: Lack of gas optimization
    * **Description**: The contract may not be optimized for gas efficiency, which could potentially make it more expensive to use and less attractive to users. If the gas requirement of a function is higher than the block gas limit, it cannot be executed. 
    * **File**: ERC20EscrowToPay.sol
    * **Exploit**: For instance, this 'payEscrow' function:
    ``` 
    function payEscrow(
    address _tokenAddress,
    address _to,
    uint256 _amount,
    bytes memory _paymentRef,
    uint256 _feeAmount,
    address _feeAddress
    ) external IsNotInEscrow(_paymentRef) {
    if (_amount == 0 || _feeAmount == 0) revert('Zero Value');
    require(_to != address(0), 'payee address cannot be 0');

    requestMapping[_paymentRef] = Request(
      _tokenAddress,
      _to,
      msg.sender,
      _amount,
      0,
      0,
      false,
      false
    );

    (bool status, ) = address(paymentProxy).delegatecall(
      abi.encodeWithSignature(
        'transferFromWithReferenceAndFee(address,address,uint256,bytes,uint256,address)',
        _tokenAddress,
        address(this),
        _amount,
        _paymentRef,
        _feeAmount,
        _feeAddress
      )
    );
    require(status, 'transferFromWithReferenceAndFee failed');
    }
    ```
  
  The assignment to the requestMapping mapping is likely to be gas-intensive, especially if the mapping is large or the values being assigned are large
    * **Recommendation**: This can be mitigated by using more efficient algorithms, reducing the number of external contract calls, and minimizing the use of storage operations. Avoid loops in your functions or actions that modify large areas of storage (this includes clearing or copying arrays in storage). Also, consider using the procedural way to define a variable of struct type.

---

8. * **Severity**: Low
   * **Title**: Bounds check bypass
   * **Description**: A bounds check bypass vulnerability occurs when an attacker is able to manipulate the index of an array operation in such a way that they can read or write to an arbitrary location in storage. This can lead to a range of security issues, such as exposing sensitive data or allowing the attacker to modify the contract's behavior.
   * **File**: ERC20EscrowToPay.sol
   * **Exploit**: `delete requestMapping[_paymentRef];`
Using the 'delete' operator on an element of a dynamic array removes the element from the array and leaves a gap in its place. If the index is not within the bounds of the array before deleting the element, it could potentially allow an attacker to access elements of the array that are outside the bounds of the array, which could lead to a "bounds check bypass" vulnerability.
   * **Recommendation**: Make sure to validate all input parameters to ensure that they are within the expected range. For example, if you are using an array index, make sure to check that it is within the bounds of the array. And if you are using a 'delete' operator, you need to shift items manually and update the "length" property.

---

9. * **Severity**: Informational
    * **Title**: Unlocked Pragma
    * **Description**: Every Solidity file specifies in the header a version number of the format `pragma solidity (^)0.8.*`. The caret (`^`) before the version number implies an unlocked pragma, meaning that the compiler will use the specified version and above, hence the term "unlocked".
    * **File(s)**: ERC20EscrowToPay.sol
    * **Recommendation**: For consistency and to prevent unexpected behavior in the future, it is recommended to remove the caret to lock the file onto a specific Solidity version.

---

10. * **Severity**: Informational
    * **Title**: Missing event emission and critical state changes
    * **Description**: Some functions such as 'payEscrow' and modifiers lack event emissions, and this is necessary because these functions and modifiers make changes to the contract
    * **File**: ERC20EscrowToPay.sol
    * **Recommendation**: Consider adding event emissions to functions, to reflect critical state changes.

---

11. * **Severity**: Informational
    * **Title**: Missing function for retrieving emergency claim period
    * **Description**: The contract does not provide a function for retrieving the value of the 'emergencyClaimPeriod' variable. 
    * **File**: ERC20EscrowToPay.sol
    * **Recommendation**: Add a function that allows users to retrieve the value of the emergencyClaimPeriod variable.

---

12. * **Severity**: Informational
    * **Title**: Missing visibility level
    * **Description**: In Solidity, the default visibility level for functions and variables is internal, which means that they are only accessible from within the contract itself. However, if you want a function or variable to be accessible from other contracts or external calls, you must specify a different visibility level, such as public or external.
    * **File**: ERC20EscrowToPay.sol
    * **Recommendation**: To fix this issue, you should review the particular function and specify the appropriate visibility level for it. You can do this by adding the public, external, or private keyword before the function declaration. For example, you could modify the constructor function like this:
```
constructor(address _paymentProxyAddress, address _admin) public {
    paymentProxy = IERC20FeeProxy(_paymentProxyAddress);
    transferOwnership(_admin);
}
```

---

## Documentation issues
1. On Line 11, the contract's purpose is not clearly stated in the documentation. The brief description provided ("Request Invoice with Escrow") does not give enough context about what the contract does or how it is intended to be used.

2. The contract's variables (e.g. emergencyClaimPeriod, frozenPeriod) are insufficiently described in the documentation, so it is not exactly clear what they represent or how they are intended to be used. (Line154-160)

3. There is no documentation for the struct. It is not clear what its fields represent or how they are used. (line 18)

4. In https://support.request.finance/essentials/escrow, the documentation states that "freezing the escrow means that the money will remain in the contract for 12 months and after that, the client will be able to withdraw the full amount". But in the smart contract, the owner is given the option to set a frozen period. This means that the owner can set a period that is longer or shorter than 12 months. The frozen period should be a variable that is assigned the period of 12 months. (line158)

5. In https://support.request.finance/essentials/escrow, the documentation states that "the contractor can initiate an emergency claim on the smart contract. When this happens, after 6 months, the contractor will receive the money without the need for the client’s approval.". But in the smart contract, the owner is given the option to set an emergency claim period. This means that the owner can set a period that is longer or shorter than 6 months. The emergency claim period should be a variable that is assigned the period of 6 months. (line154)

6. In https://support.request.finance/essentials/escrow, the description of how the escrow process works is somewhat confusing and lacks detail. It mentions the creation of an invoice, but does not explain how the funds are actually deposited into the escrow contract or how the payee initiates the payment release.

7. In https://support.request.finance/essentials/escrow, the documentation does not explain how the smart contract handles situations where the payer or payee tries to initiate a payment release or emergency claim before the specified unlock date or emergency claim period. It also does not explain how the contract handles situations where the payer tries to initiate a payment release after the specified unlock date, but before the emergency claim period has ended.

8. In https://support.request.finance/essentials/escrow, the documentation does not explain how the smart contract handles situations where the payer or payee does not have sufficient funds in their wallet to complete the payment. It also does not explain how the contract handles situations where the payer or payee does not have sufficient allowance granted to the contract to transfer the funds.

9. In https://support.request.finance/essentials/escrow, the documentation does not provide any examples or case studies to illustrate how the escrow process works in practice. This would help readers understand the process better.

10. There is no documentation for the constructor in line 142

11. Missing documentation for functions 'setEmergencyClaimPeriod' and 'setFrozenPeriod' - Line 154 & 158 

---

## Best practices

1. Use of events: It is a good practice to use events and the emit keyword to trigger them so that they can be logged and listened to by external contracts or applications.
File: ERC20EscrowToPay.sol
Location: Functions like 'payEscrow', 'payRequestFromEscrow' etc do not have events and emissions.

2. Consider the gas cost of contract operations: Some functions in the contract perform multiple operations that may be costly in terms of gas. It is a good idea to consider the gas cost of contract operations and optimize them to minimize the cost for users. This can be done by using more efficient data structures or algorithms, or by minimizing the amount of data being stored in a mapping.
File: ERC20EscrowToPay.sol
Location: Line 185 - 194

3. More detailed explanations: Some of the explanations provided in the code documentation are brief and may not provide enough detail for someone unfamiliar with the code to fully understand its purpose and function. Adding more detailed explanations and examples could help to make the code more easily understandable.
File: ERC20EscrowToPay.sol
Location: Line 11

4. Depending on the complexity of the code, it may be helpful to create additional external documentation, such as a user guide or API reference, to provide more detailed information about how to use the code. This documentation could include step-by-step instructions for common tasks, as well as explanations of the different options and parameters that are available.
File: ERC20EscrowToPay.sol

5. It is recommended to use NATSPEC format when writing comments in Solidity
File: ERC20EscrowToPay.sol

6. It's generally a good practice to mark functions as constant, view, or pure whenever possible, especially if they do not modify the state of the contract or perform external calls or I/O operations. However, it's important to carefully consider the behavior of the function and whether it is appropriate to mark it as constant, view, or pure.
File: ERC20EscrowToPay.sol

7. Using "delete" on an array leaves a gap. The length of the array remains the same. If you want to remove the empty position you need to shift items manually and update the "length" property.
File: ERC20EscrowToPay.sol
Location: Line 370

8. Visibility level: For a function or variable to be accessible from other contracts or external calls, you must specify a different visibility level, such as public or external.
File: ERC20EscrowToPay.sol
Location: Line 142

9. Avoid using large data types: The bytes memory type used for the 'paymentRef' argument is a large data type that can consume a lot of gas when it is passed as a function argument or stored in memory or storage. If you need to use large data types in your function, consider using more efficient alternatives, such as fixed-size arrays or structs.
File: ERC20EscrowToPay.sol
Location: Line 179

10. It's generally recommended to use direct calls to other contracts whenever possible, instead of 'delegate call' or 'call'.
File: ERC20EscrowToPay.sol
Location: Line 196

11. Validate input parameters: Make sure to validate all input parameters to prevent potential vulnerabilities such as "reentrancy attacks" or "bounds check bypasses", and the use of zero addresses.
File: ERC20EscrowToPay.sol
Location: Line 135 & Line 139

12. Use trusted libraries and contracts: Use trusted libraries and contracts from reputable sources to help ensure that your code is secure
File: ERC20EscrowToPay.sol
Location: Line 4 - 7

13. Variable naming: field names in the 'Request' struct do not follow a consistent naming convention.
File: ERC20EscrowToPay.sol
Location: Line 18 - 26