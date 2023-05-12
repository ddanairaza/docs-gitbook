---
description: This tutorial describes using Beam Shaders on DAppnet
---

# Using Beam Shaders

{% hint style="success" %}
Why DAppnet?&#x20;

Beam consists of three public networks:&#x20;

1. **DAppnet:** a testing sandbox that produces a pseudo-mining block every 15 seconds.&#x20;
2. **Testnet:** produces genuine mining blocks but is mostly a testing site before Mainnet deployment.
3. **Mainnet:** the living Beam network with real assets.
{% endhint %}

### Install DAppnet wallet

The latest version of the Beam DAppnet Wallet is on our [website](https://dappnet.beam.mw/downloads/).

When [installing DAppnet](https://beamx.gitbook.io/dappnet-user-guide/settings), choose the default settings to operate the local node (required) to run Beam Shader contracts.  Create a new wallet (save the seed phrase).&#x20;

{% hint style="info" %}
Beam Shaders require the use of the integrated node found in the DAppnet Wallet and must remain open for this tutorial.
{% endhint %}

### Withdraw from the Faucet

All DAppnet transactions (including Beam Shader deployment) contain transaction fees that are paid in Beam coins, and require a positive wallet balance.

Launch [My DApp Store](https://beamx.gitbook.io/dappnet-user-guide/my-dapp-store) <img src=".gitbook/assets/Screen Shot 2023-05-10 at 10.27.02 PM.png" alt="" data-size="line"> and find the Faucet Application.&#x20;

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Launch **Faucet >>> Get your first Beam >>> Confirm,**  and Beam coins will deposit into your wallet after the next mining block.

<figure><img src=".gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

### Configuring your CLI wallet

Beam shaders are deployed via CLI wallet which can be found on our [website](https://dappnet.beam.mw/downloads/). This tutorial uses Windows Command Prompt, but users can substitute [Mac](shader-sdk-mac.md) and Linux commands if desired.

Create a `shader` folder in your hard drive, and transfer CLI wallet archive. The `shader` folder should contain two files: two files: the `CLI wallet archive` and wallet `config` file.&#x20;

{% hint style="danger" %}
For this tutorial, we will use our DAppnet's `wallet.db`(for testing purposes only. This practice is not recommended with wallets containing real assets).
{% endhint %}

### Copy `wallet.db` from DAppnet Wallet

Beam wallets store wallet metadata (transaction history, balance, etc) locally on your hard drive in the `wallet.db` file.&#x20;

Locate your DAppnet `wallet.db` in the `%LOCALAPPDATA%\Beam Wallet folder`. Transfer the `wallet.db` in the same `shader` folder.

{% hint style="info" %}
Refer to our [Desktop Wallet guide](beam-wallets/desktop-wallet.md) for `wallet.db` file location for macOS and Linux operating systems.


{% endhint %}

<figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### Configure CLI wallet settings

One more crucial step before running Shader contracts involves configuring the CLI wallet settings to simplify and shorten commands.&#x20;

To connect CLI wallet's to the built-in node, first edit the `config` file using text editor (such as [Visual Studio Code](https://code.visualstudio.com/)) and enter the following:

```
log_level=debug
pass=<your wallet password>
node_addr=127.0.0.1:10005
wallet_path=wallet.db
```

The `config` file should resemble the following:

<pre><code><strong>################################################################################
</strong># General options:
################################################################################

# log level [info|debug|verbose]
log_level=debug

# file log level [info|debug|verbose]
# file_log_level=debug

# old logs cleanup period (days)
# log_cleanup_days=5

################################################################################
# Wallet options:
################################################################################

# password for the wallet
pass=123

# phrase to generate secret key according to BIP-39.
# seed_phrase=

# address of node
node_addr=127.0.0.1:10005

# path to wallet file
wallet_path=wallet.db

# command to execute [new_addr|send|receive|listen|init|info|export_miner_key|export_owner_key|generate_phrase]
# command=listen


</code></pre>

The testing environment is ready to deploy Shaders, all that remains is selecting the Shader contract.

### Choosing a  Shader application&#x20;

The [Dappnet Blockchain Explorer](https://dappnet.explorer.beam.mw/) (see **Contracts** tab) contains a list of Shader contract applications ready for testing.&#x20;

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

For the purposes of this tutorial, we will select the **Vault** Shader application.

{% hint style="info" %}
At thid time, the list of available contracts is DAppnet. The majority of deployed contracts don't contain a contract description. These features will be adjusted with future iterations of DAppnet.
{% endhint %}

**Please download the Vault application Shader from our** [**Beam repository.** ](https://github.com/BeamMW/beam/tree/master/bvm/Shaders/vault)The Shader contract will not respond to CLI commands Shader can respond to CLI commands without the precompiled source code.

Download [`app.wasm`](https://github.com/BeamMW/beam/blob/master/bvm/Shaders/vault/app.wasm) and copy it to your `shader` folder. **Note**: if you are testing several applications, create a folder for each Shader application so each application contains a sperate `app.wasm` file.

### Running Shader commands

Open as command line interface and change directory to the location of the CLI wallet executable.&#x20;

First thing we will do is print the API of the contract by running the following command:

`beam-wallet-dappnet.exe shader --shader_app_file vault\app.wasm`



Note that we only provide one parameter (--shader\_app\_file) since we assume all other parameters are set in the configuration file. We also provide a path to the application shader assuming it is located in the 'vault' folder.

The output of this command will look something like this (debug logs are on in this example)

<figure><img src=".gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Let's copy the Shader output part into the text editor and format it to a standard JSON form. The end results will look like this:

```json
 {
     "roles":
     {
         "manager":
         {
             "create":
             {},
             "destroy":
             {
                 "cid": "ContractID"
             },
             "view":
             {},
             "view_logs":
             {
                 "cid": "ContractID"
             },
             "view_accounts":
             {
                 "cid": "ContractID"
             },
             "view_account":
             {
                 "cid": "ContractID",
                 "pubKey": "PubKey"
             }
         },
         "my_account":
         {
             "view":
             {
                 "cid": "ContractID"
             },
             "get_key":
             {
                 "cid": "ContractID"
             },
             "get_proof":
             {
                 "cid": "ContractID",
                 "aid": "AssetID"
             },
             "deposit":
             {
                 "cid": "ContractID",
                 "pkForeign": "PubKey",
                 "bCoSigner": "uint32_t",
                 "amount": "Amount",
                 "aid": "AssetID"
             },
             "withdraw":
             {
                 "cid": "ContractID",
                 "pkForeign": "PubKey",
                 "bCoSigner": "uint32_t",
                 "amount": "Amount",
                 "aid": "AssetID",
                 "amountCoSigner": "Amount"
             }
         }
     }
 }
```

What you see here is the API of the Vault contract as it is presented by the Application Shader we have used. The API actions are separated into roles, in this case 'manager' and 'my\_account'. Roles are only used as semantic grouping.

Let's use the 'view\_accounts' method to list all accounts in the Vault

Since we are going to be working with a specific Contract Shader we will need to use the contract id that we have copied from the blockchain explorer: \
`d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14`

Let's run the following command:

`beam-wallet-dappnet.exe shader --shader_app_file vault\app.wasm --shader_args=cid=d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14,role=manager,action=view_accounts`

The output will show us something similar to this:

<figure><img src=".gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

In this case there are currently no accounts in the contract. Let's create one using a 'deposit' method. Let's start with depositing 1 BEAM. To do that we will run the following command:

`beam-wallet-dappnet.exe shader --shader_app_file vault\app.wasm --shader_args="cid=d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14,role=my_account,action=deposit,amount=100000000"`

Note that amount is set in Groth, which is 1^10-8 of BEAM.&#x20;

This action will take some time to complete since the transaction is created and sent to the network. The result will be similar to this:

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

The important part is the one here which explains what happened

<figure><img src=".gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Let's run the 'accounts' command again and see if anything changed?

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Indeed, now we see that there is a new account identified by a public key and having a balance of 1 Beam. As an exercise you are invited tp do the same, and also get your BEAM back. You can try to get BEAM from someone elses account (hint: you can't).



In the next part of the tutorial we will see how to code this contract and deploy it on chain.&#x20;
