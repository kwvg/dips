# Appendix C: Network Information

| Field     | Type    | Size | Description                                                                |
| --------- | ------- | ---- | -------------------------------------------------------------------------- |
| ipAddress | byte[]  | 16   | IPv6 address in network byte order. Only IPv4 mapped addresses are allowed |
| port      | uint_16 | 2    | Port (network byte order)                                                  |

## <a name="netinfo_rules">Validation Rules</a>

Network information is invalid if any of these conditions are true:

  1. `ipAddress` is set and `port` is not set to the default mainnet port
  2. `ipAddress` is set and not routable or not an IPv4 mapped address
  3. `ipAddress` is set and already used in the registered masternodes set
