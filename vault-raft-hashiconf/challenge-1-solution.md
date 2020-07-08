There is more than one approach to solve the challenge, and this ***sample*** solution.

Create the `node4` configuration file, "`config-node4.hcl`".

```
storage "raft" {
 path    = "/home/scrapbook/tutorial/raft-node4/"
 node_id = "node4"

 retry_join {
   leader_api_addr = "http://127.0.0.1:8200"
 }

 retry_join {
   leader_api_addr = "http://127.0.0.1:2200"
 }
}

listener "tcp" {
 address = "127.0.0.1:4200"
 cluster_address = "127.0.0.1:4201"
 tls_disable = true
}

disable_mlock = true
api_addr = "http://127.0.0.1:4200"
cluster_addr = "http://127.0.0.1:4201"
```

<br />

Click the **+** next to the opened Terminal, and select **Open New Terminal** to start sixth terminal (**Terminal 7**). Execute the following command to start `node4`:

```
mkdir raft-node4
vault server -config=config-node4.hcl -log-level=debug
```{{execute T7}}

**Open New Terminal** to start a new terminal (**Terminal 8**), and set the VAULT_ADDR to `http://127.0.0.1:4200`.

```
export VAULT_ADDR='http://127.0.0.1:4200'
```{{execute T8}}

Unseal `node4` using the unseal key generated by the **active** server (`node1`).

```
vault operator unseal $(grep 'Key 1:' key.txt | awk '{print $NF}')
```{{execute T8}}

Verify the vault status.

```
vault status
```{{execute T8}}

```
Key                     Value
---                     -----
Seal Type               shamir
Initialized             true
Sealed                  false
Total Shares            1
Threshold               1
Version                 1.4.2
Cluster Name            vault-cluster-4d6b6436
Cluster ID              0cab6b03-4bd6-b8bf-3998-04a2d2d3b5f1
HA Enabled              true
HA Cluster              https://127.0.0.1:2201
HA Mode                 standby
Active Node Address     http://127.0.0.1:2200
Raft Committed Index    76
Raft Applied Index      76
```

HA is enabled and active node is set to `http://127.0.0.1:2200` (`node2`).

Once Vault is unsealed, you can login using the root token.

```
vault login $(grep 'Initial Root Token:' key.txt | awk '{print $NF}')
```{{execute T8}}

Check the Raft cluster configuration.

```
vault operator raft list-peers
```{{execute T8}}

You should find `node4` in the list.

```
Node     Address           State       Voter
----     -------           -----       -----
node1    127.0.0.1:8201    follower    true
node2    127.0.0.1:2201    leader      true
node4    127.0.0.1:4201    follower    true
```

You should be able to read the secrets you created earlier.

```
vault kv get secret/credentials
```{{execute T8}}

```
====== Metadata ======
Key              Value
---              -----
created_time     2020-06-08T23:19:23.275510367Z
deletion_time    n/a
destroyed        false
version          1

====== Data ======
Key         Value
---         -----
passcode    vaultrocks
user_id     student
```