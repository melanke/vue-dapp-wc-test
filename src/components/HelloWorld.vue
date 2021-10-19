<template>
  <div class="hello">
    <button @click="initCli">Init CLI</button>
    <button @click="transferGas">Transfer Gas</button>
    <button @click="getStream">Get Stream</button>
    <button @click="withdrawStream">Withdraw Stream</button>
    <button @click="multiInvoke">Multi Invoke</button>
    <button @click="disconnect">Disconnect</button>
  </div>
</template>

<script lang="ts">
import { Vue } from 'vue-class-component'
import QRCodeModal from '@walletconnect/qrcode-modal'
import Client from '@walletconnect/client'
import { SessionTypes } from '@walletconnect/types'
import { WcSdk } from '@/components/WcSdk'

export default class HelloWorld extends Vue {
  wcClient: Client | null = null
  session: SessionTypes.Settled | null = null
  chainId = 'neo3:testnet'
  scripthash = '0xd2a4cff31913016155e38e474a2c06d08be276cf'

  async initCli (): Promise<void> {
    console.log('initCli')
    this.wcClient = await WcSdk.initClient(
      'debug', // logger: use debug to show all log information on browser console
      'wss://relay.walletconnect.org' // relayServer: which relay server do you want to use, alternatively you can use "wss://relay.walletconnect.org"
    )

    WcSdk.subscribeToEvents(this.wcClient, {
      onProposal: (uri: string) => {
        console.log('onProposal')
        QRCodeModal.open(uri, () => { /* nothing */ })
      }
    })

    this.session = await WcSdk.connect(this.wcClient, {
      chainId: this.chainId, // blockchain and network identifier
      methods: ['invokefunction', 'testInvoke', 'multiInvoke'], // which RPC methods do you plan to call
      appMetadata: {
        name: 'MyApplicationName', // your application name to be displayed on the wallet
        description: 'My Application description', // description to be shown on the wallet
        url: 'https://myapplicationdescription.app/', // url to be linked on the wallet
        icons: ['https://myapplicationdescription.app/myappicon.png'] // icon to be shown on the wallet
      }
    })
    console.log('session', this.session.state)
    QRCodeModal.close()
  }

  async transferGas (): Promise<void> {
    if (!this.session || !this.wcClient) {
      window.alert('Not session or client')
      return
    }

    console.log(this.session.state)

    const senderAddress = WcSdk.getAccountAddress(this.session)

    const from = { type: 'Address', value: senderAddress }
    const recipient = { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' }
    const value = { type: 'Integer', value: 100000000 }
    const args = { type: 'Array', value: [] }

    const resp = await WcSdk.invokeFunction(this.wcClient, this.session, this.chainId, {
      scriptHash: this.scripthash,
      operation: 'transfer',
      args: [from, recipient, value, args]
    })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async getStream (): Promise<void> {
    if (!this.session || !this.wcClient) {
      window.alert('Not session or client')
      return
    }
    const resp = await WcSdk.testInvoke(this.wcClient, this.session, this.chainId, {
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'getStream',
      args: [{ type: 'Integer', value: 2 }]
    })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async withdrawStream (): Promise<void> {
    if (!this.session || !this.wcClient) {
      window.alert('Not session or client')
      return
    }

    const streamId = { type: 'Integer', value: 2 }
    const value = { type: 'Integer', value: 1 }

    const resp = await WcSdk.invokeFunction(this.wcClient, this.session, this.chainId, {
      scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
      operation: 'withdraw',
      args: [streamId, value]
    })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async multiInvoke (): Promise<void> {
    if (!this.session || !this.wcClient) {
      window.alert('Not session or client')
      return
    }

    const senderAddress = WcSdk.getAccountAddress(this.session)

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

    const resp = await WcSdk.multiInvoke(this.wcClient, this.session, this.chainId, [req1, req2])

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async disconnect (): Promise<void> {
    if (!this.session || !this.wcClient) {
      window.alert('Not session or client')
      return
    }

    await WcSdk.disconnect(this.wcClient, this.session)
  }
}
</script>

<style scoped lang="scss">

</style>
