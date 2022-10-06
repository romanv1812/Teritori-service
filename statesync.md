
# START WITH STATE SYNC
```bash
sudo systemctl stop teritorid && \
teritorid tendermint unsafe-reset-all --home $HOME/.teritorid

RPC=78.46.16.236:44657

LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

peer="3669cd79a690f3e00e6cb87429b9d7b674955d8c@78.46.16.236:44656"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peer\"/" $HOME/.teritorid/config/config.toml

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.teritorid/config/config.toml

sudo systemctl restart teritorid && journalctl -u teritorid -f -o cat
```
