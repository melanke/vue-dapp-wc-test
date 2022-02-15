<template>
  <div class="hello">
    <button v-if="!isConnected" @click="connect">Connect</button>
    <button v-else @click="disconnect">Disconnect</button>

    <button @click="transferGas">Transfer Gas</button>
    <button @click="getStream">Get Stream</button>
    <button @click="withdrawStream">Withdraw Stream</button>
    <button @click="multiInvoke">Multi Invoke</button>
    <button @click="multiTestInvoke">Multi Test Invoke</button>
    <button @click="multiInvokeFailing">Multi Invoke Failing</button>
    <button @click="signAndVerify">Sign and Verify Message</button>
    <button @click="verifyFailling">Verify Failling</button>
    <button @click="verify">Verify Success</button>
    <button @click="ledgerTests">Ledger Tests</button>
  </div>
</template>

<script lang="ts">
import { Vue } from 'vue-class-component'
import { Argument, ContractInvocation, WcSdk, WitnessScope } from '@cityofzion/wallet-connect-sdk-core'

export default class HelloWorld extends Vue {
  wcSdk = new WcSdk()
  testnetKey = 'neo3:testnet'
  mainnetKey = 'neo3:mainnet'
  scripthash = '0xd2a4cff31913016155e38e474a2c06d08be276cf'

  get isConnected (): boolean {
    return !!this.wcSdk.session
  }

  async mounted (): Promise<void> {
    await this.wcSdk.initClient(
      'debug', // logger: use debug to show all log information on browser console
      'wss://relay.walletconnect.org' // relayServer: which relay server do you want to use, alternatively you can use "wss://relay.walletconnect.org"
    )

    this.wcSdk.subscribeToEvents({
      onProposal: (uri: string) => {
        window.open(`https://neon.coz.io/connect?uri=${uri}`, '_blank')?.focus()
      },
      onDeleted: () => {
        this.disconnect()
      }
    })

    await this.wcSdk.loadSession()

    console.log('session', this.wcSdk.session)
  }

  async connect (): Promise<void> {
    await this.wcSdk.connect({
      chains: [this.testnetKey, this.mainnetKey], // blockchain and network identifier
      methods: [
        'invokeFunction',
        'testInvoke',
        'signMessage',
        'verifyMessage'
      ], // which RPC methods do you plan to call
      appMetadata: {
        name: 'MyApplicationName', // your application name to be displayed on the wallet
        description: 'My Application description', // description to be shown on the wallet
        url: 'https://myapplicationdescription.app/', // url to be linked on the wallet
        icons: ['https://myapplicationdescription.app/myappicon.png'] // icon to be shown on the wallet
      }
    })

    alert(this.isConnected ? 'Connected successfully' : 'Connection refused')
  }

  async disconnect (): Promise<void> {
    await this.wcSdk.disconnect()
  }

  async ledgerTests (): Promise<void> {
    const resp = await this.wcSdk.invokeFunction({
      scriptHash: '0x13a83e059c2eedd5157b766d3357bc826810905e',
      operation: 'swapTokenInForTokenOut',
      args: [
        {
          type: 'Address',
          value: 'NiafyfWXhCH7XjUxybkaVePBvKBdvXur7J'
        },
        {
          type: 'Integer',
          value: '267240744'
        },
        {
          type: 'Integer',
          value: '99500000'
        },
        {
          type: 'Array',
          value: [
            {
              type: 'Hash160',
              value: '0xd2a4cff31913016155e38e474a2c06d08be276cf'
            },
            {
              type: 'Hash160',
              value: '0x48c40d4666f93408be1bef038b6722404d9a4c2a'
            }
          ]
        },
        {
          type: 'Integer',
          value: '1641239607592'
        }
      ]
    }, {
      scopes: 16,
      allowedContracts: [
        '0x13a83e059c2eedd5157b766d3357bc826810905e',
        '0x06f12a6aa2b5689ce97f16979b179fb3e31d63d7',
        '0x701f7fe4c8d325487b64d718419a2a5a4a5e38eb',
        '0xd2a4cff31913016155e38e474a2c06d08be276cf',
        '0x9f882cb625c7312f899bff70098f4929fc5b9245',
        '0x48c40d4666f93408be1bef038b6722404d9a4c2a'
      ]
    })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async transferGas (): Promise<void> {
    const senderAddress = this.wcSdk.getAccountAddress() ?? ''

    const from: Argument = { type: 'Address', value: senderAddress }
    const recipient: Argument = { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' }
    const value: Argument = { type: 'Integer', value: 100000000 }
    const args: Argument = { type: 'Array', value: [] }

    const resp = await this.wcSdk.invokeFunction({
      scriptHash: this.scripthash,
      operation: 'transfer',
      args: [from, recipient, value, args]
    }, { scopes: WitnessScope.CalledByEntry })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async getStream (): Promise<void> {
    const resp = await this.wcSdk.testInvoke({
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'getStream',
      args: [{ type: 'Integer', value: 17 }]
    }, { scopes: WitnessScope.None })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async withdrawStream (): Promise<void> {
    const streamId: Argument = { type: 'Integer', value: 17 }
    const value: Argument = { type: 'Integer', value: 1 }

    const resp = await this.wcSdk.invokeFunction({
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'withdraw',
      args: [streamId, value]
    }, { scopes: WitnessScope.CalledByEntry })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async multiInvoke (): Promise<void> {
    if (!this.wcSdk.session || !this.wcSdk) {
      window.alert('Not session or client')
      return
    }

    const senderAddress = this.wcSdk.getAccountAddress() ?? ''

    const req1: ContractInvocation = {
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'getStream',
      args: [{ type: 'Integer', value: 17 }]
    }

    const from: Argument = { type: 'Address', value: senderAddress }
    const recipient: Argument = { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' }
    const val: Argument = { type: 'Integer', value: 100000000 }
    const args: Argument = { type: 'Array', value: [] }

    const req2: ContractInvocation = {
      scriptHash: this.scripthash,
      operation: 'transfer',
      args: [from, recipient, val, args]
    }

    const resp = await this.wcSdk.invokeFunction(
      [req1, req2],
      { scopes: WitnessScope.Global, allowedContracts: ['0x010101c0775af568185025b0ce43cfaa9b990a2a'] }
    )

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async multiTestInvoke (): Promise<void> {
    if (!this.wcSdk.session || !this.wcSdk) {
      window.alert('Not session or client')
      return
    }

    const senderAddress = this.wcSdk.getAccountAddress() ?? ''

    const streamId: Argument = { type: 'Integer', value: 2 }
    const value: Argument = { type: 'Integer', value: 1 }

    const req1: ContractInvocation = {
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'withdraw',
      args: [streamId, value]
    }

    const from: Argument = { type: 'Address', value: senderAddress }
    const recipient: Argument = { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' }
    const val: Argument = { type: 'Integer', value: 100000000 }
    const args: Argument = { type: 'Array', value: [] }

    const req2: ContractInvocation = {
      scriptHash: this.scripthash,
      operation: 'transfer',
      args: [from, recipient, val, args]
    }

    const resp = await this.wcSdk.testInvoke(
      [req1, req2],
      { scopes: WitnessScope.CalledByEntry }
    )

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async multiInvokeFailing (): Promise<void> {
    if (!this.wcSdk.session || !this.wcSdk) {
      window.alert('Not session or client')
      return
    }

    const senderAddress = this.wcSdk.getAccountAddress() ?? ''

    const req1: ContractInvocation = {
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'verify',
      args: [],
      abortOnFail: true
    }

    const from: Argument = { type: 'Address', value: senderAddress }
    const recipient: Argument = { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' }
    const val: Argument = { type: 'Integer', value: 100000000 }
    const args: Argument = { type: 'Array', value: [] }

    const req2: ContractInvocation = {
      scriptHash: this.scripthash,
      operation: 'transfer',
      args: [from, recipient, val, args],
      abortOnFail: true
    }

    const resp = await this.wcSdk.invokeFunction(
      [req1, req2],
      { scopes: WitnessScope.CalledByEntry }
    )

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async signAndVerify (): Promise<void> {
    const resp = await this.wcSdk.signMessage('Caralho, muleq, o baguiu eh issumermo taix ligado na missão?')

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))

    const resp2 = await this.wcSdk.verifyMessage(resp.result)

    console.log(resp2)
    window.alert(JSON.stringify(resp2, null, 2))
  }

  async verifyFailling (): Promise<void> {
    const resp2 = await this.wcSdk.verifyMessage({
      data: '4fe1b478cf76564b2133bdff9ba97d8a360ce36d0511918931cda207c2ce589dfc07ec5d8b93ce7c3b70fc88b676cc9e08f9811bf0d5b5710a20f10c58191bfb',
      messageHex: '010001f05c3733336365623464346538666664633833656363366533356334343938393939436172616c686f2c206d756c65712c206f2062616775697520656820697373756d65726d6f2074616978206c696761646f206e61206d697373e36f3f0000',
      publicKey: '031757edb62014dea820a0b33a156f6a59fc12bd966202f0e49357c81f26f5de34',
      salt: '733ceb4d4e8ffdc83ecc6e35c4498999'
    })

    console.log(resp2)
    window.alert(JSON.stringify(resp2, null, 2))
  }

  async verify (): Promise<void> {
    const resp2 = await this.wcSdk.verifyMessage({
      publicKey: '031757edb62014dea820a0b33a156f6a59fc12bd966202f0e49357c81f26f5de34',
      data: 'aeb234ed1639e9fcc95a102633b1c70ca9f9b97e9592cc74bfc40cbc7fefdb19ae8c6b49ebd410dbcbeec6b5906e503d528e34cd5098cc7929dbcbbaf23c5d77',
      salt: '052a55a8d56b73b342a8e41da3050b09',
      messageHex: '010001f0a0303532613535613864353662373362333432613865343164613330353062303965794a68624763694f694a49557a49314e694973496e523563434936496b705856434a392e65794a6c654841694f6a45324e444d304e7a63324e6a4d73496d6c68644349364d5459304d7a4d354d5449324d33302e7253315f73735230364c426778744831504862774c306d7a6557563950686d5448477a324849524f4a4f340000'
    })

    console.log(resp2)
    window.alert(JSON.stringify(resp2, null, 2))
  }
}
</script>

<style scoped lang="scss">

</style>
