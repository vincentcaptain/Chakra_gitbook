# Configuration Script Template

Ensure that your `hardhat.config.ts` is correctly set up with the necessary network parameters. This configuration enables deployment to the appropriate blockchain networks.



```
networks: {
    scroll: {
        url: "https://scroll-sepolia.public.blastapi.io",
        accounts: ["<Your private key in this chain>"],
        gas: "auto",
        gasPrice: "auto"
    },
    chakra: {
        url: "https://rpcv1-dn-1.chakrachain.io",
        accounts: ["<Your private key in this chain>"],
        gas: "auto",
        gasPrice: "auto"
    },
    arb: {
		   url: "https://arbitrum-sepolia.public.blastapi.io",
       accounts: ["<Your private key in this chain>"],
		   gas: "auto",
       gasPrice: "auto"
    }
}

```
