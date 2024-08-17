# Introduction to the ChakraGasService Contract

`ChakraGasService` is a smart contract designed to manage gas payments and refunds for cross-chain communications within the Chakra settlement layer network. When users initiate a one-way or two-way cross-chain settlement, the gas required includes:

* Prepaid gas on the source chain
* Prepaid gas on the destination chain
* Prepaid gas for the Chakra network

All prepaid gas is paid to the Chakra smart contract, which manages gas payments and refunds for cross-chain communications on the Chakra settlement layer network. Figure below illustrates all possible gas-charging points in the end-to-end process from end-user settling funds from source chain through Chakra to the end-user on the target chain.&#x20;

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

