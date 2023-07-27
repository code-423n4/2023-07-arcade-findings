In BaseVotingVault.sol, the _manager function and the _timelock function can be changed from view functions to pure functions. 

```    
    function _timelock() internal view returns (Storage.Address storage) {
        return Storage.addressPtr("timelock");
    }
```

```    
function _manager() internal view returns (Storage.Address storage) {
        return Storage.addressPtr("manager");
    }
```