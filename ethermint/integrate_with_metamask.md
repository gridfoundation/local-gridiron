# Connect your MetaMask wallet with Ethermint.

Instructions for adding the gridiron-ethermint network to the metamask and sync an account.

## Init and start node

Init and start gridiron-ethermint node by following [these instructions](./gridiron_ethermint_node.md)

## Adding a New Network

Open the metamask extension on your browser. Then click the top right circle and go to Settings > Networks > Add Network
and fill the form as shown below.

```yaml
Network Name: Gridiron-Ethermint Local

New RPC URL: http://localhost:8545/

Chain ID: 9001

Currency Symbol (optional): PHOTON
```

## Import Account to MetaMask

Click again on the top right circle and select Import Account (make sure that the `Private Key` option is selected).

Export your private key from the terminal using the following command:

```sh
# the key is taken from the 'init.sh' script founds in the ethermint repo.
KEY="mykey"

ethermintd keys unsafe-export-eth-key $KEY --keyring-backend test
```

Then paste the exported private key and click `Import`.

Your account balance should show up as 99990000 PHOTON (or a bit less if you interact with the truffle contract).

