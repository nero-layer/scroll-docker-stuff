``` bash

CHAIN_ID_L2="31337"
SCROLL_L1_RPC="http://localhost:8543"
SCROLL_L2_RPC="http://localhost:8544"
L1_DEPLOYER_PRIVATE_KEY="0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
L2_DEPLOYER_PRIVATE_KEY="0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"
L1_ROLLUP_OPERATOR_ADDR="0x1111111111111111111111111111111111111111"


forge script scripts/foundry/DeployL1BridgeContracts.s.sol:DeployL1BridgeContracts --rpc-url $SCROLL_L1_RPC

forge script scripts/foundry/DeployL1BridgeContracts.s.sol:DeployL1BridgeContracts \
    --force --gas-limit 1000000000 --rpc-url $SCROLL_L1_RPC --broadcast

```

## Hardhat

``` bash
layer1=l1geth
npx hardhat --network $layer1 run ./scripts/deploy_proxy_admin.ts
npx hardhat --network $layer1 run scripts/deploy_scroll_chain.ts
env CONTRACT_NAME=L1ScrollMessenger npx hardhat run --network $layer1 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L1GatewayRouter npx hardhat run --network $layer1 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L1StandardERC20Gateway npx hardhat run --network $layer1 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L1CustomERC20Gateway npx hardhat run --network $layer1 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L1ERC721Gateway npx hardhat run --network $layer1 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L1ERC1155Gateway npx hardhat run --network $layer1 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L1ETHGateway npx hardhat run --network $layer1 scripts/deploy_proxy_contract.ts

# Error cause not WETH deployed, not going to bother
# env CONTRACT_NAME=L1WETHGateway npx hardhat run --network $layer1 scripts/deploy_proxy_contract.ts

```


``` bash
layer2=l2geth
npx hardhat --network $layer2 run scripts/deploy_proxy_admin.ts

# Has an error
# npx hardhat --network $layer2 run scripts/deploy_l2_messenger.ts

npx hardhat --network $layer2 run scripts/deploy_l2_token_factory.ts
env CONTRACT_NAME=L2GatewayRouter npx hardhat run --network $layer2 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L2StandardERC20Gateway npx hardhat run --network $layer2 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L2CustomERC20Gateway npx hardhat run --network $layer2 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L2ERC721Gateway npx hardhat run --network $layer2 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L2ERC1155Gateway npx hardhat run --network $layer2 scripts/deploy_proxy_contract.ts
env CONTRACT_NAME=L2ETHGateway npx hardhat run --network $layer2 scripts/deploy_proxy_contract.ts
# env CONTRACT_NAME=L2WETHGateway npx hardhat run --network $layer2 scripts/deploy_proxy_contract.ts

# Had to come back and install this one
env CONTRACT_NAME=L2ScrollMessenger npx hardhat run --network $layer2 scripts/deploy_l2_messenger.ts

```

``` bash
L2_STANDARD_ERC20_GATEWAY_PROXY_ADDR=$(cat ./deployments/l2geth.json | jq  '.L2StandardERC20Gateway.proxy')
echo $L2_STANDARD_ERC20_GATEWAY_PROXY_ADDR

L2_SCROLL_STANDARD_ERC20_ADDR=$(cat ./deployments/l2geth.json | jq  '.L2StandardERC20Gateway.implementation')
echo $L2_SCROLL_STANDARD_ERC20_ADDR

L2_SCROLL_STANDARD_ERC20_FACTORY_ADDR=$(cat ./deployments/l2geth.json | jq  '.ScrollStandardERC20Factory')
echo $L2_SCROLL_STANDARD_ERC20_FACTORY_ADDR

L2_GATEWAY_ROUTER_PROXY_ADDR=$(cat ./deployments/l2geth.json | jq  '.L2GatewayRouter.proxy')
echo $L2_GATEWAY_ROUTER_PROXY_ADDR

L2_ERC1155_GATEWAY_PROXY_ADDR=$(cat ./deployments/l2geth.json | jq  '.L2ERC1155Gateway.proxy')
echo $L2_ERC1155_GATEWAY_PROXY_ADDR


# initalize contracts in layer 1, should set proper bash env variables first
npx hardhat --network $layer1 run scripts/initialize_l1_erc20_gateway.ts

# Error with this one
npx hardhat --network $layer1 run scripts/initialize_l1_gateway_router.ts

# Script does not exist
# npx hardhat --network $layer1 run scripts/initialize_scroll_chain.ts

# Error zkRollypAddress
npx hardhat --network $layer1 run scripts/initialize_l1_messenger.ts

# Error Missing Address
npx hardhat --network $layer1 run scripts/initialize_l1_custom_erc20_gateway.ts

# Error Missing Address
npx hardhat --network $layer1 run scripts/initialize_l1_erc1155_gateway.ts

# Error Missing Address
npx hardhat --network $layer1 run scripts/initialize_l1_erc721_gateway.ts

```


## L2 Initalize

``` bash
# initalize contracts in layer 2, should set proper bash env variables first
L1_STANDARD_ERC20_GATEWAY_PROXY_ADDR=$(cat ./deployments/l1geth.json | jq  '.L1StandardERC20Gateway.proxy')
echo $L1_STANDARD_ERC20_GATEWAY_PROXY_ADDR

L1_CUSTOM_ERC20_GATEWAY_PROXY_ADDR=$(cat ./deployments/l1geth.json | jq  '.L1CustomERC20Gateway.proxy')
echo $L1_STANDARD_ERC20_GATEWAY_PROXY_ADDR


npx hardhat --network $layer2 run scripts/initialize_l2_erc20_gateway.ts
# npx hardhat --network $layer2 run scripts/initialize_l2_gateway_router.ts
npx hardhat --network $layer2 run scripts/initialize_l2_custom_erc20_gateway.ts
# npx hardhat --network $layer2 run scripts/initialize_l2_erc1155_gateway.ts
# npx hardhat --network $layer2 run scripts/initialize_l2_erc721_gateway.ts
# npx hardhat --network $layer2 run scripts/initialize_l2_token_factory.ts

```