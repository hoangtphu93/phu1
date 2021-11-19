# phu1

1
import React from 'react';
2
import ReactDOM from 'react-dom';
3
import App from './App';
4
import getConfig from './config.js';
5
import * as nearAPI from 'near-api-js';
6
// Initializing contract
7
async function initContract() {
8
  const nearConfig = getConfig(process.env.NODE_ENV || 'testnet');
9
  // Initializing connection to the NEAR TestNet
10
  const near = await nearAPI.connect({
11
    deps: {
12
      keyStore: new nearAPI.keyStores.BrowserLocalStorageKeyStore()
13
    },
14
    ...nearConfig
15
  });
16
  // Needed to access wallet
17
  const walletConnection = new nearAPI.WalletConnection(near);
18
  // Load in account data
19
  let currentUser;
20
  if(walletConnection.getAccountId()) {
21
    currentUser = {
22
      accountId: walletConnection.getAccountId(),
23
      balance: (await walletConnection.account().state()).amount
24
    };
25
  }
26
  // Initializing our contract APIs by contract name and configuration
27
  const contract = await new nearAPI.Contract(walletConnection.account(), nearConfig.contractName, {
28
    // View methods are read-only – they don't modify the state, but usually return some value
29
    viewMethods: ['getMessages'],
30
    // Change methods can modify the state, but you don't receive the returned value when called
31
    changeMethods: ['addMessage'],
32
    // Sender is the account ID to initialize transactions.
33
    // getAccountId() will return empty string if user is still unauthorized
34
    sender: walletConnection.getAccountId()
35
  });
36
  return { contract, currentUser, nearConfig, walletConnection };
37
}
38
window.nearInitPromise = initContract()
39
  .then(({ contract, currentUser, nearConfig, walletConnection }) =&gt; {
40
    ReactDOM.render(
41
      &lt;App
42
        contract={contract}
43
        currentUser={currentUser}
44
        nearConfig={nearConfig}
45
        wallet={walletConnection}
46
      /&gt;,
47
      document.getElementById('root')
48
    );
49
  });
