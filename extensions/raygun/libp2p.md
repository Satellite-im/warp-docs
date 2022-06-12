## LibP2P Raygun Extension Overview

***Note: This extension requires a `MultiPass` extension and `warp-mp-solana` will be used for this overview. It also assumes you are using tokio for async runtime***

### Importing extension into cargo project

In your cargo project add the following

```
[dependencies]
warp = { git = "https://github.com/Satellite-im/Warp" }
warp-rg-libp2p = { git = "https://github.com/Satellite-im/Warp" }
warp-mp-solana = { git = "https://github.com/Satellite-im/Warp" }
```

### Starting Extension 

```rust
// Create an account
let mut solana_ext = SolanaAccount::with_devnet();
let mut tesseract = Tesseract::default();
tesseract.unlock(b"my super password")?;
solana_ext.set_tesseract(tesseract);
solana_ext.create_identity(None, None)?;

let account = Arc::new(Mutex::new(Box::new(solana_ext)));

// Create an instance dealing with libp2p

// Bootstrap nodes for querying the DHT. Optional, but helpful in discovering of peers and building out DHT
let bootstrap = vec![
        (
            "QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN",
            "/dnsaddr/bootstrap.libp2p.io",
        ),
        (
            "QmQCU2EcMqAqQPR2i9bChDtGNJchTbq5TbXJJ16u19uLTa",
            "/dnsaddr/bootstrap.libp2p.io",
        ),
        (
            "QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb",
            "/dnsaddr/bootstrap.libp2p.io",
        ),
        (
            "QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",
            "/dnsaddr/bootstrap.libp2p.io",
        ),
    ]
    .iter()
    .map(|(p, a)| (p.to_string(), a.to_string()))
    .collect::<Vec<(String, String)>>();

// This will listen on all ip addresses and a random port. Comment out this line and uncomment the next if you want a fixed port and a specific ip address.
let addr = "/ip4/0.0.0.0/tcp/0".parse().unwrap();
// let addr = "/ip4/10.0.0.15/tcp/4005".parse().unwrap();

// Creating a trait object 
let mut chat: Box<dyn RayGun> = Box::new(Libp2pMessaging::new(account.clone(), None, addr, bootstrap)
        .await?);
```

### Sending a Message

We will be using a random uuid for the conversation id.

```rust
let conversation_id = Uuid::new_v4();

// Note: This is used to "subscribe" to the conversation for the time being but will change to `chat.join_group(GroupId::from_id(conversation_id))?;

chat.ping(conversation_id).await?;

chat.send(conversation_id, None, vec!["Hello, World".into()]).await?;
```

Note: Although the message is sent and returned successfully, there is no guarantee users subscribed will receive it unless they are subscribed and connected to each other

### Obtaining a Message

Note: This function will return messages that are both sent and received. The best way to determine if it was from you or another user is to use the `Message::sender` for the sender and compare it against yours. 

```rust

let messages = chat.get_messages(conversation_id, Default::default(), None).await?;

``` 

### Editing a Message 

You are able to edit a message that you sent. To do so you would need to obtain the message id that you wish to edit. Note: You will only be able to edit what you sent, and unable to edit other messages 


```rust
let message = messages.get(0).unwrap();

let id = message.id();

chat.send(conversation_id, Some(id), vec!["My edited message".into()]).await?;

```

### Pinning a Message

```rust
chat.pin(conversation_id, id, PinState::Pin).await?;
```

You can unpin a message by using `PinState::Unpin`

```rust
chat.pin(conversation_id, id, PinState::Unpin).await?;
```

### Reacting to a Message

Note: You would be providing the unicode that represents the emoji that would be displayed. In this example, we would just be providing a string.

```rust
chat.react(conversation_id, id, ReactionState::Add, ":rocket:".into()).await?;
```

You can also remove your reaction by using `ReactionState::Remove` along with the emoji you wish to remove

```rust
chat.react(conversation_id, id, ReactionState::Remove, ":rocket:".into()).await?;
```

### Replying to a message

This would allow you to reply to a specific message that would be associated with your response.  

```rust
chat.reply(conversation_id, id, vec!["Commenting on this message".into()]).await?;
```

### Deleting a Message

Note: You are able to delete your message or the senders. This may change in the future to allow deleting the whole conversation from both ends. 

```rust
chat.delete(conversation_id, id).await?;
```