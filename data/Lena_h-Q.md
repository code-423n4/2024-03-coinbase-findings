## [L01] Undeclared Variable in `FCL::ecAff_IsZero` 

## Details
The Natspec for the function ``FCL::ecAff_IsZero`` mentions the variable `x` of type uint256, this is undeclared in the function. 


```javascript
@>  function ecAff_IsZero(uint256, uint256 y) internal pure returns (bool flag) {
        return (y == 0);
    }
```

## Impact
This will likely cause ``FCL::ecAff_add`` to malfunction as it calls the function ``FCL::ecAff_IsZero``.

``
function ecAff_add(uint256 x0, uint256 y0, uint256 x1, uint256 y1) internal view returns (uint256, uint256) {
        uint256 zz0;
        uint256 zzz0;

        if (ecAff_IsZero(x0, y0)) return (x1, y1);
        if (ecAff_IsZero(x1, y1)) return (x0, y0);
        if ((x0 == x1) && (y0 == y1)) {
            (x0, y0, zz0, zzz0) = ecZZ_Dbl(x0, y0, 1, 1);
        } else {
            (x0, y0, zz0, zzz0) = ecZZ_AddN(x0, y0, 1, 1, x1, y1);
        }

        return ecZZ_SetAff(x0, y0, zz0, zzz0);
    }

``


## Tools Used 

Manual Review

## Recommended Mitigation
``FCL::ecAff_IsZero`` should be corrected to look include the variable `x`

```
+ function ecAff_IsZero(uint256 x, uint256 y) internal pure returns (bool flag) {
        return (y == 0);
    }
```