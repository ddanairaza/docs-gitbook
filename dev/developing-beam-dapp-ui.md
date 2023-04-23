# Developing Beam DApp UI

Most of Beam Decentralized Applications include a front end component which can run within the Beam Wallet or as a standalone website that connects to Beam Web Wallet.

These front end components are usually implemented using standard Web Frameworks such as React or Vue (or any other). The communication with the smart contract that is deployed on chain is handled through the Application Shader, which is provided as a Web Assembly component embedded within the web application.

In this tutorial we will review how to set up the development environment for building Beam DApps using the Desktop Wallet

### Configure the 'testapp' in the wallet

1. Download and install [Beam Desktop wallet](https://beam.mw/downloads/dappnet)\
   \
   We recommend using DAPPNET (the testing network created especially for testing dapps), as it is the most convenient for this purpose. You will also need to get some DAPPNET Beam to pay transaction fees, which can be done using the Faucet application you can find in the built in DApp Store.\

2. Find the location of your Beam Wallet data folder\
   \
   In the settings screen of the wallet, open the 'Troubleshooting' section. It will show you the location of your wallet folder on your platform



<figure><img src=".gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

3\. Open the folder and locate the settings.ini file

Edit the file and add the following section:

```
[devapp]
name=testapp
url=http://127.0.0.1:13666/index.html
```

Restart the Desktop Wallet. Now in the DApp Store screen you will see the placeholder for the test app.

<figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

### Running the application

For this section we assume that contract is already deployed on the blockchain and that application shader assembly is present. As an example we will use the BANS application that can be seen here: [https://github.com/BeamMW/bans](https://github.com/BeamMW/bans)

In order to run this application you will need the following pre-requisutes:

1. [nodejs ](https://nodejs.org/en/)for your platform
2. yarn (can be installed with 'npm install --global yarn', once you install nodejs)

Now change directory to where you have cloned the BANS repository and run:

```
yarn install
yarn start
```

If any questions are asked during installation just chose default answers this should work.

Once the local development server is running, you can launch the 'testapp' in the wallet. This will show you the BANS screen

<figure><img src=".gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

### Using the Developer Console

During application development, it is useful to be able to see the developer console, as you would in a regular browser. Beam Desktop Wallet provides this option when launched with a flag:

```
--remote-debugging-port=20000
```

After you relaunch the wallet with this flag, you can open a regular web browser and navigate to: [http://localhost:20000/](http://localhost:20000/)

