<template>
  <div class="hello">
    <span v-if="!wcSdk">Loading...</span>
    <template v-else>
      <button v-if="!wcSdk?.isConnected()" @click="wcSdk?.connect">Connect</button>
      <button v-else @click="wcSdk.disconnect">Disconnect</button>

      <button @click="transferGas">Transfer Gas</button>
      <button @click="multiInvoke">Multi Invoke</button>
      <button @click="multiTestInvoke">Multi Test Invoke</button>
      <button @click="multiInvokeFailing">Multi Invoke Failing</button>
      <button @click="signAndVerify">Sign and Verify Message</button>
      <button @click="verifyFailling">Verify Failling</button>
      <button @click="verify">Verify Success</button>
    </template>
  </div>
</template>

<script lang="ts">
import { Vue } from 'vue-class-component'
import SignClient from '@walletconnect/sign-client'
import WcSdk, { WitnessScope } from '@cityofzion/wallet-connect-sdk-core'

export default class HelloWorld extends Vue {
  wcSdk: WcSdk | null = null
  scripthash = '0xd2a4cff31913016155e38e474a2c06d08be276cf'

  async mounted (): Promise<void> {
    this.wcSdk = new WcSdk(await SignClient.init({
      projectId: 'a9ff54e3d56a52230ed8767db4d4a810',
      relayUrl: 'wss://relay.walletconnect.com',
      metadata: {
        name: 'MyApplicationName', // your application name to be displayed on the wallet
        description: 'My Application description', // description to be shown on the wallet
        url: 'https://myapplicationdescription.app/', // url to be linked on the wallet
        icons: ['https://myapplicationdescription.app/myappicon.png'] // icon to be shown on the wallet
      }
    }))

    await this.wcSdk.manageSession()
  }

  async transferGas (): Promise<void> {
    const resp = await this.wcSdk?.invokeFunction({
      invocations: [{
        scriptHash: this.scripthash,
        operation: 'transfer',
        args: [
          { type: 'Address', value: this.wcSdk?.getAccountAddress() ?? '' },
          { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' },
          { type: 'Integer', value: 100000000 },
          { type: 'Array', value: [] }
        ]
      }],
      signers: [{ scopes: 1 }]
    })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async multiInvoke (): Promise<void> {
    const resp = await this.wcSdk?.invokeFunction({
      invocations: [
        {
          scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
          operation: 'getStream',
          args: [{ type: 'Integer', value: 17 }]
        }, {
          scriptHash: this.scripthash,
          operation: 'transfer',
          args: [
            { type: 'Address', value: this.wcSdk?.getAccountAddress() ?? '' },
            { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' },
            { type: 'Integer', value: 100000000 },
            { type: 'Array', value: [] }
          ]
        }
      ],
      signers: [{ scopes: WitnessScope.Global, allowedContracts: ['0x010101c0775af568185025b0ce43cfaa9b990a2a'] }]
    }
    )

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async multiTestInvoke (): Promise<void> {
    const resp = await this.wcSdk?.testInvoke({
      invocations: [
        {
          scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
          operation: 'withdraw',
          args: [
            { type: 'Integer', value: 2 },
            { type: 'Integer', value: 1 }
          ]
        },
        {
          scriptHash: this.scripthash,
          operation: 'transfer',
          args: [
            { type: 'Address', value: this.wcSdk?.getAccountAddress() ?? '' },
            { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' },
            { type: 'Integer', value: 100000000 },
            { type: 'Array', value: [] }
          ]
        }
      ],
      signers: [{ scopes: WitnessScope.CalledByEntry }]
    })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async multiInvokeFailing (): Promise<void> {
    const resp = await this.wcSdk?.invokeFunction({
      invocations: [
        {
          scriptHash: '0x010101c0775af568185025b0ce43cfaa9b990a2a',
          operation: 'verify',
          args: [],
          abortOnFail: true
        },
        {
          scriptHash: this.scripthash,
          operation: 'transfer',
          args: [
            { type: 'Address', value: this.wcSdk?.getAccountAddress() ?? '' },
            { type: 'Address', value: 'NbnjKGMBJzJ6j5PHeYhjJDaQ5Vy5UYu4Fv' },
            { type: 'Integer', value: 100000000 },
            { type: 'Array', value: [] }
          ],
          abortOnFail: true
        }],
      signers: [{ scopes: WitnessScope.CalledByEntry }]
    })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))
  }

  async signAndVerify (): Promise<void> {
    if (!this.wcSdk) return
    const resp = await this.wcSdk.signMessage({ message: 'Caralho, muleq, o baguiu eh issumermo taix ligado na missão?', version: 2 })

    console.log(resp)
    window.alert(JSON.stringify(resp, null, 2))

    const resp2 = await this.wcSdk?.verifyMessage(resp)

    console.log(resp2)
    window.alert(JSON.stringify(resp2, null, 2))
  }

  async verifyFailling (): Promise<void> {
    const resp2 = await this.wcSdk?.verifyMessage({
      data: '4fe1b478cf76564b2133bdff9ba97d8a360ce36d0511918931cda207c2ce589dfc07ec5d8b93ce7c3b70fc88b676cc9e08f9811bf0d5b5710a20f10c58191bfb',
      messageHex: '010001f05c3733336365623464346538666664633833656363366533356334343938393939436172616c686f2c206d756c65712c206f2062616775697520656820697373756d65726d6f2074616978206c696761646f206e61206d697373e36f3f0000',
      publicKey: '031757edb62014dea820a0b33a156f6a59fc12bd966202f0e49357c81f26f5de34',
      salt: '733ceb4d4e8ffdc83ecc6e35c4498999'
    })

    console.log(resp2)
    window.alert(JSON.stringify(resp2, null, 2))
  }

  async verify (): Promise<void> {
    const resp2 = await this.wcSdk?.verifyMessage({
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
