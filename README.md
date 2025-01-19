{
  "name": "@solvprotocol/contracts-v3-sft-wrapped-token",
  "version": "1.0.0",
  "devDependencies": {
    "@ethersproject/abi": "^5.7.0",
    "@ethersproject/providers": "^5.7.0",
    "@nomicfoundation/hardhat-chai-matchers": "^1.0.3",
    "@nomicfoundation/hardhat-foundry": "^1.1.1",
    "@nomicfoundation/hardhat-network-helpers": "^1.0.4",
    "@nomicfoundation/hardhat-toolbox": "^1.0.2",
    "@nomiclabs/hardhat-ethers": "^2.1.1",
    "@nomiclabs/hardhat-etherscan": "^3.1.0",
    "@openzeppelin/hardhat-upgrades": "^3.0.2",
    "@typechain/ethers-v5": "^10.1.0",
    "@typechain/hardhat": "^6.1.2",
    "@types/chai": "^4.3.3",
    "@types/mocha": "^9.1.1",
    "@types/node": "^18.7.13",
    "chai": "^4.3.6",
    "dotenv": "^10.0.0",
    "ethers": "^5.7.0",
    "hardhat": "^2.10.2",
    "hardhat-deploy": "^0.11.15",
    "hardhat-deploy-ethers": "^0.3.0-beta.13",
    "hardhat-gas-reporter": "^1.0.8",
    "hardhat-tracer": "^1.1.0-rc.6",
    "solidity-coverage": "^0.7.21",
    "ts-node": "^10.9.1",
    "typechain": "^8.1.0",
    "typescript": "^4.8.2"
  },
  "dependencies": {
    "@openzeppelin/contracts": "5.0.1",
    "@openzeppelin/contracts-upgradeable": "5.0.1"
  }
}
const transparentUpgrade = require('./utils/transparentUpgrade');

module.exports = async ({ getNamedAccounts, deployments, network }) => {

  const { deployer } = await getNamedAccounts();
  const owner = deployer;

  const market = {
    bsctest: '0x929b1B405714ef93CdFFFd6492009baff351f788',
  };

  // target token, currency, poolId
  const poolIds = {
    bsctest: [
      [
        '0x1cF0e51005971c5B78b4A8feE419832CFCCD8cf9', // target token - SolvBTC
        '0xbFEfd7c0BB235E67E314ae65bd9C4685dBE9A45E', // currency - BTCB
        '0x9f4baed29e08317798d7d50e376733dac70eb6819ab6a843029c138f06cea479', // pool ID
      ],
      [
        '0xB4618618b6Fcb61b72feD991AdcC344f43EE57Ad', // target token - SolvBTC.BBN
        '0x1cF0e51005971c5B78b4A8feE419832CFCCD8cf9', // currency - SolvBTC
        '0xaf3b2b789b70339ac56f23c2c8bfd0edd2b1b496f174aefa46880e011fc86187', // pool ID
      ],
      [
        '0xaDAe5fc8d830f86f53E20c8a39F7E12Ff6d4E87c', // target token - SolvBTC.ENA
        '0x1cF0e51005971c5B78b4A8feE419832CFCCD8cf9', // currency - SolvBTC
        '0xc0aefb6754da0510f31decd714b5b3f349b4bf1875c1ab6ddafedba6f33d3e72', // pool ID
      ],
      [
        '0x89E573571B6786b11643585acbCcF3Cb3ABef81e', // target token - SolvBTC.DEFI
        '0x1cF0e51005971c5B78b4A8feE419832CFCCD8cf9', // currency - SolvBTC
        '0xde178805efb7fbbf779048a5a09fb176f7c28fb87204718fcc9c6f927eea8140', // pool ID
      ]
    ]
  }

  // currency, target token, path
  const pathInfos = {
    bsctest: [
      [
        '0xbFEfd7c0BB235E67E314ae65bd9C4685dBE9A45E', // currency - BTCB
        '0x1cF0e51005971c5B78b4A8feE419832CFCCD8cf9', // target token - SolvBTC
        [] // path
      ],
      [
        '0x1cF0e51005971c5B78b4A8feE419832CFCCD8cf9', // currency - SolvBTC
        '0xB4618618b6Fcb61b72feD991AdcC344f43EE57Ad', // target token - SolvBTC.BBN
        [], // path
      ],
      [
        '0x1cF0e51005971c5B78b4A8feE419832CFCCD8cf9', // currency - SolvBTC
        '0xaDAe5fc8d830f86f53E20c8a39F7E12Ff6d4E87c', // target token - SolvBTC.ENA
        [], // path
      ],
      [
        '0x1cF0e51005971c5B78b4A8feE419832CFCCD8cf9', // currency - SolvBTC
        '0x89E573571B6786b11643585acbCcF3Cb3ABef81e', // target token - SolvBTC.DEFI
        [], // path
      ],
      [
        '0xbFEfd7c0BB235E67E314ae65bd9C4685dBE9A45E', // currency - BTCB
        '0xaDAe5fc8d830f86f53E20c8a39F7E12Ff6d4E87c', // target token - SolvBTC.ENA
        [ '0x1cF0e51005971c5B78b4A8feE419832CFCCD8cf9' ], // path
      ]
    ]
  };

  const contractName = 'SolvBTCRouterV2';
  const firstImplName = contractName + 'Impl';
  const proxyName = contractName + 'Proxy';

  const versions = {}
  const upgrades = versions[network.name]?.map(v => {return firstImplName + '_' + v}) || []

  const { proxy, newImpl, newImplName } = await transparentUpgrade.deployOrUpgrade(
    firstImplName,
    proxyName,
    {
      contract: contractName,
      from: deployer,
      log: true
    },
    {
      initializer: { 
        method: "initialize", 
        args: [ owner ] 
      },
      upgrades: upgrades
    }
  );

  const SolvBTCRouterV2Factory = await ethers.getContractFactory("SolvBTCRouterV2", deployer);
  const solvBTCRouterV2 = SolvBTCRouterV2Factory.attach(proxy.address);
  
  const setMarketTx = await solvBTCRouterV2.setOpenFundMarket(market[network.name]);
  console.log(`Set OpenFundMarket ${market[network.name]} at tx: ${setMarketTx.hash}`);
  await setMarketTx.wait(1);

  for (let poolId of poolIds[network.name]) {
    const setPoolIdTx = await solvBTCRouterV2.setPoolId(poolId[0], poolId[1], poolId[2]);
    console.log(`Set PoolInfo at tx: ${setPoolIdTx.hash}`);
    await setPoolIdTx.wait(1);
  }

  for (let pathInfo of pathInfos[network.name]) {
    const setPathTx = await solvBTCRouterV2.setPath(pathInfo[0], pathInfo[1], pathInfo[2]);
    console.log(`Set Path at tx: ${setPathTx.hash}`);
    await setPathTx.wait(1);
  }

};

module.exports.tags = ['SolvBTCRouterV2']
