
MultiOwnable.sol


1)Gas Inefficiencies in isOwnerAddress and isOwnerPublicKey Functions
In these functions, unnecessary encoding of addresses or public keys leads to gas inefficiencies. Solidity mappings allow direct access using addresses or keys without the need for additional encoding. You can directly use the address as the key in the mapping, eliminating the need for encoding and reducing gas consumption.

function isOwnerAddress(address account) public view virtual returns (bool) {
    return _getMultiOwnableStorage().isOwner[account];
}

2)Redundant Encoding in _initializeOwners Function
The _initializeOwners function redundantly encodes addresses, which adds unnecessary gas costs. Since the input parameter is already an array of addresses, encoding them again is redundant. You  should avoid encoding addresses when unnecessary to optimize gas consumption.

function _initializeOwners(address[] memory owners) internal virtual {
    for (uint256 i; i < owners.length; i++) {
        if (owners[i] == address(0)) {
            revert InvalidOwnerBytesLength(abi.encode(owners[i]));
        }
    }
}

3) Gas Consumption in _addOwnerAtIndex Function
In the _addOwnerAtIndex function, separate storage assignments increase gas consumption. Combining them into a single operation can reduce gas costs. You should combine storage assignments to minimize gas consumption.

function _addOwnerAtIndex(bytes memory owner, uint256 index) internal virtual {
    if (isOwnerBytes(owner)) revert AlreadyOwner(owner);

    MultiOwnableStorage storage storageRef = _getMultiOwnableStorage();
    storageRef.isOwner[owner] = true;
    storageRef.ownerAtIndex[index] = owner;

    emit AddOwner(index, owner);
}

