# _tile_: a time-locked encryption API ⏱🔐

_tile_ is a simple API for time-locked encryption (aka timed-release encryption).
_tile_ publishes X25519 public keys, and _also_ the corresponding private keys,
but only _after_ a predefined date in the future.

![Yale Model #2, c. 1890 Single Pin Dial Time Lock](http://www.my-time-machines.net/Yale_Single_Pin_Dial-100a-4.jpg)

## API

_tile_ is very easy to use.
<details>
  <summary>Retrieve a public key</summary>
  
```Shell
curl -s \
     -d '{"reveal_not_before": "2020-08-21T18:25:43.511Z",
          "keyring_label": "2020-08-20_mushroom-paradox"}' \
     -H "Content-Type: application/json" \
     -X POST \
     https://tile-experimental.cryptographic.services:8082/v0/public_key \
     | jq
```

The parameters of the request are:
* `keyring_label`. See table below for possible values.
* `reveal_not_before`. An approximate time for disclosing the corresponding private key.


Typical successful response

```JSON
{
    "data": {
        "key_serial": 1598034600,
        "keyring_label": "2020-08-20_mushroom-paradox",
        "public_key": "bNhrEUObdoBbzeFsdb+W5bj/S6LwSr1nESb5b35wdWY=",
        "reveal_after": "2020-08-21T18:30:00+00:00Z"
    }
}
```

* The `public_key` field is the base64 representation of a X25519
public key, suitable for use with libsodium or NaCl.
* The `reveal_after` field is the "rounded up" version of the passed 
`reveal_not_before` parameter.
* The `key_serial` is an identifier that can be used to retrieve the
corresponding private key whenever is available.

The public keys are also available as a batch [in this folder](pk/).

</details>

<details>
  <summary>Retrieve a private key</summary>
  
```Shell
curl -s \
   -d '{"key_serial": "1598034600", "keyring_label":"2020-08-20_mushroom-paradox"}'  \
   -H "Content-Type: application/json" \
   -X POST \
   https://tile-experimental.cryptographic.services:8082/v0/private_key \
   | jq
```

Typical successful response:

```JSON
{
    "data": {
        "key_serial": 1598034600,
        "keyring_label": "2020-08-20_mushroom-paradox",
        "private_key": "Bd5ZOhlg5DRiKwXTK5fVnfYvlyy6MweSqcch1goZC7Q=",
        "reveal_after": "2020-08-21T18:30:00+00:00Z"
    }
}
```

If you ask for a key not yet revealed you'll get this response:

```JSON
{
    "error": {
        "code": "400",
        "message": "key not yet revealed"
    }
}
```

</details>

## Implementation

_tile_ is dead simple. The current version of tile
uses bread and butter crypto. Key storage and
computation happens on an embedded system
behind a data diode and without external access.
Meaning, it won't be easy for you to hack it from your couch.
I use a GPS module to keep track of time.

The current trust model is easy: you'll have to fully trust me.
You can deploy/build your own service to relax this assumption.
I'll add a write up in the future on how to do this.

I cannot provide any uptime guarantees at the moment. I'm hoping to keep this running
for some months, but consider all this experimental and best effort at best.
I built this functionality because I needed it and thought it could be interesting to others.

Note that public keys need to be certified somehow. For the moment we
just rely on transport security when retrieving them. If I signed them,
would you verify them? Which key would you use?

## Applications

Time-locked encryption is a useful building block for more complex functionality,
such as sending your diary to the future,
fair bidding or broadcast encryption.

## Key labels

| label                        | description  | end of life |
|------------------------------|--------------|-------------|
| 2020-08-20_purple-verse      | Staging (DO NOT USE) | n/a |
| 2020-08-20_mushroom-paradox  | "Production"         | 2020-08-31 |

## References

Picture above of a Yale Single Pin Dial Time Lock from around 1890 
is taken from the
[Magnificent Time Machines by Mark Frank](http://www.my-time-machines.net/timelock_index.htm).

## Contact

Oscar Reparaz <firstname.lastname@esat.kuleuven.be>
