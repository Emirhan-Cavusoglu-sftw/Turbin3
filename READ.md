


# Solana Airdrop, Transfer, and Enrollment Project

This project demonstrates key interactions with the Solana blockchain using TypeScript and the Solana Web3.js library. The primary scripts in this project allow users to create a Solana wallet, claim an airdrop of Devnet SOL tokens, transfer SOL between accounts, and interact with a program on the Solana Devnet.

## Transaction Proof

You can view the successful final project transaction on Solana Devnet Explorer using the link below:

[Check out the transaction here](https://explorer.solana.com/tx/3HuVDdHhJSVNENRzT8zvePszLcbBbUryCYGeUW9wq5eY2dcyAZk4XBGHEkvXsXe7Tdvre7tWqEz3B7i5rFeLPELY?cluster=devnet)

## Project Structure

This repository contains the following TypeScript scripts:

- **keygen.ts**: This script generates a new Solana keypair (wallet), displaying the public and secret keys for the wallet.
- **airdrop.ts**: Using this script, users can request a SOL airdrop on the Solana Devnet for the generated wallet. The wallet’s secret key is imported from the `dev-wallet.json` file.
- **transfer.ts**: This script allows users to transfer 0.1 SOL from the primary wallet to another wallet on the Solana Devnet.
- **enroll.ts**: This script interacts with a program on the Solana Devnet using Anchor, registering the user’s GitHub username to a Program Derived Address (PDA) associated with their wallet. The program uses the IDL (Interface Definition Language) to interact with the contract on-chain.

## How to Run

### 1. Install Dependencies

Ensure you have Node.js and Yarn installed on your machine. Then, run the following command to install the necessary dependencies:

```bash
yarn install
```

### 2. Generate a Keypair

Generate a new Solana keypair by running:

```bash
yarn keygen
```

This will create a keypair and display the public and secret keys.

```typescript
import { Keypair } from "@solana/web3.js";

// Generate a new keypair
let kp = Keypair.generate();
console.log(`You've generated a new Solana wallet: ${kp.publicKey.toBase58()}`);
console.log(`Solana Wallet Secret Key: ${kp.secretKey}]`);
```

The output will look something like this:

```plaintext
You've generated a new Solana wallet: 2sNvwMf15WPp94kywgvfn3KBNPNZhr5mWrDHmgjkjMhN
Solana Wallet Secret Key: [63,224,142,...]
```

To save the private key locally, create a `dev-wallet.json` file and paste the secret key:

```json
[63,224,142,...]
```

### 3. Claim Airdrop

Run the airdrop script to claim 2 SOL on Solana Devnet:

```bash
yarn airdrop
```

This script connects to the Solana Devnet and requests 2 SOL for your wallet.

```typescript
import { Connection, Keypair, LAMPORTS_PER_SOL } from "@solana/web3.js";
import wallet from "./dev-wallet.json";

const keypair = Keypair.fromSecretKey(new Uint8Array(wallet));
const connection = new Connection("https://api.devnet.solana.com");

(async () => {
    try {
        const txhash = await connection.requestAirdrop(keypair.publicKey, 2 * LAMPORTS_PER_SOL);
        console.log(`Success! Check out your TX here: 
        https://explorer.solana.com/tx/${txhash}?cluster=devnet`);
    } catch (e) {
        console.error(`Oops, something went wrong: ${e}`);
    }
})();
```

### 4. Transfer SOL

To transfer 0.1 SOL from your wallet to another wallet, run:

```bash
yarn transfer
```

In `transfer.ts`, you will import necessary components and define the second wallet’s public key.

```typescript
import { Transaction, SystemProgram, Connection, Keypair, PublicKey, sendAndConfirmTransaction } from "@solana/web3.js";
import wallet from "./dev-wallet.json";

const from = Keypair.fromSecretKey(new Uint8Array(wallet));
const to = new PublicKey("Second wallet public key here");
const connection = new Connection("https://api.devnet.solana.com");

(async () => {
    try {
        const transaction = new Transaction().add(
            SystemProgram.transfer({
                fromPubkey: from.publicKey,
                toPubkey: to,
                lamports: LAMPORTS_PER_SOL / 100,
            })
        );
        transaction.recentBlockhash = (await connection.getLatestBlockhash('confirmed')).blockhash;
        transaction.feePayer = from.publicKey;

        const signature = await sendAndConfirmTransaction(connection, transaction, [from]);
        console.log(`Success! Check out your TX here: 
        https://explorer.solana.com/tx/${signature}?cluster=devnet`);
    } catch (e) {
        console.error(`Oops, something went wrong: ${e}`);
    }
})();
```

### 5. Complete Enrollment

Finally, run the enrollment script:

```bash
yarn enroll
```

This script will submit your GitHub username to the WBA program on the Solana Devnet using a PDA.

```typescript
import { Connection, Keypair, PublicKey } from "@solana/web3.js";
import { Program, Wallet, AnchorProvider } from "@coral-xyz/anchor";
import { IDL, WbaPrereq } from "./programs/wba_prereq";
import wallet from "./dev-wallet.json";

const keypair = Keypair.fromSecretKey(new Uint8Array(wallet));
const connection = new Connection("https://api.devnet.solana.com");
const github = Buffer.from("<your github account>", "utf8");

const provider = new AnchorProvider(connection, new Wallet(keypair), { commitment: "confirmed" });
const program: Program<WbaPrereq> = new Program(IDL, provider);

const enrollment_seeds = [Buffer.from("prereq"), keypair.publicKey.toBuffer()];
const [enrollment_key, _bump] = PublicKey.findProgramAddressSync(enrollment_seeds, program.programId);

(async () => {
  try {
    const txhash = await program.methods
      .complete(github)
      .accounts({
        signer: keypair.publicKey,
      })
      .signers([keypair])
      .rpc();
    console.log(`Success! Check out your TX here:
https://explorer.solana.com/tx/${txhash}?cluster=devnet`);
  } catch (e) {
    console.error(`Oops, something went wrong: ${e}`);
  }
})();
```

## Required Environment

- Node.js
- Yarn
- Solana Web3.js
- Anchor Framework



```
