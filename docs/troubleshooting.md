# Troubleshooting

- [Generic Troubleshooting](#generic-troubleshooting)
- [Inspect Go Runtime Debug Data](#inspect-go-runtime-debug-data)
- [Specific Errors](#specific-errors)

<br>

---

<br>

## Generic Troubleshooting

### **Ensure `rly` package is properly installed**

   Run:
   ```shell
   $ rly version
   ```

   If this returns an error, make sure you have Go installed and your Go environment is setup. Then redo [Step 1](../README.md#basic-usage---relaying-packets-across-chains).


### **Verify valid `keys`, `balance`, and `path`**

```shell
$ rly chains list
```

Output should show all checkboxes:
```shell
-> type(cosmos) key(✔) bal(✔) path(✔)
```

### **Verify valid `chain`, `client`, and `connection`**

```shell
$ rly paths list
```

If output:
```shell
-> chns(✘) clnts(✘) conn(✘)
```
Verify that you have a healthy RPC address. This means the relayer was unable to query the latest height of one or both the chains.

If:
```shell
-> chns(✔) clnts(✘) conn(✘)
```
Your client is the culprit here. Your client may be invalid or expired.

### **Ensure Client is not `expired`**

```shell
$ rly query clients-expiration <PATH-NAME>
```

<br>

---

<br>

## Inspect Go runtime debug data

If you started `rly` with `--enable-debug-server` argument,
you can open `http://127.0.0.1:5183` in your browser to explore details from the Go runtime.

If you need active assistance from the Relayer development team regarding an unresponsive Relayer instance,
it will be helpful to provide the output from `http://127.0.0.1:7597/debug/pprof/goroutine?debug=2` at a minimum.

<br>

---

<br>

## Specific Errors

<details>
<summary>error querying block data</summary>

<br>
The relayer looks back in time at historical transactions and needs to have an index of them.

Specifically check `~/.<node_data_dir>/config/config.toml` has the following fields set:
```toml
indexer = "kv"
index_all_tags = true
```

</details>


<details>
<summary>error building or broadcasting transaction</summary>

<br>
When preparing a transaction for relaying, the amount of gas that the transaction will consume is unknown.  To compute how much gas the transaction will need, the transaction is prepared with 0 gas and delivered as a `/cosmos.tx.v1beta1.Service/Simulate` query to the RPC endpoint.  Recently chains have been creating AnteHandlers in which 0 gas triggers an error case:

```
lvl=info msg="Error building or broadcasting transaction" provider_type=cosmos chain_id=evmos_9001-2 attempt=1 max_attempts=5 error="rpc error: code = InvalidArgument desc = provided fee < minimum global fee (0aevmos < ). Please increase the gas price.: insufficient fee: invalid request"
```

A workaround is available in which the `min-gas-amount` may be set in the chain's configuration to enable simulation with a non-zero amount of gas.

```
    evmos:
        type: cosmos
        value:
            key: relayer
            chain-id: evmos_9001-2
            rpc-addr: http://127.0.0.1:26657
            account-prefix: evmos
            keyring-backend: test
            gas-adjustment: 1.2
            gas-prices: 20000000000aevmos
            min-gas-amount: 1
            debug: false
            timeout: 20s
            output-format: json
            sign-mode: direct
```

</details>


<details>
<summary>invalid header: new header has a time from the future</summary>

<br>
This is most likely an rpc issue.
The latest block time on the source and destination chain have likely drifted apart.

You can confirm by this by checking the latest block time on each chain:

```shell
grpcurl -plaintext <GRP-URL:PORT> cosmos.base.tendermint.v1beta1.Service.GetLatestBlock | grep '"time":'
```

The solution here is to either use a different RPC endpoint OR if you are in control of the RPC, try restarting the node.

</details>

<br>
<br>

[<-- Create Path Across Chains](create-path-across-chain.md) - [Features -->](./features.md)