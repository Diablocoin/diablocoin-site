# diablocoin-site
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract Diablocoin is ERC20, Ownable {
    uint256 public maxSupply = 100_000_000 * 10 ** decimals();

    address public devWallet = 0x2D64DeFb96958fD2C4266551bA2545A83d45362F;
    uint256 public lockedUntil;
    bool public airdropExecuted;

    constructor() ERC20("Diablocoin", "DBC") {
        _mint(address(this), maxSupply);
        lockedUntil = block.timestamp + 90 days;

        // 1 Mio für Start-Liquidität freigeben
        _transfer(address(this), devWallet, 1_000_000 * 10 ** decimals());
    }

    function releaseLiquidity() external {
        require(block.timestamp >= lockedUntil, "Noch gesperrt");
        _transfer(address(this), devWallet, 2_000_000 * 10 ** decimals());
    }

    function airdrop(address[] calldata recipients, uint256 amountEach) external onlyOwner {
        require(!airdropExecuted, "Airdrop bereits ausgeführt");
        for (uint256 i = 0; i < recipients.length; i++) {
            _transfer(address(this), recipients[i], amountEach * 10 ** decimals());
        }
        airdropExecuted = true;
    }

    function _transfer(address from, address to, uint256 amount) internal override {
        uint256 burnAmount = (amount * 3) / 100;
        uint256 devAmount = (amount * 3) / 100;
        uint256 sendAmount = amount - burnAmount - devAmount;

        super._transfer(from, address(0), burnAmount);
        super._transfer(from, devWallet, devAmount);
        super._transfer(from, to, sendAmount);
    }
}