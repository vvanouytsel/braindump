`nftables` is the replacement for the deprecated `iptables`.

## Quick Overview

Nftables uses the following objects:
* A `table` contains a set of `chains`
	*  `nft add table <table_address_family> <table_name>`
* A `chain` contains a set of `rules` and has a default policy
	* `nft add chain ip my-table my-chain { type filter hook input priority -150 \; policy drop \; }`
* A `rule` determines if a package is allowed or blocked
	* Add rule at the end -> `nft add rule <table_address_family> <table_name> <chain_name> <rule>`
	* Add rule in the beginning -> `nft add insert <table_address_family> <table_name> <chain_name> <rule>`

You can list all the rules via the `nft list ruleset` command.

Reference: <https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_firewalls_and_packet_filters/getting-started-with-nftables_firewall-packet-filters#con_basics-of-nftables-tables_assembly_creating-and-managing-nftables-tables-chains-and-rules>

| Type     | Address families    | Hooks                                          | Description                                                                                                                                   |
| :------- | :------------------ | :--------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| `filter` | all                 | all                                            | Standard chain type                                                                                                                           |
| `nat`    | `ip`, `ip6`, `inet` | `prerouting`, `input`, `output`, `postrouting` | Chains of this type perform native address translation based on connection tracking entries. Only the first packet traverses this chain type. |
| `route`  | `ip`, `ip6`         | `output`                                       | Accepted packets that traverse this chain type cause a new route lookup if relevant parts of the IP header have changed.                      |

```bash
nft add rule inet nftables_svc INPUT tcp dport 22 accept
nft add rule inet nftables_svc INPUT tcp dport 443 accept
nft add rule inet nftables_svc INPUT reject with icmpx type port-unreachable
```