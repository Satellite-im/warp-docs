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
    // Use development network.
    let mut account = SolanaAccount::with_devnet();
    // This is to allow `Tesseract` to be shared across multiple threads in a safe manner

    // In real world application, you would want this to be loading from a file 
    let mut tesseract = Tesseract::default();
    // This is to allow `Tesseract` to be shared across multiple threads in a safe manner
    let tesseract = Arc::new(Mutex::new(tesseract));
    tesseract.unlock(b"this is my totally secured password that should nnever be embedded in code").unwrap();
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


### Sending/Receiving Friend Request and Managing Friends List

***TODO***