# Local Gridiron

Instructions for locally setup, building and running of gridiron rollapp.

## Set up work environment

Install go (version v1.18): https://golang.org/doc/install.

Install ignite: https://docs.ignite.com/guide/install

## Run gridiron settlement

Clone the gridiron settlement repository:

```sh
git clone https://github.com/gridfoundation/gridiron.git --branch v0.1.0-alpha && cd gridiron
```

Build and init the chain:

```sh
export CHAIN_ID="local-testnet"
export KEY_NAME="local-user"
export MONIKER_NAME="local"
export SETTLEMENT_RPC="0.0.0.0:36657"
export GRPC_ADDRESS="0.0.0.0:8090"
export GRPC_WEB_ADDRESS="0.0.0.0:8091"
export P2P_ADDRESS="0.0.0.0:36656"

sh scripts/setup_local.sh
```

Run the chain:

```sh
sh scripts/run_local.sh
```

## Setup and build gridiron rollapp

Build the chain:

```sh
git clone https://github.com/gridfoundation/local-gridiron.git && cd local-gridiron

export WORKSPACE_PATH=$HOME/workspace

cd checkers/build_chain_script && sh build.sh
```

Or build it manually using [these instructions](/checkers/build_chain.md)

Setting up the RDK:

```sh
cd "$WORKSPACE_PATH/checkers"
go mod edit -replace github.com/cosmos/cosmos-sdk=github.com/gridfoundation/rdk@v0.1.2-alpha
go mod tidy && go mod download
ignite chain build
```

Init checkers-rollapp chain:

```sh
KEY_PLAYER_1="player1"
KEY_PLAYER_2="player2"
ROLLAPP_ID="checkers"

checkersd tendermint unsafe-reset-all
checkersd init checkers-test --chain-id "$ROLLAPP_ID"
checkersd keys add "$KEY_PLAYER_1"
checkersd keys add "$KEY_PLAYER_2"
checkersd add-genesis-account "$(checkersd keys show "$KEY_PLAYER_1" -a)" 100000000000stake
checkersd add-genesis-account "$(checkersd keys show "$KEY_PLAYER_2" -a)" 100000000000stake
checkersd gentx "$KEY_PLAYER_1" 100000000stake --chain-id "$ROLLAPP_ID"
checkersd collect-gentxs
```

## Register the rollapp on the gridiron settlement layer

Create rollapp entity in the gridiron settlement

```sh
CHAIN_ID="local-testnet"
KEY_NAME="local-user"

furyd tx rollapp create-rollapp "$ROLLAPP_ID" stamp1 "genesis-path/1" 3 100 '{"Addresses":[]}' \
  --from "$KEY_NAME" \
  --chain-id "$CHAIN_ID" \
  --keyring-backend test
```

Initialize and attach a Sequencer to the rollapp:

```sh
MONIKER_NAME="local"
DESCRIPTION="{\"Moniker\":\"$MONIKER_NAME\",\"Identity\":\"\",\"Website\":\"\",\"SecurityContact\":\"\",\"Details\":\"\"}";
CREATOR_ADDRESS="$(furyd keys show "$KEY_NAME" -a --keyring-backend test)"
CREATOR_PUB_KEY="[10,32,240,168,86,204,92,175,25,200,177,126,91,59,1,62,58,142,225,222,7,40,5,64,53,144,42,48,218,76,112,151,113,14]"

furyd tx sequencer create-sequencer "$CREATOR_ADDRESS" "$CREATOR_PUB_KEY" "$ROLLAPP_ID" "$DESCRIPTION" \
  --from "$KEY_NAME" \
  --chain-id "$CHAIN_ID" \
  --keyring-backend test
```

Make sure the rollap and the sequencer are added:
```sh
furyd q rollapp list-rollapp
furyd q sequencer list-sequencer
```

## Run gridiron rollapp

Run the checkers-rollapp chain:

```sh
SETTLEMENT_RPC="0.0.0.0:36657"
SETTLEMENT_CONFIG="{\"node_address\": \"http:\/\/$SETTLEMENT_RPC\", \"rollapp_id\": \"$ROLLAPP_ID\", \"fury_account_name\": \"$KEY_NAME\", \"keyring_home_dir\": \"$HOME/.gridiron/\", \"keyring_backend\":\"test\"}"
NAMESPACE_ID=000000000000FFFF

checkersd start --furyint.aggregator true \
  --furyint.da_layer mock \
  --furyint.settlement_layer gridiron \
  --furyint.settlement_config "$SETTLEMENT_CONFIG" \
  --furyint.block_batch_size 500 \
  --furyint.namespace_id "$NAMESPACE_ID" \
  --furyint.block_time 0.2s
```

## Interact with rollapp

Interact with the checkers rollapp using the [following examples](/checkers/interaction.md)
