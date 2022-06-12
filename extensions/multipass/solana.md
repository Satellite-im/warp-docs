# Solana MultiPass Extension Overview



## Importing extension into cargo project

In your cargo project add the following

```
[dependencies]
warp = { git = "https://github.com/Satellite-im/Warp" }
warp-extensions = { git = "https://github.com/Satellite-im/Warp", features = ["mp_solana"] }
```

## Starting Extension 

Here you will also need to utilize tesseract when utilizing multipass extension

```rust

    // In real world application, you would want this to be loading from a file 
    let mut tesseract = Tesseract::default();
    tesseract.unlock(b"this is my totally secured password that should nnever be embedded in code").unwrap();
    // Use development network.
    let mut account = SolanaAccount::with_devnet(&tesseract);

```

## Creating a new account

### With a username

```rust
    account.create_identity(Some("NewAcctName"), None).unwrap();
```
### Without a username

**Note: If a username is not defined, one will automatically be generated**

```rust
    account.create_identity(None, None).unwrap();
```

After the function successfully executes, there will be two encrypted fields within tesseract called `mnemonic` and `privkey`. 
If these fields exist and contained valid information related to a registered account, this will not be required and would
return an error.

## Fetching Account 

### Your own

```rust
    let ident = account.get_own_identity().unwrap();
```

### By Public Key
```rust
    let ident = account.get_identity(Identifier::public_key(PublicKey::from_bytes(....))).unwrap()
```

### By Username

**At this time, fetching by username is only available to find accounts that are cached that were previously looked up via public key.**

## Getting private key

```rust
    let privkey = account.decrypt_private_key(None).unwrap();
```

This returns an array of bytes that is related for your private key. You would need to import them into the proper keypair to utilize them directly.


## Sending/Receiving Friend Request and Managing Friends List

***Note: Many of the functions for handling friends would error if you attempt to add or manage yourself within your account***

This overview will assume that our friend public key is `68vtRPQcsV7ruWXa6Z8Enrb6TsXhbRzMywgCnEVyk7Va`. This public key would get decoded using bs58 (or any base58 decoder) into an array of bytes by doing `let pubkey = bs58::decode("68vtRPQcsV7ruWXa6Z8Enrb6TsXhbRzMywgCnEVyk7Va").into_vec().map(PublicKey::from_vec).unwrap()`

### Sending a Friend Request

You can send a friend request with anybody who has an solana account. 

```rust
    account.send_request(pubkey).unwrap();
```

### Accept a Friend Request

You can accept a friend request sent to you.

```rust
    account.accept_request(pubkey).unwrap();
```

### Deny a Friend Request

You would be able to deny a friend request. Note that once it is denied, the account would need to send another if you wish for them to be friends.

```rust
    account.deny_request(pubkey).unwrap();
```

### Close a Friend Request

```rust
    account.close_request(pubkey).unwrap();
```

### List Incoming Request

This would allow you to view all incoming request that are pending acceptance or denial.

```rust
    account.list_incoming_request(pubkey).unwrap();    
```

### List Outgoing Request

This would allow you to view all outgoing request you have sent that have yet to be accepted or denied.

```rust
    account.list_outgoing_request(pubkey).unwrap();
```

### List All Request

You would be able to view all incoming and outgoing request regardless of their status.

```rust
    account.list_all_request(pubkey).unwrap();
```

### Remove Friend

You would be able to remove a friend from your friend list.  

```rust
    account.remove_friend(pubkey).unwrap();
```

### Block Friend

You would be able to block a friend from your friend list. ***Note: This functionality is not implemented but currently a placeholder***  

```rust
    account.block_friend(pubkey).unwrap();
```


### List Current Friends

This will list all your friends, returning their `Identity`.

```rust
    account.list_friends().unwrap()
```

### Check Friendship Status

You would be able to check to determine if you are friends with somebody. Will return true if you are friends and false if you are not.

```rust
    account.has_friend(pubkey)
```


### Key Exchange

This would allow you to perform a Key Exchange (ECDH) with a friend. ***Note: This function requires a X25519 Public Key to be provided, which can be created using `X25519Secret::public_key` after importing the ED25519 Keypair using `X25519Secret::from_ed25519_keypair` and importing the bytes into `PublicKey` that is supplied by MultiPass***

```rust
    let x25519_pubkey = PublicKey::from(.....);

    account.key_exchange(x25519_pubkey);
```

***Notice: This function may be removed to allow key exchange to be handled outside of MultiPass. It is recommended that key exchange is performed externally for the time being.***