# Appendix C: Network Information

## Masternode Information

The fields below are represented as `networkInfo` and are applicable to all masternodes types as defined in
[Appendix B](masternode-types.md)

| Field     | Type    | Size | Description                                                                |
| --------- | ------- | ---- | -------------------------------------------------------------------------- |
| ipAddress | byte[]  | 16   | IPv6 address in network byte order. Only IPv4 mapped addresses are allowed |
| port      | uint_16 | 2    | Port (network byte order)                                                  |

### <a name="mninfo_rules">Validation Rules</a>

Network information is invalid if any of these conditions are true:

  1. `ipAddress` is set and `port` is not set to the default mainnet port
  2. `ipAddress` is set and not routable or not an IPv4 mapped address
  3. `ipAddress` is set and already used in the registered masternodes set

## Platform Information

The fields below are represented as `platformInfo` and are only applicable for EvoNodes (type 1).

| Field            | Type    | Size   | Description                                                                              |
| ---------------- | ------- | ------ | ---------------------------------------------------------------------------------------- |
| platformP2PPort  | uint_16 | 0 or 2 | TCP port of Dash Platform peer-to-peer communication between nodes (network byte order). |
| platformHTTPPort | uint_16 | 0 or 2 | TCP port of Platform HTTP/API interface (network byte order).                            |

### <a name="plinfo_rules">Validation Rules</a>

Platform information is invalid if any of these conditions are true:

  1. `platformP2PPort`, `platformHTTPPort` and [masternode information](#masternode-information) fields are not distinct
  2. `platformP2PPort` and `platformHTTPPort` aren't valid port values [1, 65535]
  3. For mainnet evonodes, `platformP2PPort` and `platformHTTPPort` aren't set to the default values
    1. Default Platform P2P port: 26656
    2. Default Platform HTTP port: 443
