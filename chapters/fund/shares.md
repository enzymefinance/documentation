# shares

## Shares.sol

Description
...

Inherits
Spoke, StandardToken (links)

On construction, the contract becomes a spoke of the hub.

Public State variables

None.

Structs

None.

Enums

None.

Public Functions

`function createFor(address who, uint amount) auth`

function calls internal function mint

`function destroyFor(address who, uint amount) auth`

function calls internal function burn

`function transfer(address to, uint amount) public returns (bool)`

function reverts...

`function transferFrom(address from, address to, uint amount) public returns (bool)`

function reverts...

`function approve(address spender, uint amount) public returns (bool)`

function reverts...

`function increaseApproval(address spender, uint amount) public returns (bool)`

function reverts...

`function decreaseApproval(address spender, uint amount) public returns (bool)`

function reverts...


document factory?