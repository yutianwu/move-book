# 账户

<!--

- user is an account
    - account is identified by an address
    - account is generated from a private key
    - account can own objects
    - account can send transactions
    - every transaction has a sender
    - sender is identified by an address
    - sui cryptographic agility
    - sui account types
    - supported curves: ed25519, secp256k1, zklogin

 -->

账户是识别用户的一种方式。账户从私钥生成，并由地址标识。账户可以拥有对象，可以发送交易。每个交易都有发送者，发送者由[地址](./address)标识。

Sui支持多种用于账户生成的加密算法。支持的两种曲线是ed25519、secp256k1，还有一种特殊的账户生成方式——zklogin。加密敏捷性——Sui的独特功能——允许账户生成的灵活性。

<!-- The cryptographic agility allows for flexibility in the account generation -->

## 延伸阅读

- [Sui博客](https://blog.sui.io)中的[Sui加密学](https://blog.sui.io/wallet-cryptography-specifications/)
- [Sui文档](https://docs.sui.io)中的[密钥和地址](https://docs.sui.io/concepts/cryptography/transaction-auth/keys-addresses)
- [Sui文档](https://docs.sui.io)中的[签名](https://docs.sui.io/concepts/cryptography/transaction-auth/signatures)