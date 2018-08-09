# Generating Ethereum Addresses

In this exercise, we will choose a private key and generate the associated public key and Ethereum address from this private key using the *openssl* command line program.

Ethereum uses the [Elliptic Curve Digital Signature Algorithm (ECDSA)] for validating the origin and integrity of messages. In particular, Ethereum uses the mathematical constants defined in the [secp256k1 standard]. See [What is the math behind elliptic curve cryptography?] for more details on the maths. See [How are ethereum addresses generated?] for a description of this process.

**Note** that this is not a recommended way to generate your keys. This exercise just demonstrates the relationship between the private key and the associated Ethereum address.

<br />

<hr />

## Table Of Contents

* [Private Key](#private-key)
* [Generate Public Key](#generate-public-key)
* [See Also](#see-also)

<br />

<hr />

### Private Key

The private key in Ethereum is a 32 byte (or 256 bit) random number with a range from 1 to 8^32 (or 2^256). Not all numbers in this range form a valid private key. From [here](https://github.com/ethereum/go-ethereum/blob/cab1cff11cbcd4ff60f1a149deb71ec87413b487/crypto/crypto.go#L114-L116) `0000000000000000000000000000000000000000000000000000000000000001` is the first valid number. From [here](https://github.com/ethereum/go-ethereum/blob/cab1cff11cbcd4ff60f1a149deb71ec87413b487/crypto/crypto.go#L38) and [here](https://github.com/ethereum/go-ethereum/blob/cab1cff11cbcd4ff60f1a149deb71ec87413b487/crypto/crypto.go#L110-L112) `fffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141` is the maximum number.

**Note** that it is very important that your real private keys are generated from a secure random number generator - see [Secure Cryptocurrency Seeds] and [Why shouldn't we roll our own?].

The [Ethereum Private Key Database] provides a list of 32 private key / Ethereum address pairs per page in  3618502788666131106986593281521497120401173883721090761956411348172442546699 pages. Click on a few Ethereum address on the first page and check if there are any ethers in these accounts.

We will choose the last private key from the first page of [Ethereum Private Key Database] for this exercise - `000000000000000000000000000000000000000000000000000000000000001f` - the associated Ethereum address for this private key is `0x6b09D6433a379752157fD1a9E537c5CAe5fa3168`.

<br />

<hr />

### Generate Public Key

From [How to convert an ECDSA key to PEM format], we want to add the prefix `302e0201010420` before the private key and add the postfix `a00706052b8104000a` after the private key. Our hex string is now `302e0201010420000000000000000000000000000000000000000000000000000000000000001fa00706052b8104000a`.

On OS/X with *openssl* installed, run the following command. See [Command Line Elliptic Curve Operations] for further information on *openssl*. The `xxd` command converts the hex string into binary form to be fed into *openssl* [DER](https://wiki.openssl.org/index.php/DER) format.

```bash
$ echo "302e0201010420000000000000000000000000000000000000000000000000000000000000001fa00706052b8104000a" | xxd -r -p - | openssl ec -inform der -noout -text
read EC key
Private-Key: (256 bit)
priv: 31 (0x1f)
pub: 
    04:6a:24:5b:f6:dc:69:85:04:c8:9a:20:cf:de:d6:
    08:53:15:2b:69:53:36:c2:80:63:b6:1c:65:cb:d2:
    69:e6:b4:e0:22:cf:42:c2:bd:4a:70:8b:3f:51:26:
    f1:6a:24:ad:8b:33:ba:48:d0:42:3b:6e:fd:5e:63:
    48:10:0d:8a:82
ASN1 OID: secp256k1
```

We need to remove the `04` prefix and remove the colons from the *pub* key above to generate the public key `6a245bf6dc698504c89a20cfded60853152b695336c28063b61c65cbd269e6b4e022cf42c2bd4a708b3f5126f16a24ad8b33ba48d0423b6efd5e6348100d8a82`.

Using the `geth console` JavaScript command, we calculate the [Keccak]-256 hash of the public key. See [Which cryptographic hash function does Ethereum use?] for more details on keccak-256.

```javascript
> web3.sha3("6a245bf6dc698504c89a20cfded60853152b695336c28063b61c65cbd269e6b4e022cf42c2bd4a708b3f5126f16a24ad8b33ba48d0423b6efd5e6348100d8a82", {encoding: 'hex'});
"0x37c33922dc113a7af93268576b09d6433a379752157fd1a9e537c5cae5fa3168"
```

You can also use the [Online SHA-3 Keccak Calculator], setting *Hash Size* to **256**, *Function* to **Keccak**, *Format* to **Hex**, pasting the public key into the *Message* text box and clicking *Calculate*.

We need to take the last 20 bytes (40 hex characters) of the keccak-256 hash to compute the Ethereum address:

```javascript
> web3.sha3("6a245bf6dc698504c89a20cfded60853152b695336c28063b61c65cbd269e6b4e022cf42c2bd4a708b3f5126f16a24ad8b33ba48d0423b6efd5e6348100d8a82", {encoding: 'hex'}).substring(26);
"6b09d6433a379752157fd1a9e537c5cae5fa3168"
```

The generated Ethereum public key is `6b09d6433a379752157fd1a9e537c5cae5fa3168`. Compare this to the public key from the [Ethereum Private Key Database] of `0x6b09D6433a379752157fD1a9E537c5CAe5fa3168` and note that some of the alphabetic hexadecimal characters are uppercase. [EIP-55 Mixed-case checksum address encoding] describes the mix-case checksum algorithm.

<br />

<hr />

### See Also

* [Create full Ethereum wallet, keypair and address](https://kobl.one/blog/create-full-ethereum-keypair-and-address/)
* [Mastering Ethereum - Keys and Addresses](https://github.com/ethereumbook/ethereumbook/blob/develop/keys-addresses.asciidoc)
* [Account uniqueness guaranteed?](https://ethereum.stackexchange.com/a/4300/1268)

<br />

<br />

(c) BokkyPooBah / Bok Consulting Pty Ltd - Aug 10 2018. Free for personal use. Please contact BokkyPooBah for commercial use.

[Ethereum Private Key Database]: https://ethdir.io/
[Elliptic Curve Digital Signature Algorithm (ECDSA)]: https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm
[secp256k1 standard]: http://www.secg.org/sec2-v2.pdf
[Secp256k1]: https://en.bitcoin.it/wiki/Secp256k1
[What is the math behind elliptic curve cryptography?]: https://hackernoon.com/what-is-the-math-behind-elliptic-curve-cryptography-f61b25253da3
[How to convert an ECDSA key to PEM format]: https://stackoverflow.com/a/48102827
[How are ethereum addresses generated?]: https://ethereum.stackexchange.com/a/3619/1268
[EIP-55 Mixed-case checksum address encoding]: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md
[Online SHA-3 Keccak Calculator]: https://leventozturk.com/engineering/sha3/
[Keccak]: https://keccak.team/keccak.html
[Which cryptographic hash function does Ethereum use?]: https://ethereum.stackexchange.com/questions/550/which-cryptographic-hash-function-does-ethereum-use
[Command Line Elliptic Curve Operations]: https://wiki.openssl.org/index.php/Command_Line_Elliptic_Curve_Operations
[Why shouldn't we roll our own?]: https://security.stackexchange.com/questions/18197/why-shouldnt-we-roll-our-own
[Secure Cryptocurrency Seeds]: https://medium.com/@peterryszkiewicz/secure-cryptocurrency-seeds-fdd99f39df7e