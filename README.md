#!/bin/bash

# Exit on error
set -e

# Output each command
set -x

# Update and install necessary dependencies with legacy peer dependencies
npm install --legacy-peer-deps

# Install Hardhat and related plugins with legacy peer dependencies
npm install --legacy-peer-deps @nomiclabs/hardhat-ethers @nomiclabs/hardhat-waffle @openzeppelin/hardhat-upgrades ethers

# Initialize a new Hardhat project
npx hardhat init

# Create basic Hardhat config
cat <<EOL > hardhat.config.js
require('@nomiclabs/hardhat-waffle');
require('@openzeppelin/hardhat-upgrades');
require('@nomiclabs/hardhat-ethers');

module.exports = {
  solidity: '0.8.4',
  networks: {
    hardhat: {},
    swisstronik: {
      url: 'https://json-rpc.testnet.swisstronik.com/',
      accounts: ['d7b360f03559641212880574eb8edac7fbd0f4d828ad72a5c098319c8ad8478b']
    }
  }
};
EOL

# Create a sample contract
mkdir -p contracts
cat <<EOL > contracts/Sample.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

contract Sample {
    string public message;

    constructor(string memory _message) {
        message = _message;
    }

    function updateMessage(string memory _message) public {
        message = _message;
    }
}
EOL

# Create deployment script
mkdir -p scripts
cat <<EOL > scripts/deploy.js
async function main() {
    const [deployer] = await ethers.getSigners();
    console.log('Deploying contracts with the account:', deployer.address);

    const Sample = await ethers.getContractFactory('Sample');
    const sample = await Sample.deploy('Hello, Swisstronik!');
    console.log('Sample contract deployed to:', sample.address);
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
EOL

echo "Project setup complete. You can now deploy your contract using 'npx hardhat run scripts/deploy.js --network swisstronik'."
