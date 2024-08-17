# Glossary

There are some new concepts that team Chakra has raised during the implementation of the modular settlement layer. We have compiled this list to make sure that developers going through this documents are on the same page with us.&#x20;

**Settlement:** The process of finalizing a transaction or series of transactions, ensuring that all parties have fulfilled their obligations. In the context of the Chakra Settlement Layer, this involves converting assets into Settlement Artifacts and back.

**Handler Contract**: A smart contract within the Chakra Settlement Layer encapsulates token-related operations. It is the entry point for managing and executing settlements, ensuring that transactions are processed securely and efficiently.

**Settlement Primitive**: The method used to convert BTC (or other assets) into Settlement Artifacts. It also encompasses the reverse process, where Settlement Artifacts are converted back into BTC. Each Settlement Primitive may vary in terms of security, operational status, and financial scale, influencing the value of the associated Settlement Artifacts.

**Settlement Artifact**: An intermediate product used in the settlement process that cannot participate in market circulation. Its primary role is to serve as the underlying asset for LRT/LST within the Chakra Settlement Layer. Settlement Artifacts, in conjunction with Settlement Primitives, help maintain the security of BTC settlements. The market determines the value of a Settlement Artifact and is influenced by the characteristics of the corresponding Settlement Primitive. To read more on the utilities and benefits of Settlement Artifacts, please see [here](chakra-settlement-artifacts/).

**Quorum Certificate (QC)**: a cryptographic proof that a certain number of validators in a blockchain network have agreed on a particular piece of data, such as a block or a message. The QC is used to prove that a consensus has been reached among the validators.

**Cross-chain Transfer:** Transferring tokens or assets from one blockchain network to another using the Chakra Settlement Layer involves several steps, including initiating the transfer, processing it through Chakra validators, and finalizing it on the destination chain.

**Validator:** Validators are participants in the Chakra network responsible for confirming and processing transactions. They ensure the security and reliability of cross-chain transfers and settlements.

**Nonce Manager:** A component that manages and tracks the nonce values for transactions. It ensures that each transaction is unique and processed in the correct order.

**ERC20 Transfer Payload:** A structured data format used to encapsulate the details of an ERC20 token transfer, including information such as the sender, receiver, token amount, and associated metadata.

**Cross-chain Message:** A message containing encoded information about a cross-chain transfer. Chakra validators process it to facilitate the movement of assets between blockchain networks.

**Codec:** A component that encodes and decodes cross-chain messages and payloads. It ensures the data is correctly formatted for transmission and processing within the Chakra Settlement Layer.

**Payload Type:** An identifier specifies the payload type processed in a cross-chain message. Examples include ERC20 token transfers and other asset movements.

**CrossChainMsgStatus:** A status indicator for cross-chain messages. It denotes the current state of the message, such as pending, processed, or failed, ensuring proper tracking and handling of cross-chain transactions.
