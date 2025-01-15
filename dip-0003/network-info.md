# Appendix C: Network Information

## Legacy Format

### Masternode Information

The fields below are represented as `networkInfo` and are applicable to all masternodes types as defined in
[Appendix B](masternode-types.md)

| Field     | Type    | Size | Description                                                                |
| --------- | ------- | ---- | -------------------------------------------------------------------------- |
| ipAddress | byte[]  | 16   | IPv6 address in network byte order. Only IPv4 mapped addresses are allowed |
| port      | uint_16 | 2    | Port (network byte order)                                                  |

#### <a name="mninfo_rules">Validation Rules</a>

Network information is invalid if any of these conditions are true:

  1. `ipAddress` is set and `port` is not set to the default mainnet port
  2. `ipAddress` is set and not routable or not an IPv4 mapped address
  3. `ipAddress` is set and already used in the registered masternodes set

### Platform Information

The fields below are represented as `platformInfo` and are only applicable for EvoNodes (type 1).

| Field            | Type    | Size   | Description                                                                              |
| ---------------- | ------- | ------ | ---------------------------------------------------------------------------------------- |
| platformP2PPort  | uint_16 | 0 or 2 | TCP port of Dash Platform peer-to-peer communication between nodes (network byte order). |
| platformHTTPPort | uint_16 | 0 or 2 | TCP port of Platform HTTP/API interface (network byte order).                            |

#### <a name="plinfo_rules">Validation Rules</a>

Platform information is invalid if any of these conditions are true:

  1. `platformP2PPort`, `platformHTTPPort` and [masternode information](#masternode-information) fields are not distinct
  2. `platformP2PPort` and `platformHTTPPort` aren't valid port values [1, 65535]
  3. For mainnet evonodes, `platformP2PPort` and `platformHTTPPort` aren't set to the default values
    1. Default Platform P2P port: 26656
    2. Default Platform HTTP port: 443

## Extended Format

The fields below are represented as `networkInfo` and subsume the responsibility of `platformInfo`.
Applicability of fields based on masternode type will be defined by the field individually.

| Field     | Type    | Size     | Description                                                                                    |
| --------- | ------- | -------- | ---------------------------------------------------------------------------------------------- |
| p_count   | uint_8  | 1        | Number of purposes for which addresses are defined                                             |
| p_entries | byte[]  | variable | Array of pairings between purpose codes and arrays of addresses containing network information |

### `p_count` field

* The value of this field MUST correspond to the number of [`p_entries`](#p_entries-field)

### `p_entries` field

This field is an array of [`p_count`](#p_count-field) elements of type [`p_entry`](#p_entry-type) where:

* `p_entries[0]`
    * MUST have [`purpose`](#p_entrypurpose-field) code `CORE_P2P`
    * MUST be registered regardless of masternode type (as defined in [Appendix B](masternode-types.md)).
    * [`entries[0]`](#p_entryentries-field) MUST have a [`type`](#entrytype-field) of `0x01` (IPv4 address).
    * [`entries`](#p_entryentries-field) MUST NOT have a [`type`](#entrytype-field) of `0xD0` (domain name).
* `p_entries[1]`
    * MUST have [`purpose`](#p_entrypurpose-field) code `PLATFORM_P2P`.
    * MUST be registered for EvoNodes (type 1).
    * MUST NOT be registered for regular masternodes (type 0).
    * [`entries[0]`](#p_entryentries-field) MUST have a [`type`](#entrytype-field) of `0x01` (IPv4 address).
    * [`entries`](#p_entryentries-field) MUST NOT have a [`type`](#entrytype-field) of `0xD0` (domain name).
* `p_entries[2]`
    * MUST have [`purpose`](#p_entrypurpose-field) code `PLATFORM_API`.
    * MUST be registered for EvoNodes (type 1).
    * MUST NOT be registered for regular masternodes (type 0).
    * [`entries`](#p_entryentries-field) MAY have a [`type`](#entrytype-field) of `0xD0` (domain name).
      * The element's [`address`](#entryaddress-field) MUST resolve to any [`address`](#entryaddress-field) in
        `p_entries[1]` of [`type`](#entrytype-field) `0x01` (IPv4 address) or `0x02` (IPv6 address)
        * This resolution check MUST be done during PoSe verification and is RECOMMENDED to be done by clients when
          attempting to connect to nodes using the [`address`](#entryaddress-field) supplied.

### `p_entry` type

| Field     | Type    | Size     | Description                                                                        |
| --------- | ------- | -------- | ---------------------------------------------------------------------------------- |
| purpose   | uint_8  | 1        | Network activity associated with address information                               |
| count     | uint_8  | 1        | Number of addresses through which the masternode is accessible                     |
| entries   | byte[]  | variable | Array of length `count` containing network information used to connect to the node |

#### `p_entry.purpose` field

The value of this field MUST be set to one of the values given below

| Purpose Code | Name           | Description                                              |
| ------------ | -------------- | -------------------------------------------------------- |
| `0x01`       | `CORE_P2P`     | A node running Dash Core, exposing P2P functionality     |
| `0x02`       | `PLATFORM_P2P` | A node running Dash Platform, exposing P2P functionality |
| `0x03`       | `PLATFORM_API` | A node running Dash Platform HTTP API endpoints          |

#### `p_entry.count` field

* The value of this field MUST correspond to the number of [`entry`](#entry-type) elements in the
  [`entries`](#entries-field) field.

#### `p_entry.entries` field

This field is an array of [`p_entry.count`](#p_entrycount-field) elements of type [`entry`](#entry-type) where:

* There MUST be at least one element and at most thirty-two elements.
* Elements in `entries` MUST allow duplicate [`type`](#entrytype-field)s but MUST NOT allow duplicate
  [`address`](#entryaddress-field)es even if listed under unique [`type`](#entrytype-field)s unless it is registered
  under a dissimilar [`purpose`](#p_entrypurpose-field) and differentiated by [`port`](#entryport-field).

#### `entry` type

| Field   | Type    | Size     | Description                                                     |
| ------- | ------- | -------- | --------------------------------------------------------------- |
| type    | uint_8  | 1        | Network identifier                                              |
| address | byte[]  | variable | Address of `type` that can be used to connect to the masternode |
| port    | uint_16 | 2        | Port (network byte order)                                       |

##### `entry.type` field

The network identifier field MUST support the following BIP 155 [network IDs](https://github.com/bitcoin/bips/blob/17c04f9fa1ecae173d6864b65717e13dfc1880af/bip-0155.mediawiki#specification):

| Network ID | Address Length (bytes) | Description                             |
| ---------- | ---------------------- | --------------------------------------- |
| `0x01`     | 4                      | IPv4 address (globally routed internet) |
| `0x02`     | 16                     | IPv6 address (globally routed internet) |
| `0x04`     | 32                     | Tor v3 hidden service address           |
| `0x05`     | 32                     | I2P overlay network address             |
| `0x06`     | 16                     | CJDNS overlay network address           |
| `0x07`     | 16                     | Yggdrasil overlay network address       |

The network identifier field MUST NOT support the following BIP 155 network IDs:

| Network ID | Address Length (bytes) | Description                   |
| ---------- | ---------------------- | ----------------------------- |
| `0x03`     | 10                     | Tor v2 hidden service address |

The network identifier field MUST support the following [extensions](#extensions):

| Network ID | Address Length (bytes) | Description                            |
| ---------- | ---------------------- | -------------------------------------- |
| `0xD0`     | variable               | Domain name (globally routed internet) |

* Network IDs not enumerated in the above tables MUST NOT be permitted

##### `entry.address` field

* [`address`](#entryaddress-field) of [`type`](#entrytype-field)s originating from BIP 155 MUST be compliant with
  encoding standards as defined by BIP 155 (e.g.
  [TorV3](https://github.com/bitcoin/bips/blob/17c04f9fa1ecae173d6864b65717e13dfc1880af/bip-0155.mediawiki#appendix-b-tor-v3-address-encoding),
  [I2P](https://github.com/bitcoin/bips/blob/17c04f9fa1ecae173d6864b65717e13dfc1880af/bip-0155.mediawiki#appendix-c-i2p-address-encoding),
  [CJDNS](https://github.com/bitcoin/bips/blob/17c04f9fa1ecae173d6864b65717e13dfc1880af/bip-0155.mediawiki#appendix-d-cjdns-address-encoding),
  [Yggdrasil](https://github.com/bitcoin/bips/blob/17c04f9fa1ecae173d6864b65717e13dfc1880af/bip-0155.mediawiki#appendix-e-yggdrasil-address-encoding)).
* `address` of [`type`](#entrytype-field)s originating from [extensions](#extensions) MUST be compliant with the
  specification as defined (e.g. [Internet domain names](#extension-a-internet-domain-names))

##### `entry.port` field

* This field MUST be any integer between 1024 and 65535 unless the [`purpose`](#p_entrypurpose-field) is `PLATFORM_API`,
  which may use the following ports.

  | Port Number | Description                           |
  | ----------- | ------------------------------------- |
  | 80          | Hypertext Transfer Protocol (HTTP)    |
  | 443         | HTTP over TLS                         |

  * It is RECOMMENDED for nodes to use the default values expected for the chain on which the masternode is operating

  * This field MUST NOT permit a value considered "prohibited" (see below)

    <details>

    <summary>Prohibited ports:</summary>

    | Port Number | Description                           |
    | ----------- | ------------------------------------- |
    | 1719        | H.323 registration                    |
    | 1720        | H.323 call signaling                  |
    | 1723        | Point-to-Point Tunneling Protocol     |
    | 2049        | Network File System                   |
    | 3659        | Apple SASL                            |
    | 4045        | NFS lock daemon                       |
    | 5060        | Session Initiation Protocol (SIP)     |
    | 5061        | SIP over TLS                          |
    | 6000        | X11                                   |
    | 6566        | Scanner Access Now Easy (SANE) daemon |
    | 6660–6664   | Internet Relay Chat (Unofficial)      |
    | 6665-6669   | IRC (Official)                        |
    | 6679        | IRC over TLS (Unofficial)             |
    | 6697        | IRC over TLS (Official)               |
    | 8332        | Bitcoin JSON-RPC server               |
    | 8333        | Bitcoin P2P                           |
    | 10080       | Amanda                                |
    | 18332       | Bitcoin JSON-RPC server (Testnet)     |
    | 18333       | Bitcoin P2P (Testnet)                 |

    </details>

* This field MUST be ignored for connecting to [`address`](#entryaddress-field) of [`type`](#entrytype-field)
  where ports are immaterial.

## Extensions

Extensions are supported types of addresses that are beyond the types enumerated in BIP 155. Their network IDs are defined
in the [`entry.type`](#entrytype-field) extensions table.

### Extension A: Internet domain names

Domain names resolved by DNS on the globally routed internet have the following [`entry.address`](#entryaddress-field) structure

| Field   | Type   | Size     | Description                                                     |
| ------- | ------ | -------- | --------------------------------------------------------------- |
| size    | uint_8 | 1        | Length of the domain name in bytes                              |
| name    | byte[] | variable | Domain name encoded in US-ASCII                                 |

DNS records for a domain name MUST have a valid `A` or `AAAA` entry.

#### `size` field

* This field MUST hold a value of at least 4
* This field MUST NOT hold a value greater than 253 pursuant to [RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035#section-2.3.4),
  taking into consideration the leading and trailing bytes as specified in [Section 3.1](https://datatracker.ietf.org/doc/html/rfc1035#section-3.1)

#### `name` field

* This field MUST only permit letters, digits and a hyphen (`-`) in each label pursuant to
  [RFC 1035](https://datatracker.ietf.org/doc/html/rfc1035#section-2.3.1).
  * A label is defined as a string of 63 characters or less beginning with a letter and ending with a letter or number
    separated by the period (`.`) delimiter.
* This field MUST NOT permit upper-case letters despite being permitted by RFC 1035 in light of its case-insensitivity.
* This field MUST have a string containing at least two labels separated by the specified delimiter.
  * "Dotless domains" (as described by [RFC 7085](https://datatracker.ietf.org/doc/html/rfc7085#section-2)) MUST NOT be
    permitted.
* This field MUST NOT permit top level domains associated with private/internal networks, including but not limited to,
  the TLD assigned by [RFC 6762](https://datatracker.ietf.org/doc/html/rfc6762#section-3) for multicast DNS, alternatives
  to it enumerated in [Appendix G](https://datatracker.ietf.org/doc/html/rfc6762#appendix-G) or the special-use domain
  assigned by [RFC 8375](https://datatracker.ietf.org/doc/html/rfc8375).

# References

* [BIP 0155: addrv2 message](https://github.com/bitcoin/bips/blob/17c04f9fa1ecae173d6864b65717e13dfc1880af/bip-0155.mediawiki)
* [RFC 1035: Domain Names - Implementation and Specification](https://datatracker.ietf.org/doc/html/rfc1035)
* [RFC 6762: Multicast DNS](https://datatracker.ietf.org/doc/html/rfc6762)
* [RFC 7085: Top-Level Domains That Are Already Dotless](https://datatracker.ietf.org/doc/html/rfc7085)
* [RFC 8375: Special-Use Domain 'home.arpa.'](https://datatracker.ietf.org/doc/html/rfc8375)
