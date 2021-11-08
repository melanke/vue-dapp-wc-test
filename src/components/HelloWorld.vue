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
  </div>
</template>

<script lang="ts">
import { Vue } from 'vue-class-component'
import { ContractInvocation, WcSdk, WitnessScope } from '@/components/WcSdk'
import { SessionTypes } from '@walletconnect/types/dist/cjs'
import QRCodeModal from '@walletconnect/qrcode-modal'

export default class HelloWorld extends Vue {
  wcSdk = new WcSdk()
  testnetKey = 'neo3:testnet'
  mainnetKey = 'neo3:mainnet'
  scripthash = '0xd2a4cff31913016155e38e474a2c06d08be276cf'

  get isConnected (): SessionTypes.Settled | undefined {
    return this.wcSdk.session
  }

  async mounted (): Promise<void> {
    await this.wcSdk.initClient(
      'debug', // logger: use debug to show all log information on browser console
      'wss://relay.walletconnect.org' // relayServer: which relay server do you want to use, alternatively you can use "wss://relay.walletconnect.org"
    )

    this.wcSdk.subscribeToEvents({
      onProposal: (uri: string) => {
        QRCodeModal.open(uri, () => { /* nothing */ })
        // window.open(`https://neon-dev.coz.io/?uri=${uri}`, '_blank')?.focus()
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
      methods: ['invokefunction', 'testInvoke', 'multiInvoke', 'multiTestInvoke'], // which RPC methods do you plan to call
      appMetadata: {
        name: 'MyApplicationName', // your application name to be displayed on the wallet
        description: 'My Application description', // description to be shown on the wallet
        url: 'https://myapplicationdescription.app/', // url to be linked on the wallet
        icons: ['https://myapplicationdescription.app/myappicon.png'] // icon to be shown on the wallet
      }
    })
    QRCodeModal.close()

    alert(this.isConnected ? 'Connected successfully' : 'Connection refused')
  }

  async transferGas (): Promise<void> {
    const senderAddress = this.wcSdk.getAccountAddress()

    const from = { type: 'Address', value: senderAddress }
    const recipient = { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' }
    const value = { type: 'Integer', value: 100000000 }
    const args = { type: 'Array', value: [] }

    const resp = await this.wcSdk.invokeFunction({
      scriptHash: this.scripthash,
      operation: 'transfer',
      args: [from, recipient, value, args]
    })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async getStream (): Promise<void> {
    const resp = await this.wcSdk.testInvoke({
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'getStream',
      args: [{ type: 'Integer', value: 17 }],
      signer: { scopes: WitnessScope.None }
    })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async withdrawStream (): Promise<void> {
    const streamId = { type: 'Integer', value: 17 }
    const value = { type: 'Integer', value: 1 }

    const resp = await this.wcSdk.invokeFunction({
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'withdraw',
      args: [streamId, value]
    })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async multiInvoke (): Promise<void> {
    if (!this.wcSdk.session || !this.wcSdk) {
      window.alert('Not session or client')
      return
    }

    const senderAddress = this.wcSdk.getAccountAddress()

    const req1: ContractInvocation = {
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'getStream',
      args: [{ type: 'Integer', value: 17 }]
    }

    const from = { type: 'Address', value: senderAddress }
    const recipient = { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' }
    const val = { type: 'Integer', value: 100000000 }
    const args = { type: 'Array', value: [] }

    const req2: ContractInvocation = {
      scriptHash: this.scripthash,
      operation: 'transfer',
      args: [from, recipient, val, args],
      signer: { scopes: WitnessScope.Global, allowedContracts: ['0x010101c0775af568185025b0ce43cfaa9b990a2a'] }
    }

    const resp = await this.wcSdk.multiInvoke([req1, req2])

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async multiTestInvoke (): Promise<void> {
    if (!this.wcSdk.session || !this.wcSdk) {
      window.alert('Not session or client')
      return
    }

    const senderAddress = this.wcSdk.getAccountAddress()

    const streamId = { type: 'Integer', value: 2 }
    const value = { type: 'Integer', value: 1 }

    const req1 = {
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'withdraw',
      args: [streamId, value]
    }

    const from = { type: 'Address', value: senderAddress }
    const recipient = { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' }
    const val = { type: 'Integer', value: 100000000 }
    const args = { type: 'Array', value: [] }

    const req2 = {
      scriptHash: this.scripthash,
      operation: 'transfer',
      args: [from, recipient, val, args]
    }

    const resp = await this.wcSdk.multiTestInvoke([req1, req2])

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async multiInvokeFailing (): Promise<void> {
    if (!this.wcSdk.session || !this.wcSdk) {
      window.alert('Not session or client')
      return
    }

    const senderAddress = this.wcSdk.getAccountAddress()

    const req1 = {
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'verify',
      args: [],
      abortOnFail: true
    }

    const from = { type: 'Address', value: senderAddress }
    const recipient = { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' }
    const val = { type: 'Integer', value: 100000000 }
    const args = { type: 'Array', value: [] }

    const req2 = {
      scriptHash: this.scripthash,
      operation: 'transfer',
      args: [from, recipient, val, args],
      abortOnFail: true
    }

    const resp = await this.wcSdk.multiInvoke([req1, req2])

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async disconnect (): Promise<void> {
    await this.wcSdk.disconnect()
  }
}
</script>

<style scoped lang="scss">

</style>
