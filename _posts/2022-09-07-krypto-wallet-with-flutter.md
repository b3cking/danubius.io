---
layout: post
author: Bence Ignácz
title: How to create crypto wallet mobile app with Flutter
image: crypto-wallet-with-flutter-cover.png
keywords: performance, productivity, selfcare, work-life balance, test
---
{% include image.html src="/images/crypto-wallet-with-flutter-cover.png" alt="Crypto wallet with Flutter" %}

This post will show you how to create a bare minimum Mobile Crypto Wallet application with Flutter. This is not a step-by-step tutorial, it just highlights the critical solutions and keywords. 

<!--more-->

## What is a wallet

Crypto wallets store your private key(s) for your address(es). Your address is the key to the door of the blockchain. An address represents an account on the network. With a wallet, you can easily access your account, and send, receive and spend tokens like Bitcoin or Ether.

## A bare minimum crypto wallet

{% video ./videos/krypto_wallet.mp4 720 ./images/crypto-wallet-video-preview.png %}

A bare minimum crypto wallet application can create and store a new wallet and address, send and receive tokens. To support the process of address exchange between the participants, we implement a QR generator screen and a QR scanner screen. The QR code contains the recipient's address.
For easy network change, we add a dropdown list with network names. 


## Project setup

We have used the following dependencies:
* `web3dart`: Dart implementation for web3. It is a bit tricky because it has 2 different repositories and one is deprecated. This is the current latest: [xclud/web3dart](https://github.com/xclud/web3dart).
* `bip39`: Bip39 specification implementation for Dart. More about bip39 later...
* `shared_preferences`: To store the generated wallet. **This is not secure, don't use it in production!**
* `mobile_scanner`: For QR code reading.
* `qr_flutter`: For QR code generation.

Our sample application using the following versions:
```yaml
  cupertino_icons: ^1.0.2
  web3dart: ^2.4.1
  bip39: ^1.0.6
  shared_preferences: ^2.0.15
  mobile_scanner: ^2.0.0
  qr_flutter: ^4.0.0
```

## Using Ganache

Ganache is a one-click blockchain for development purposes. This cool tool is a member of the Truffle suite.
You can download it from the [Truffle Suite page](https://trufflesuite.com/ganache/).
Ganache is fully simulating a blockchain network. If you installed and started, you got a default workspace with 10 pre-built addresses and passphrases for testing.
You can create your own workspace and you can customize the account pre-seed, gas fees, block time, etc.
To access to Ganache server from a device or emulator, you need to configure the server to listen on all interfaces. Click on the cog icon on the top right side and select the server tab. Change the hostname to `0.0.0.0 - All Interfaces`.

{% include image.html src="/images/crypto-wallet-ganache.png" alt="Ganache" %}

## Recovery Phrase and BIP-39

Wallets are using asymmetric encryption. Hence they have a private and a public key. As the name is mentioned, the private key is confidential. You should never share it. To move your wallet address to another wallet app, you need to provide the keys. Usually, the private key is generated from a security phrase (or seed phrase). This seed is a list of 12 or 24 words, generated randomly. The recovery share should keep in a secure way by the end user. If she/he lost it, he lost the wallet as well.

BIP-39 comes from Bitcoin Improvement Proposal: 39. This is a standard and contains 2048 easy-to-remember words.

To generate a BIP-39 compliant word list, use the `bip39.generateMnemonic()` function.

{% include image.html src="/images/crypto-wallet-seed-phrase.png" alt="Seed phrase" %}

## Create private key and generate wallet

To create a private key with web3dart we can use the EthPrivateKey class. Before creating a key from hex we need to convert the recovery phrase to hexadecimal representation. There is a helper method in bip39 lib, the `mnemoicToSeedHex(mnemonic)` function. The second important thing is protecting the wallet with a password. And the last one is a random number for the cryptography algorithm to generate a secure wallet.
This is a sample implementation:

```java
    var mnemonic = bip39.generateMnemonic();
    var passwd = "supers3cr3tp4word";
    var fromHex = mnemonicToSeedHex(mnemonic);
    EthPrivateKey privateKey = EthPrivateKey.fromHex(fromHex);
    Wallet wallet =  Wallet.createNew(privateKey, passwd, Random.secure());
```

## Load Wallet and connect to network using Infura

To load the wallet from a JSON file, you can use `Wallet.fromJson(jsonDecode(walletJson), walletPassword)` function. Infura is an Infrastructure as a Service solution and toolset for easy development.
We create a Web3 client with an HTTP provider and connect to the Rinkbey testnet. You can register and create your free API key on [infura.io](https://infura.io/)

> Rinkeby test network is deprecated from 5th October. You can use Goerli testnet.

```java
  var apiUrl = "https://rinkeby.infura.io/v3/<YOUR_API_KEY>";
  var httpClient = Client();
  var ethClient = Web3Client(apiUrl, httpClient);
```

After creating the client you can get the credentials and get the balance of the address:

```java
var credentials = EthPrivateKey.fromInt(wallet!.privateKey.privateKeyInt);
ethClient
    .getBalance(credentials.address)
    .then((balance) => {
          setState(() {
            _address = credentials.address.toString();
            _balance = balance.getInWei.toDouble() * 0.000000000000000001;
          })
        })
    .onError((error, stackTrace) => {
          ScaffoldMessenger.of(context).showSnackBar(
            const SnackBar(
              backgroundColor: Colors.red,
              content: Text(
                'Unable to connect to network!',
                style: TextStyle(color: Colors.white),
              ),
            ),
          ),
        });
```

{% include image.html src="/images/crypto-wallet-home.png" alt="Wallet home" %}

## Send tokens to address

From this point sending tokens is very straightforward. Just create a transaction, and define the max gas fee and the amount to send. You can use predefined units with EtherUnit class. In our example, we are using Wei.

```java
Transaction transaction = Transaction(
  to: EthereumAddress.fromHex(to, enforceEip55: true),
  maxGas: 100000,
  value: EtherAmount.fromUnitAndValue(EtherUnit.wei, amount),
);
```

Then you can send the transaction to the chain with the Web3 client. If the transaction has been sent, you will receive the transaction hash. You can check your transaction and mining state on the Ganache dashboard.

```java
_ethClient?.sendTransaction(_credentials!, transaction)
  .then((value) {
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(
        content: Text('Transaction Sent'),
        backgroundColor: Colors.green,
      ),
    );
    setState(() {
      transactionSignature = value;
    });
  }).onError((error, stackTrace) {
    print(error);
    print(stackTrace);
});
```

That's all. The UI is up to you
