## Project Hydro (HydroBlockchain)

### About
Hydro is the fintech blockchain providing a security and identity protocol for 2FA, MFA, KYC, payments, and documents.
Project Hydro is web3, multi-chain decentralized ecosystem which uses cutting-edge cryptographic technology to secure user identities, accounts, and transactions. Hydro aims to decentralize financial services by bringing public blockchain technology to traditional private systems. It also allows developers around the world to boost their platforms and applications with this innovative technology.
Hydro is an open source blockchain project comprising of multiple smart contracts (protocols) and a dApp store, all powered by the HYDRO token.

## IceFMS.sol
It is a library built to creating sorting order for maximizing space utilization for robust File Management System.

The contract was written in version 0.5.1. The *pragma* keyword was used to enable compiler checks.

Five files were imported which includes:
1. **SafeMath.sol**
Mathematical operations with safety checks that revert on errors like overflow for multiplication, division by zero, overflow for addition and subtraction and dividing by zero for modulus. 

2. **SafeMath8.sol**
Same as SafeMath.sol but for uint8.

3. **IceGlobal.sol**
handle critical file management function. Function that helps to get global items info, to get global items stamping info from the entire File Management System of Ice. It also contains file to get global iems of a file, global indexes of a files, reserve and return global item slot, add items to the global items, delete items, transfer history of EINs and many more. This is where all functions related to the global scope of ice is stored. IceGlobal.sol imported **IdentityRegistryInterface.sol:** this is an interface name IdentityRegistryInterface. It contains isSigned function, Identity View Functions, Identity Management Functions, Recovery Management Functions.

4. **IceSort.sol**
this is the core file that handles the sorting of files using LinkedList. It is used to preserve the order of the files and check the validity of the order for the arrangement of the files


5. **IceFMSAdv.sol**
this imported SafeMath.sol, IceGlobal.sol and IceSort.sol


```solidity 
using SafeMath for uint;
using IceGlobal for IceGlobal.GlobalRecord;
using IceGlobal for IceGlobal.Association;
using IceGlobal for IceGlobal.UserMeta;
using IceGlobal for mapping (uint => bool);
using IceGlobal for mapping (uint8 => IceGlobal.ItemOwner);
using IceGlobal for uint;
using IceSort for mapping (uint => IceSort.SortOrder);
```
This is using all the libraries imported into he contract.

### Events
```solidity
event SharingCompleted(uint EIN, uint index1, uint index2, uint recipientEIN);

event SharingRejected(uint EIN, uint index1, uint index2, uint recipientEIN);

event SharingRemoved(uint EIN, uint index1, uint index2, uint recipientEIN);
```
i. event SharingCompleted(uint EIN, uint index1, uint index2, uint recipientEIN): is emitted when the sharing is completed.
ii. event SharingRejected(uint EIN, uint index1, uint index2, uint recipientEIN); is emitted when the sharing is rejected.
iii. event SharingRemoved(uint EIN, uint index1, uint index2, uint recipientEIN); is emitted when the sharing is removed.

1. shareItemToEINs()
	This function is used to share  an item to many users, and this function is usually called by the owner of the item to be shared. It has eight parameters which includes:
	i. self: is the mappings of all pointer to the GlobalRecord Struct (IceGlobal Library) which forms shares in Ice Contract. The GlobalRecord struct has member uint i1 and uint i2, used to access global index 1 and 2.
	ii. _globalItems: this is the mappings of all items stored by all users.
	iii. _totalShareOrderMapping: mapping of the entire shares order using SortOrder Struct (IceSort Library).
	iv. _shareCountMapping: mapping of all share count.
	v. _blacklist: is the entire mapping of the Blacklist for all users.
	vi. _rec: is the GlobalRecord Struct (IceGlobal Library).
	vii. _ein is the primary user who initiates sharing of the item.
	viii. _toEINs is the array of the users with whom the item will be shared.

2. _shareItemToEIN()
	This function is used to share an item to a specific user, and this function is usually called by the owner of the item to be shared. It has five parameters which includes:
    i. self: is the mappings of all shares associated with the recipient user.
    ii. _globalItems: is the mapping of all items stored by all users in the Ice FMS
    iii. _shareOrder: is the mapping of the shares order using SortOrder Struct (IceSort Library) of the recipient user
    iv. _shareCount: is the mapping of all share count
    v. _rec: is the GlobalRecord Struct (IceGlobal Library)
    vi. _toEIN: is the ein of the recipient user
    And there are many more functions in the library.

## IceFMS Library
```solidity
using SafeMath for uint;
using SafeMath8 for uint8;

using IceGlobal for IceGlobal.GlobalRecord;
using IceGlobal for IceGlobal.Association;
using IceGlobal for IceGlobal.UserMeta;
using IceGlobal for mapping (uint => bool);
using IceGlobal for mapping (uint8 => IceGlobal.ItemOwner);
using IceGlobal for mapping (uint => mapping (uint => IceGlobal.Association));
using IceGlobal for uint;

using IceSort for mapping (uint => IceSort.SortOrder);
using IceSort for IceSort.SortOrder;

using IceFMSAdv for mapping (uint => mapping(uint => IceGlobal.GlobalRecord));
```
This is using all the libraries imported into he contract.
### Structs

#### FileMeta
This is a struct called *FileMeta* which define the multihash function for storing of hash. The members of the struct includes:
1. bytes32 name: stores the name of the file
2. bytes32 hash: stores the hash of the file
3. bytes22 hashExtraInfo: stores any extra info for the hash of the file
4. bool encrypted: whether the file is encrypted or not
5. bool markedForTransfer: mark the file as transferred. The transfer status of the file.
6. uint8 protocol: stores the protocol of the file stored, where; 0 is URL, 1 is IPFS.
7. uint8 transferCount: to maintain the transfer count for mapping
8. uint8 hashFunction: store the hash of the file for verification, where 0x000 is for deleted files.
9. uint8 hashSize: store the length of the digest; i.e the length of the hash
10. uint32 timestamp: to store the timestamp of the block when file is created
```solidity
struct FileMeta {
    bytes32 name;
    bytes32 hash;
    bytes22 hashExtraInfo;
    bool encrypted;
    bool markedForTransfer;
    uint8 protocol;
    uint8 transferCount;
    uint8 hashFunction;
    uint8 hashSize;
    uint32 timestamp;
}
```

#### File
This is a struct called *File* which actually stores the information about the File. The members of the struct includes:

1. IceGlobal.GlobalRecord rec: store the association in global record. GlobalRecord is a struct in the IceGlobal.sol
2. bytes protocolMeta: store metadata of the protocol
3.  FileMeta fileMeta: store metadata associated with file
4. File Properties - Encryption Properties mapping (address => bytes32) encryptedHash;
5. uint associatedGroupIndex: to store the group index of the group that holds the file
6. uint associatedGroupFileIndex: to store the mapping of file in the specific group order
7. uint transferEIN: to record EIN of the user to whom trasnfer is inititated
8. uint transferIndex: to record the transfer specific index of the transferee
9. mapping (uint => uint) transferHistory; To maintain histroy of transfer of all EIN
```solidity
struct File {
    IceGlobal.GlobalRecord rec; 
    bytes protocolMeta;
    FileMeta fileMeta;
    mapping (address => bytes32) encryptedHash; 
    uint associatedGroupIndex;
    uint associatedGroupFileIndex;
    uint transferEIN;
    uint transferIndex;
    mapping (uint => uint) transferHistory;
}
```

### Group
This is a struct called “Group” which the main purpose is to sort the files in linear group like a folder. 0 or default groupID is root. The members of the struct includes:
1. IceGlobal.GlobalRecord rec: store the association in global record.
2. string name: the name of the Group
3. mapping (uint => IceSort.SortOrder) groupFilesOrder: the order of files in the current group
4. uint groupFilesCount: to keep the count of group files
```solidity
struct Group {
    IceGlobal.GlobalRecord rec;
    string name;
    mapping (uint => IceSort.SortOrder) groupFilesOrder;
    uint groupFilesCount;
}
```

## Function

### File Function
1. getFileInfo(): the function visibility is external. It is a view function. It takes in a parameter “self” with a type of “File” Struct. This function is used to get the file information of an EIN (a particular user) like protocol, protocolMeta, file name, file hash, extra info of hash, the function to store the hash, size of the hash and the status of the encryption.
    ```solidity
    function getFileInfo(File storage self) external view returns (
        uint8 protocol, bytes memory protocolMeta, string memory fileName, bytes32 fileHash, 
        bytes22 hashExtraInfo, uint8 hashFunction, uint8 hashSize, bool encryptedStatus
    ) {
        protocol = self.fileMeta.protocol;
        protocolMeta = self.protocolMeta;

        fileName = bytes32ToString(self.fileMeta.name);
        fileHash = self.fileMeta.hash;

        hashExtraInfo = self.fileMeta.hashExtraInfo;
        hashFunction = self.fileMeta.hashFunction;
        hashSize = self.fileMeta.hashSize;
        
        encryptedStatus = self.fileMeta.encrypted;
    }
    ```

2. getFileOtherInfo(): this function used to get more information about an EIN that were not gotten in the previous function like timestamp of the file, group in which the file is associated to in the user’s FMS. The function visibility is external. It is a view function. It takes in a parameter “self” with a type of “File” Struct.
    ```solidity
    function getFileOtherInfo(File storage self) external view returns (
        uint32 timestamp, uint associatedGroupIndex, uint associatedGroupFileIndex
    ) {
        timestamp = self.fileMeta.timestamp;
        associatedGroupIndex = self.associatedGroupIndex;
        associatedGroupFileIndex = self.associatedGroupFileIndex;
    }
    ```
3. getFileTransferInfo(): the function is used to get the transfer info of a file for an EIN. It returns the number of times the file has been transferred, the EIN of the user in which the file is currently scheduled for transfer, the transfer index of the target EIN where the file is currently mapped, and indicates if the file is marked for transfer or not. The function visibility is external. It is a view function. It takes in a parameter “self” with a type of “File” Struct.
    ```solidity
    function getFileTransferInfo(File storage self) external view returns (
        uint transferCount, uint transferEIN, uint transferIndex, bool markedForTransfer
    ) {
        transferCount = self.fileMeta.transferCount; 
        transferEIN = self.transferEIN; 
        transferIndex = self.transferIndex; 
        markedForTransfer = self.fileMeta.markedForTransfer;
    }
    ```
4. getFileTransferOwners(): this function is used to get the file transfer info of an EIN. It takes in two parameters: “self” with a type of “File” Struct, and _transferIndex. It returns the EIN of the user who originally owned that file.  The function visibility is external. It is a view function.
    ```solidity
    function getFileTransferOwners(File storage self, uint _transferIndex) external view returns (uint previousOwnerEIN) {
        previousOwnerEIN = self.transferHistory[_transferIndex];
    }
    ```
5. createFileObject(): the function is used to create a basic file object for a given file. It takes in four parameters: “self” with a type of “File” Struct with a location of storage, _protocolMeta which is bytes, _groupIndex and _groupFilesCount.
    ```solidity
    function createFileObject(File storage self, bytes calldata _protocolMeta, uint _groupIndex, uint _groupFilesCount) external {
        self.protocolMeta = _protocolMeta;
        self.associatedGroupIndex = _groupIndex;
        self.associatedGroupFileIndex = _groupFilesCount;
    }
    ```

6. createFileMetaObject(): this function is used to create a File Meta Object and attach it to File Struct
    ```solidity
    function createFileMetaObject(
        File storage self, uint8 _protocol, bytes32 _name, bytes32 _hash,
        bytes22 _hashExtraInfo, uint8 _hashFunction, uint8 _hashSize, bool _encrypted
        ) external {
        self.fileMeta = FileMeta(
            _name,
            _hash,
            _hashExtraInfo,
            _encrypted,
            false,
            _protocol,
            1,
            _hashFunction,
            _hashSize,
            uint32(now)
        );
    }
    ```

7. writeFile(): it is used to write file to a user FMS and returns the updated file count. It calls a function **addFileToGroup** used to add file to a group, and **addToSortOrder function** used to facilitate adding of double linked list used to preserve order and form cicular linked list and the file was imported from IceSort.sol
    ```solidity
    function writeFile(
        File storage self, Group storage group, uint _groupIndex, 
        mapping(uint => IceSort.SortOrder) storage _userFileOrderMapping, 
        uint _maxFileIndex, uint _nextIndex, uint _transferEin, bytes32 _encryptedHash
    ) external returns (uint newFileCount) {
        (self.associatedGroupIndex, self.associatedGroupFileIndex) = addFileToGroup(group, _groupIndex, _nextIndex);
        self.encryptedHash[msg.sender] = _encryptedHash;
        self.transferHistory[0] = _transferEin;
        newFileCount = _userFileOrderMapping.addToSortOrder(_userFileOrderMapping[0].prev, _maxFileIndex, 0);
    }
    ```

8. moveFileToGroup(): this function is used to move file from one group to another and returns the group file index using a function called inside this function block called **remapFileToGroup()** that helps to remap file from one group to another. First, the function check if the new group is valid using a function called “condVaidSortOrder()” from IceSort.sol to check that the group order is valid. It also check whether a file has been stamped or not, an unstamped file can’t be moved.
    ```solidity
    function moveFileToGroup(
        File storage self, uint _fileIndex,
        mapping(uint => IceFMS.Group) storage _groupMapping, 
        mapping(uint => IceSort.SortOrder) storage _groupOrderMapping,
        uint _newGroupIndex,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems
    ) external returns (uint groupFileIndex){
        _groupOrderMapping[_newGroupIndex].condValidSortOrder(_newGroupIndex);
        self.rec.getGlobalItemViaRecord(_globalItems).condUnstampedItem();
        _groupMapping[self.associatedGroupIndex].rec.getGlobalItemViaRecord(_globalItems).condUnstampedItem(); 
        
        _groupMapping[_newGroupIndex].rec.getGlobalItemViaRecord(_globalItems).condUnstampedItem(); 
        
        groupFileIndex = remapFileToGroup(
            self, 
            _fileIndex,
            _groupMapping[self.associatedGroupIndex], 
            _groupMapping[_newGroupIndex], 
            _newGroupIndex
        );
    }
    ```

9. _deleteFileMappings(); this is an internal function that is used to delete file mapping from a user’s file management system. It first remove all item shared to other used then later remove that file from the global record.
    ```solidity
     function _deleteFileMappings(
        uint _ein,
        IceGlobal.Association storage _globalItemIndividual,
        mapping (uint => mapping(uint => IceGlobal.GlobalRecord)) storage _totalSharesMapping,
        mapping (uint => mapping(uint => IceSort.SortOrder)) storage _totalShareOrderMapping, 
        mapping (uint => uint) storage _shareCountMapping
    ) internal {
        _totalSharesMapping.removeAllShares(
            _ein,
            _globalItemIndividual, 
            _totalShareOrderMapping, 
            _shareCountMapping
        );
        
        _globalItemIndividual.deleteGlobalRecord();
    }
    ```
    
10. _deleteFileObject(): is an internal function used to delete the file object. It first remove the file from the group that holds the file
    ```solidity
    function _deleteFileObject(
        mapping (uint => File) storage _files,
        uint _ein, uint _fileIndex,
        mapping (uint => IceSort.SortOrder) storage _fileOrderMapping,
        mapping (uint => uint) storage _fileCountMapping,
        Group storage _fileGroup
    )
    internal {
        removeFileFromGroup(
            _fileGroup, 
            _files[_fileIndex].associatedGroupFileIndex
        );

        _files[_fileIndex] = _files[_fileCountMapping[_ein]];
        _fileCountMapping[_ein] = _fileOrderMapping.stichSortOrder(_fileIndex, _fileCountMapping[_ein], 0);
    }
    ```

11. deleteFile(): this function deletes the file of a particular owner. It uses **_deleteFileMappings()**; which is an internal function to delete the file from global mapping and uses **_deleteFileObject()** to delete the file object.
    ```solidity
    function deleteFile(
        mapping (uint => File) storage self,
        uint _ein, uint _fileIndex,
        IceGlobal.Association storage _globalItemIndividual,
        
        mapping (uint => IceSort.SortOrder) storage _fileOrderMapping,
        mapping (uint => uint) storage _fileCountMapping,
        Group storage _fileGroup,
        IceSort.SortOrder storage _fileGroupOrder,
        
        mapping (uint => mapping(uint => IceGlobal.GlobalRecord)) storage _totalSharesMapping,
        mapping (uint => mapping(uint => IceSort.SortOrder)) storage _totalShareOrderMapping, 
        mapping (uint => uint) storage _shareCountMapping
    )
    public {
        _fileIndex.condValidItem(_fileCountMapping[_ein]);
        _globalItemIndividual.condUnstampedItem();
        _fileGroupOrder.condValidSortOrder(self[_fileIndex].associatedGroupFileIndex);
        condItemMarkedForTransfer(self[_fileIndex]);
        
        _deleteFileMappings(
            _ein, 
            _globalItemIndividual, 
            _totalSharesMapping, 
            _totalShareOrderMapping, 
            _shareCountMapping
        );
        
        delete (self[_fileIndex]);
        
        _deleteFileObject(
            self, 
            _ein, 
            _fileIndex, 
            _fileOrderMapping, 
            _fileCountMapping, 
            _fileGroup
        );
    }
    ```

### File To Group Function
1. addFileToGroup(): this is a function used to add file to a group by using the index of the file to add it and then map group index to the group order. It takes in three parameters; self, which is a pointer to the group Struct, the group index and the file index. Then it returns associatedGroupIndex and associatedGroupFileIndex.
    ```solidity
    function addFileToGroup(Group storage self, uint _groupIndex, uint _fileIndex
    ) public returns (
        uint associatedGroupIndex, uint associatedGroupFileIndex
    ) {
        uint currentIndex = self.groupFilesCount;
        self.groupFilesCount = self.groupFilesOrder.addToSortOrder(self.groupFilesOrder[0].prev, currentIndex, _fileIndex);

        associatedGroupIndex = _groupIndex;
        associatedGroupFileIndex = self.groupFilesCount;
    }
    ```
2. removeFileFromGroup(): is a function used to remove file from a group by using the groupFileOrderIndex of the file to remove it. It takes in two parameters; self, which is a pointer to the group Struct, and the groupFileOrderIndex.
    ```solidity
    function removeFileFromGroup(Group storage self, uint _groupFileOrderIndex) public {
        uint maxIndex = self.groupFilesCount;
        uint pointerID = self.groupFilesOrder[maxIndex].pointerID;

        self.groupFilesCount = self.groupFilesOrder.stichSortOrder(_groupFileOrderIndex, maxIndex, pointerID);
    }
    ```
3. remapFileToGroup(); this function is used to remap file from one group to another. It takes in five parameters and return the groupFileIndex which is a uint.
    ```solidity
    function remapFileToGroup(
        File storage self, uint _existingFileIndex, Group storage _oldGroup,
        Group storage _newGroup, uint _newGroupIndex
    ) public returns (uint groupFileIndex) {
        removeFileFromGroup(_oldGroup, self.associatedGroupFileIndex);

        (self.associatedGroupIndex, self.associatedGroupFileIndex) = addFileToGroup(_newGroup, _newGroupIndex, _existingFileIndex);
        
        groupFileIndex = self.associatedGroupFileIndex;
    }
    ```
4. condItemMarkedForTransfer(): is a function that is used to check if a file has been marked for transfer. The visibility of the function is public and it is a view function. It checks if the group file exist or not.
    ```solidity
    function condItemMarkedForTransfer(File storage self) public view {
        require (
            (self.fileMeta.markedForTransfer == false),
            "File already marked for Transfer"
        );
    }
    ```
5. condMarkedForTransferee(); to check if the user you want to transfer to is eligible to receive the file.
    ```solidity
    function condMarkedForTransferee(File storage self, uint _transfereeEIN) public view {
        require (
            (self.transferEIN == _transfereeEIN),
            "File not marked for Transfers"
        );
    }
    ```
6. condNonReservedItem(): to check if the file that you want to transfer is not a modified item. The visibility of the function is public, and it is a view function. It takes in the a parameter; index. If the index is zero, then it is a reserved item, but can be transferred if it’s a number greater than zero.
    ```solidity
    function condNonReservedItem(uint _index) public pure {
        require (
            (_index > 0),
            "Reserved Item"
        );
    }
    ```

### Group Functions
1. getGroup(): this function returns information about a particular group of files. The visibility of the function is *external*  and it is a view function. It returns index and name of the group. It takes in three parameters; self, which is a pointer to the group Struct, the group index and the group count (count of the number of groups for that specific user). The function checks if it is a valid item.
    ```solidity
    function getGroup(
        Group storage self, uint _groupIndex, uint _groupCount
    )  external view returns (
        uint index, string memory name
    ) {
        _groupIndex.condValidItem(_groupCount);
    
        index = _groupIndex;
    
        if (_groupIndex == 0) {
            name = "Root";
        } else {
            name = self.name;
        }
    }
    ```
2. createGroup(): this function is used to create a new group for the user. It takes in eight parameters and returns newGlobalIndex1,  newGlobalIndex2 and nextGroupIndex. The visibility of the function is *external*.
    ```solidity
    function createGroup(
        mapping (uint => Group) storage self, uint _ein, string calldata _groupName,
        mapping (uint => IceSort.SortOrder) storage _groupOrderMapping, 
        mapping (uint => uint) storage _groupCountMapping,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems,
        uint _globalIndex1, uint _globalIndex2
    )
    external returns (
        uint newGlobalIndex1,
        uint newGlobalIndex2,
        uint nextGroupIndex
    ) {
        (newGlobalIndex1, newGlobalIndex2, nextGroupIndex) = _createGroupInner(
           self, 
           _ein, 
           _groupName, 
           _groupOrderMapping, 
           _groupCountMapping, 
           _globalItems, 
           _globalIndex1, 
           _globalIndex2
        );
    }
    ```

3. _createGroupInner(): this is an internal function used by createGroup() function to facilitate in creating new Group for the user, it takes in eight parameters and returns newGlobalIndex1,  newGlobalIndex2 and nextGroupIndex. 
    ```solidity
    function _createGroupInner(
        mapping (uint => Group) storage groups,
        uint _ein, string memory _groupName,
        mapping (uint => IceSort.SortOrder) storage _groupOrderMapping, 
        mapping (uint => uint) storage _groupCountMapping,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems, uint _globalIndex1, uint _globalIndex2
    ) internal returns (
        uint newGlobalIndex1, uint newGlobalIndex2, uint nextGroupIndex
    ) {
        (newGlobalIndex1, newGlobalIndex2) = IceGlobal.reserveGlobalItemSlot(_globalIndex1, _globalIndex2);

        nextGroupIndex = _groupCountMapping[_ein].add(1);
        
        _globalItems.addItemToGlobalItems(newGlobalIndex1, newGlobalIndex2, _ein, nextGroupIndex, false, false, 0);
        
        groups[nextGroupIndex] = IceFMS.Group(
            IceGlobal.GlobalRecord(newGlobalIndex1, newGlobalIndex2),
            _groupName,
            0 
        );

        _groupCountMapping[_ein] = _groupOrderMapping.addToSortOrder(_groupOrderMapping[0].prev, _groupCountMapping[_ein], 0);

    }
    ```
4. renameGroup(): this is a function that is to rename an existing group for the user/ein. It takes in four parameters; self, a pointer to the Group Struct, _groupIndex, _groupCount, _groupName. The visibility of the function is *external*.
    ```solidity
    function renameGroup(Group storage self, uint _groupIndex, uint _groupCount, string calldata _groupName) external {
        condNonReservedGroup(_groupIndex);
        _groupIndex.condValidItem(_groupCount);

        self.name = _groupName;
    }
    ```

5. deleteGroup(): this function is used to delete an existing group for the user/ein. It takes in nine parameters. The visibility of the function is *external*. It returns currentGroupIndex.
    ```solidity
    function deleteGroup(
        mapping (uint => Group) storage self, uint _ein, uint _groupIndex,
        mapping (uint => IceSort.SortOrder) storage _groupOrderMapping, 
        mapping (uint => uint) storage _groupCountMapping,
        mapping (uint => mapping(uint => IceGlobal.GlobalRecord)) storage _totalSharesMapping,
        mapping (uint => mapping(uint => IceSort.SortOrder)) storage _totalShareOrderMapping, 
        mapping (uint => uint) storage _shareCountMapping,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems
    ) external returns (uint currentGroupIndex) {
        condGroupEmpty(self[_groupIndex]);
        condNonReservedGroup(_groupIndex);
        _groupIndex.condValidItem(_groupCountMapping[_ein]);
        
        currentGroupIndex = _groupCountMapping[_ein];

        IceGlobal.Association storage globalItem = self[_groupIndex].rec.getGlobalItemViaRecord(_globalItems);
        _totalSharesMapping.removeAllShares(
            _ein,
            globalItem, 
            _totalShareOrderMapping, 
            _shareCountMapping
        );
        
        globalItem.deleteGlobalRecord();

        self[_groupIndex] = self[currentGroupIndex];
        _groupCountMapping[_ein] = _groupOrderMapping.stichSortOrder(_groupIndex, currentGroupIndex, 0);

        delete (self[currentGroupIndex]);
    }
    ```
6. condGroupEmpty(): it is a function that check that Group (the struct) Order is valied. The visibility of the function is *public*. It takes in a parameters self, a pointer to the Group Struct.
    ```solidity
    function condGroupEmpty(Group storage self) public view {
        require (
            (self.groupFilesCount == 0),
            "Group has Files"
        );
    }
    ```
7. condNonReservedGroup(): the function takes in a parameter; _index. If the _index is zero, then it is a reserved item, but can be transferred if it’s a number greater than zero. The visibility of the function is *internal*, and it is a pure function.
    ```solidity
    function condNonReservedGroup(uint _index) internal pure {
        require ((_index > 0), "Reserved Item");
    }
    ```

### Transfer Function
1. doInitiateFileTransferChecks(): this is a function to check file transfer conditions before initiating a file transfer. It takes in nine parameters. The visibility of the function is *external*, and it is a view function.
    ```solidity
    function doInitiateFileTransferChecks(
        mapping (uint => mapping(uint => File)) storage self,
        uint _transfererEIN, uint _transfereeEIN, uint _fileIndex, 
        mapping (uint => uint) storage _fileCountMapping,
        mapping (uint => mapping(uint => Group)) storage _totalGroupsMapping,
        mapping (uint => mapping(uint => bool)) storage _blacklist,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems,
        IdentityRegistryInterface _identityRegistry
    ) external view {
        IceGlobal.condEINExists(_transfereeEIN, _identityRegistry);
        IceGlobal.condUniqueEIN(_transfererEIN, _transfereeEIN);
        _fileIndex.condValidItem(_fileCountMapping[_transfererEIN]);
        self[_transfererEIN][_fileIndex].rec.getGlobalItemViaRecord(_globalItems).condUnstampedItem();
        condItemMarkedForTransfer(self[_transfererEIN][_fileIndex]);
        _totalGroupsMapping[_transfererEIN][self[_transfererEIN][_fileIndex].associatedGroupIndex].rec.getGlobalItemViaRecord(_globalItems).condUnstampedItem();
        _blacklist[_transfereeEIN].condNotInList(_transfererEIN);
    }
    ```
2. doFileTransferPart1(): this function is used to do file transfer(Part 1) from previous (current) owner to new owner. It takes in six parameters. The visibility of the function is *external*. It returns nextTransfereeIndex.
    ```solidity
    function doFileTransferPart1 (
        mapping (uint => mapping(uint => File)) storage self,
        uint _transfererEIN, uint _transfereeEIN, uint _fileIndex, 
        mapping (uint => mapping(uint => IceSort.SortOrder)) storage _totalFilesOrderMapping, 
        mapping (uint => uint) storage _fileCountMapping
    ) external returns (uint nextTransfereeIndex) {
        IceGlobal.condCheckUnderflow(_fileCountMapping[_transfererEIN]);
        
        uint currentTransfereeIndex = _fileCountMapping[_transfereeEIN];
        nextTransfereeIndex =  currentTransfereeIndex.add(1);

        self[_transfereeEIN][nextTransfereeIndex] = self[_transfererEIN][_fileIndex];

        uint8 tc = self[_transfereeEIN][nextTransfereeIndex].fileMeta.transferCount.add(1);

        self[_transfereeEIN][nextTransfereeIndex].transferHistory[tc] = _transfereeEIN;
        self[_transfereeEIN][nextTransfereeIndex].fileMeta.markedForTransfer = false;
        self[_transfereeEIN][nextTransfereeIndex].fileMeta.transferCount = tc;

        _fileCountMapping[_transfereeEIN] = _totalFilesOrderMapping[_transfereeEIN].addToSortOrder(
            _totalFilesOrderMapping[_transfereeEIN][0].prev, 
            currentTransfereeIndex, 
            0
        );
    }
    ```

3. doFileTransferPart2(): this function is used to do file transfer(Part 2) from previous (current) owner to new owner. It takes in eight parameters. The visibility of the function is *external*.
    ```solidity
    function doFileTransferPart2 (
        mapping (uint => mapping(uint => File)) storage self,
        uint _transfereeEIN, uint _fileIndex, uint _toRecipientGroup, uint _recipientGroupCount, uint _nextTransfereeIndex,
        mapping (uint => mapping(uint => Group)) storage _totalGroupsMapping,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems
    ) external {
        _toRecipientGroup.condValidItem(_recipientGroupCount); 
        (self[_transfereeEIN][_nextTransfereeIndex].associatedGroupIndex, self[_transfereeEIN][_nextTransfereeIndex].associatedGroupFileIndex) = addFileToGroup(
            _totalGroupsMapping[_transfereeEIN][_toRecipientGroup], 
            _toRecipientGroup, 
            _nextTransfereeIndex
        );
        
        IceGlobal.Association storage globalItem = self[_transfereeEIN][_fileIndex].rec.getGlobalItemViaRecord(_globalItems);

        globalItem.ownerInfo.EIN = _transfereeEIN;
        globalItem.ownerInfo.index = _nextTransfereeIndex;
    }
    ```
4. doPermissionedFileTransfer(): this function is used to initiate requested file transfer in a permissioned manner. It takes in six parameters. The visibility of the function is *external*.
    ```solidity
    function doPermissionedFileTransfer(
        File storage self, uint _transfereeEIN,
        mapping (uint => IceGlobal.GlobalRecord) storage _transfers,
        mapping (uint => IceSort.SortOrder) storage _transferOrderMapping, 
        mapping (uint => uint) storage _transferCountMapping,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems
    )
    external {
        _initiatePermissionedFileTransfer (
            self,
            _transfereeEIN,
            _transfers,
            _transferOrderMapping,
            _transferCountMapping,
            _globalItems
        );
    }
    ```
5. _initiatePermissionedFileTransfer(): this function is used to initiate requested file transfer. It takes in six parameters. The visibility of the function is *internal*.
    ```solidity
    function _initiatePermissionedFileTransfer(
        File storage self, uint _transfereeEIN,
        mapping (uint => IceGlobal.GlobalRecord) storage _transfers,
        mapping (uint => IceSort.SortOrder) storage _transferOrderMapping, 
        mapping (uint => uint) storage _transferCountMapping,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems
    ) internal {
        self.rec.getGlobalItemViaRecord(_globalItems).condUnstampedItem();
        IceGlobal.condCheckOverflow(_transferCountMapping[_transfereeEIN]);
        
        uint nextTransferIndex = _transferCountMapping[_transfereeEIN] + 1;

        self.fileMeta.markedForTransfer = true;
        self.transferEIN = _transfereeEIN;
        self.transferIndex = nextTransferIndex;

        _transfers[nextTransferIndex] = self.rec;

        _transferCountMapping[_transfereeEIN] = _transferOrderMapping.addToSortOrder(_transferOrderMapping[0].prev, _transferCountMapping[_transfereeEIN], 0);
    }
    ```
6. acceptFileTransferPart1():  this function is used to accept file transfer (part 1) from a user. The visibility of the function is *external*.
    ```solidity
    function acceptFileTransferPart1(
        mapping (uint => mapping(uint => File)) storage self,
        uint _transfererEIN, uint _transfereeEIN, uint _fileIndex
    ) external {
        condMarkedForTransferee(self[_transfererEIN][_fileIndex], _transfereeEIN);
        
        self[_transfererEIN][_fileIndex].fileMeta.markedForTransfer = false;
    }
    ```

7. acceptFileTransferPart2(): this function is used to accept file transfer (part 2) from a user. The visibility of the function is *external*.
    ```solidity
    function acceptFileTransferPart2(
        mapping (uint => mapping(uint => File)) storage self,
        uint _transfereeEIN, uint _transferSpecificIndex, 
        mapping (uint => IceGlobal.GlobalRecord) storage _transfersMapping,
        mapping (uint => IceSort.SortOrder) storage _transferOrderMapping, 
        mapping (uint => uint) storage _transferCountMapping,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems
    ) external {
        removeFileFromTransfereeMapping(
            self,
            _transfereeEIN,
            _transferSpecificIndex,
            _transfersMapping,
            _transferOrderMapping,
            _transferCountMapping,
            _globalItems
        );
    }
    ```
8. cancelFileTransfer(): this function is used to cancel file transfer inititated by the current owner and / or recipient. It takes in eight parameters. The visibility of the function is *external*.
    ```solidity
    function cancelFileTransfer(
        mapping (uint => mapping(uint => File)) storage self,
        uint _transfererEIN, uint _transfereeEIN,  uint _fileIndex,
        mapping (uint => IceGlobal.GlobalRecord) storage _transfersMapping,
        mapping (uint => IceSort.SortOrder) storage _transferOrderMapping, 
        mapping (uint => uint) storage _transferCountMapping,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems
    ) external {
        condMarkedForTransferee(self[_transfererEIN][_fileIndex], _transfereeEIN);
        
        _cancelFileTransfer(
            self,
            _transfererEIN,
            _transfereeEIN,
            _fileIndex,
            _transfersMapping,
            _transferOrderMapping, 
            _transferCountMapping,
            _globalItems
        );
    }
    ```
9. _cancelFileTransfer(): this function to cancel file transfer inititated by the current owner or the recipient. It takes in eight parameters. The visibility of this function is *private*. This function is being called in the **cancelFileTransfer()** function.
    ```solidity
    function _cancelFileTransfer (
        mapping (uint => mapping(uint => File)) storage self,
        uint _transfererEIN, uint _transfereeEIN,  uint _fileIndex,
        mapping (uint => IceGlobal.GlobalRecord) storage _transfersMapping,
        mapping (uint => IceSort.SortOrder) storage _transferOrderMapping, 
        mapping (uint => uint) storage _transferCountMapping,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems
    ) private {
        self[_transfererEIN][_fileIndex].fileMeta.markedForTransfer = false;

        uint transferSpecificIndex = self[_transfererEIN][_fileIndex].transferIndex;
        removeFileFromTransfereeMapping(
            self,
            _transfereeEIN,
            transferSpecificIndex,
            _transfersMapping,
            _transferOrderMapping,
            _transferCountMapping,
            _globalItems
        );
    }
    ```
10. removeFileFromTransfereeMapping(): this function is used to remove file from Transfers mapping of Transferee after file is transferred to them. It takes in eight parameters. It is used in acceptFileTransferPart2() and _cancelFileTransfer() functions. The visibility of the function is *private*.
    ```solidity
    function removeFileFromTransfereeMapping(
        mapping (uint => mapping(uint => File)) storage self,
        uint _transfereeEIN, uint _transferSpecificIndex,
        mapping (uint => IceGlobal.GlobalRecord) storage _transfersMapping,
        mapping (uint => IceSort.SortOrder) storage _transferOrderMapping, 
        mapping (uint => uint) storage _transferCountMapping,
        mapping (uint => mapping(uint => IceGlobal.Association)) storage _globalItems
    ) internal {
        _transfersMapping[_transferSpecificIndex].getGlobalItemViaRecord(_globalItems).condItemIsFile();
        
        IceGlobal.Association memory item = _transfersMapping[_transferSpecificIndex].getGlobalItemViaRecord(_globalItems);

        uint currentTransferIndex = _transferCountMapping[_transfereeEIN];

        require (
            (currentTransferIndex > 0),
            "Index Not Found"
        );

        _transfersMapping[_transferSpecificIndex] = _transfersMapping[currentTransferIndex];
        _transferCountMapping[_transfereeEIN] = _transferOrderMapping.stichSortOrder(_transferSpecificIndex, currentTransferIndex, 0);
 
        self[item.ownerInfo.EIN][item.ownerInfo.index].transferIndex = _transferSpecificIndex;
    }
    ```

### String / Bytes Conversion
1. stringToBytes32(): this is a helper function to convert string to bytes32 format. The visibility of the function is public and it is a pure function. It returns result which is a bytes32. It takes in a parameter; string.
    ```solidity
    function stringToBytes32(string memory _source) public pure returns (bytes32 result) {
        bytes memory tempEmptyStringTest = bytes(_source);
        string memory tempSource = _source;
        
        if (tempEmptyStringTest.length == 0) {
            return 0x0;
        }
    
        assembly {
            result := mload(add(tempSource, 32))
        }
    }
    ```
2. bytes32ToString(): this is a helper function to convert bytes32 to string format. It takes in a parameter _x which is a bytes32 and returns result which is a string.
    ```solidity
    function bytes32ToString(bytes32 _x) public pure returns (string memory result) {
        bytes memory bytesString = new bytes(32);
        uint charCount = 0;
        for (uint j = 0; j < 32; j++) {
            byte char = byte(bytes32(uint(_x) * 2 ** (8 * j)));
            if (char != 0) {
                bytesString[charCount] = char;
                charCount++;
            }
        }
        bytes memory bytesStringTrimmed = new bytes(charCount);
        for (uint j = 0; j < charCount; j++) {
            bytesStringTrimmed[j] = bytesString[j];
        }
        
        result = string(bytesStringTrimmed);
    }
    ```