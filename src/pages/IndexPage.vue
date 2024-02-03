<template>
  <q-page class="row items-center justify-evenly">
    <example-component
      title="Example component"
      active
      :todos="todos"
      :meta="meta"
    ></example-component>
  </q-page>
</template>

<script lang="ts">
import { Todo, Meta } from '../components/models';
import ExampleComponent from 'components/ExampleComponent.vue';
import { defineComponent, ref, onMounted } from 'vue';
import { Web3Auth } from '@web3auth/modal';
import { CHAIN_NAMESPACES, IProvider } from '@web3auth/base';
import Web3 from 'web3';

export default defineComponent({
  name: 'IndexPage',
  components: { ExampleComponent },
  setup() {
    const todos = ref<Todo[]>([
      {
        id: 1,
        content: 'ct1',
      },
      {
        id: 2,
        content: 'ct2',
      },
      {
        id: 3,
        content: 'ct3',
      },
      {
        id: 4,
        content: 'ct4',
      },
      {
        id: 5,
        content: 'ct5',
      },
    ]);
    const meta = ref<Meta>({
      totalCount: 1200,
    });

    const loggedIn = ref<boolean>(false);
    let provider = <IProvider | null>null;

    // IMP START - SDK Initialization
    // IMP START - Dashboard Registration
    const clientId =
      'BPi5PB_UiIZ-cPz1GtV5i1I2iOSOHuimiXBI0e-Oe_u6X3oVAbCiAZOTEBtTXw4tsluTITPqA8zMsfxIKMjiqNQ'; // get from https://dashboard.web3auth.io
    // IMP END - Dashboard Registration

    const chainConfig = {
      chainNamespace: CHAIN_NAMESPACES.EIP155,
      chainId: '0x1', // Please use 0x1 for Mainnet
      rpcTarget: 'https://rpc.ankr.com/eth',
      displayName: 'Ethereum Mainnet',
      blockExplorer: 'https://etherscan.io/',
      ticker: 'ETH',
      tickerName: 'Ethereum',
    };

    const web3auth = new Web3Auth({
      clientId,
      chainConfig,
      web3AuthNetwork: 'sapphire_mainnet',
    });
    // IMP END - SDK Initialization

    onMounted(async () => {
      const init = async () => {
        try {
          // IMP START - SDK Initialization
          await web3auth.initModal();
          // IMP END - SDK Initialization
          provider = web3auth.provider;

          if (web3auth.connected) {
            loggedIn.value = true;
          }
        } catch (error) {
          console.error(error);
        }
      };

      init();
    });

    const login = async () => {
      // IMP START - Login
      provider = await web3auth.connect();
      // IMP END - Login
      if (web3auth.connected) {
        loggedIn.value = true;
      }
    };

    const getUserInfo = async () => {
      // IMP START - Get User Information
      const user = await web3auth.getUserInfo();
      // IMP END - Get User Information
      uiConsole(user);
    };

    const logout = async () => {
      // IMP START - Logout
      await web3auth.logout();
      // IMP END - Logout
      provider = null;
      loggedIn.value = false;
      uiConsole('logged out');
    };

    // IMP START - Blockchain Calls
    const getAccounts = async () => {
      if (!provider) {
        uiConsole('provider not initialized yet');
        return;
      }
      const web3 = new Web3(provider as any);

      // Get user's Ethereum public address
      const address = await web3.eth.getAccounts();
      uiConsole(address);
    };

    const getBalance = async () => {
      if (!provider) {
        uiConsole('provider not initialized yet');
        return;
      }
      const web3 = new Web3(provider as any);

      // Get user's Ethereum public address
      const address = (await web3.eth.getAccounts())[0];

      // Get user's balance in ether
      const balance = web3.utils.fromWei(
        await web3.eth.getBalance(address), // Balance is in wei
        'ether'
      );
      uiConsole(balance);
    };

    const signMessage = async () => {
      if (!provider) {
        uiConsole('provider not initialized yet');
        return;
      }
      const web3 = new Web3(provider as any);

      // Get user's Ethereum public address
      const fromAddress = (await web3.eth.getAccounts())[0];

      const originalMessage = 'YOUR_MESSAGE';

      // Sign the message
      const signedMessage = await web3.eth.personal.sign(
        originalMessage,
        fromAddress,
        'test password!' // configure your own password here.
      );
      uiConsole(signedMessage);
    };
    // IMP END - Blockchain Calls

    function uiConsole(...args: any[]): void {
      const el = document.querySelector('#console>p');
      if (el) {
        el.innerHTML = JSON.stringify(args || {}, null, 2);
      }
      console.log(...args);
    }

    return {
      todos,
      meta,
      loggedIn,
      provider,
      web3auth,
      login,
      logout,
      getUserInfo,
      getAccounts,
      getBalance,
      signMessage,
    };
  },
});
</script>
