# Assignment 9 - Financial Blockchain Capstone: African Remittances Payment Prototype

A simplified remittance platform. A sender converts rand (ZAR) into a simulated stablecoin (**mZAR**) at a 1:1 vault rule, sends it to a recipient on a proof-of-work blockchain using signed transactions, and the recipient redeems it back into rand. Every step is recorded on the chain, can be verified by anyone, and is exposed through a small payment API and a block explorer.


## Requirements

- Python 3.10 or newer
- `pip install flask ecdsa jupyter`

## Run it

1. Open `Assignment9_PartA_Presentation_Final.ipynb` in Jupyter.
2. Run Sections 1 to 7 and the **Demo setup** cell. 
3. Run the demo steps one cell at a time, in order. To rebuild everything, use Kernel > Restart and Run All.
4. Run the last cell to stop the API.

## Standard remittance path (the demo script)

| Step | API call | What to look for |
|---|---|---|
| i. Create wallets | `POST /wallets` for Sender (1 000 ZAR) and Recipient (0) | Addresses and public keys only. A Reserve agent is created with the service. |
| ii. On-ramp | `POST /onramp` `{"name": "Sender", "zar": 1000}` | Vault goes from 0 to 1 000 ZAR, Sender holds 1 000 mZAR, receipt issued |
| iii. Transfer | `POST /transfer` then `POST /mine` | 750 mZAR moves in a mined block, vault unchanged |
| iv. Off-ramp | `POST /offramp` `{"name": "Recipient", "amount": 750}` | 750 mZAR burned, vault down to 250 ZAR, receipt id issued |
| v. Verify | `GET /verify`, `GET /tx/<tx_id>`, `GET /explorer` | `valid: true`, transfer confirmed in its block |
| vi. Invalid transfer | `POST /submit` with a changed amount, `POST /transfer` over the balance, replay | HTTP 400 with the reason |



## API endpoints

| Endpoint | Purpose |
|---|---|
| `POST /wallets` | Create a wallet with a bank balance in ZAR |
| `GET /balance/<name>` | mZAR and ZAR balances |
| `POST /onramp` | ZAR to mZAR, with receipt |
| `POST /transfer` | Sign and queue a transfer |
| `POST /submit` | Queue a transaction signed elsewhere |
| `POST /mine` | Mine the waiting transactions into a block |
| `POST /offramp` | mZAR to ZAR, with receipt |
| `GET /verify` | Verify the whole chain |
| `GET /tx/<tx_id>` | Transaction lookup with block and confirmations |
| `GET /reserve` | Vault and supply report (the 1:1 rule) |
| `GET /receipts` | List receipts |
| `GET /explorer` | Readable view of every block |

## Design choices

- **1:1 vault rule.** ZAR goes into the vault before mZAR is minted, and mZAR is burned before ZAR is paid out. `GET /reserve` shows `backed_one_to_one`.
- **Proof of work, difficulty 3.** A block hash must start with three zeros. It is low so the demo is fast.
- **No mining reward.** A reward would create unbacked mZAR. Only the reserve agent can mint.
- **Signed transactions with nonces.** ECDSA (secp256k1) signatures can be verified by anyone with the public key. A per-sender nonce blocks replay.
- **Balances come from the chain.** They are rebuilt by replaying transactions, so they cannot disagree with the ledger.
- **Confirmation gate.** `MIN_CONFIRMATIONS = 1` in the demo. The off-ramp refuses to pay out if the latest incoming payment has fewer confirmations.


## Group details

GitHub link, member contributions and the AI declaration are in the report appendix.
- `<Thapelo Raseu and Lindokuhle Nsibande>`: Worked together on Part A.
- `<Ntumeleng Singo and Ontshiametse Aphane>`: Worked together on Part B and C
