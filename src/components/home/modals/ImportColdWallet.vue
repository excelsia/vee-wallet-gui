<template>
  <b-modal id="importModal"
           centered
           lazy
           @cancel="importCancel"
           @hide="importClose"
           @ok="importOk"
           @show="showingUp"
           :ok-disabled="!coldAddress || !isValidAddress"
           :cancel-disabled="qrInit"
           title="Import Cold Wallet"
           hide-header
           hide-footer
           ref="importModal">
    <button
      :disabled="qrInit"
      class="close btn-close"
      @click="closeModal">
      <img src="../../../assets/imgs/icons/operate/ic_close.svg">
    </button>
    <b-container fluid
                 class="c-import">
      <b-form-group label="Cold Wallet Address"
                    label-for="coldAddress">
        <b-form-input id="coldAddress"
                      type="text"
                      v-model="coldAddress"
                      :state="isValidAddress"
                      aria-describedby="inputLiveHelp inputLiveFeedback"
                      placeholder="Please input cold wallet address.">
        </b-form-input>
        <b-form-invalid-feedback id="inputLiveFeedback">
          Invalid cold wallet address. <span v-if="addressExisted">The address has existed.</span>
        </b-form-invalid-feedback>
      </b-form-group>
      <b-form-group v-if="version === '0.2.0'"
                    label="Cold Wallet Public Key"
                    label-for="coldPubKey">
        <b-form-input id="coldPubKey"
                      type="text"
                      v-model="coldPubKey"
                      :state="isValidPubKey"
                      aria-describedby="inputLiveHelp inputLiveFeedback"
                      placeholder="Please input public key of cold wallet.">
        </b-form-input>
        <b-form-invalid-feedback id="inputLiveFeedback">Invalid cold wallet public key.
        </b-form-invalid-feedback>
      </b-form-group>
      <b-form-group label="Cold Wallet Version"
                    label-for="version">
        <b-form-select id=version
                       v-model="version"
                       :options="options()"
                       :state="iscurversion"
                       aria-describedby="inputLiveHelp inputLiveFeedback"></b-form-select>
        <b-form-valid-feedback id="inputLiveFeedback">You can find it in Settings -> About Us in your ColdWallet Application</b-form-valid-feedback>
        <b-form-invalid-feedback id="inputLiveFeedback">Your coldwallet is not the lastest version ,please update it!
        </b-form-invalid-feedback>
      </b-form-group>
      <p class="qrInfo">Tips: Please confirm if your browser's camera is available.</p>
      <div class="scan-pane">
        <qrcode-reader @init="onInit"
                       @decode="onDecode"
                       :paused="paused">
          <img v-if="qrInit"
               class="qrcode-waiting"
               height="100"
               width="100"
               src="../../../assets/imgs/icons/wallet/ic_wait.svg">
        </qrcode-reader>
        <b-btn class="scan-again-btn"
               variant="warning"
               @click="scanAgain"
               centered>Scan again</b-btn>
      </div>
    </b-container>
    <b-row class="btn-bottom">
      <b-col class="col-lef">
        <b-button
          class="btn-back"
          block
          variant="light"
          size="lg"
          @click="closeModal">Cancel
        </b-button>
      </b-col>
      <b-col class="col-rit">
        <b-button
          class="btn-confirm"
          :style="{background:(isValidAddress ? '' : '#FFBE96')}"
          variant="warning"
          size="lg"
          block
          :disabled="isConfirm"
          @click="importOk">Confirm
        </b-button>
      </b-col>
    </b-row>
  </b-modal>
</template>

<script>
import crypto from '@/utils/crypto'
import { PUBLIC_KEY_LENGTH } from '@/constants.js'
export default {
    name: 'ImportColdWallet',
    props: {
        address: {
            type: String,
            default: '',
            require: true
        }
    },
    data: function() {
        return {
            qrInit: false,
            paused: false,
            coldAddress: '',
            coldPubKey: '',
            version: ''
        }
    },
    computed: {
        isValidAddress: function() {
            if (this.coldAddress === this.address) {
                return false
            }
            if (!this.coldAddress) {
                return void 0
            }
            let isValid = false
            try {
                isValid = crypto.isValidAddress(this.coldAddress)
            } catch (e) {
                console.log(e)
            }
            return isValid
        },
        isValidPubKey: function() {
            if (!this.coldPubKey) {
                return void 0
            }
            return this.coldPubKey.length <= PUBLIC_KEY_LENGTH
        },
        iscurversion: function() {
            if (!this.version) {
                return void 0
            }
            return this.version === '0.3.0'
        },
        isConfirm() {
            return !((this.coldAddress && this.isValidAddress && this.version === '0.3.0') || (this.coldAddress && this.isValidPubKey && this.isValidAddress && this.version === '0.2.0'))
        },
        addressExisted: function() {
            return this.coldAddress === this.address
        }
    },
    methods: {
        showingUp() {
            this.coldAddress = ''
            this.version = '0.3.0'
            this.coldPubKey = ''
        },
        importClose: function(evt) {
            if (this.qrInit) {
                evt.preventDefault()
            }
        },
        options() {
            return ['0.3.0', '0.2.0']
        },
        importOk: function(evt) {
            if (this.qrInit || !this.coldAddress || !this.isValidAddress) {
                evt.preventDefault()
            } else {
                this.$emit('import-cold', this.coldAddress, this.coldPubKey)
                this.closeModal()
            }
        },
        importCancel: function(evt) {
            if (this.qrInit) {
                evt.preventDefault()
            }
        },
        async onInit(promise) {
            try {
                this.qrInit = true
                await promise
            } catch (error) {
                if (error.name === 'NotAllowedError') {
                    throw Error('user denied camera access permission')
                } else if (error.name === 'NotFoundError') {
                    throw Error('no suitable camera device installed')
                } else if (error.name === 'NotSupportedError') {
                    throw Error('page is not served over HTTPS (or localhost)')
                } else if (error.name === 'NotReadableError') {
                    throw Error('maybe camera is already in use')
                } else if (error.name === 'OverconstarinedError') {
                    throw Error('pass constraints do not match any camera')
                } else {
                    throw Error('browser is probably lacking features(WebRTC, Canvas)')
                }
            } finally {
                this.qrInit = false
            }
        },
        onDecode: function(decodeString) {
            let start = decodeString.indexOf('address')
            if (start === -1) {
                this.scanAgain()
            }
            this.paused = true
            let isNewVersion = decodeString.indexOf('version')
            if (isNewVersion !== -1) {
                let end = decodeString.indexOf('$')
                this.coldAddress = decodeString.substring(start + 8, end)
                start = decodeString.indexOf('version')
                end = decodeString.length
                this.version = decodeString.substring(start + 8, end)
            } else {
                let end = decodeString.indexOf('&')
                this.coldAddress = decodeString.substring(start + 8, end)
                start = decodeString.indexOf('publicKey')
                end = decodeString.length
                this.coldPubKey = decodeString.substring(start + 10, end)
                this.version = '0.2.0'
            }
        },
        scanAgain: function() {
            this.paused = false
            this.coldAddress = ''
            this.coldPubKey = ''
            this.version = '0.3.0'
        },
        closeModal: function() {
            this.$refs.importModal.hide()
        }
    }
}
</script>

<style scoped>
.scan-pane {
    width: 320px;
    margin: auto;
}
.qrcode-waiting {
    margin-left: 100px;
}
.qrInfo {
    margin-bottom: 20px;
    color: #9091a3;
}
.scan-again-btn {
    margin-top: 10px;
    background: rgba(255,135,55,0.00);
    border: 1px solid #FF8737;
    border-radius: 4px;
    font-size: 13px;
    color: #FF8737;
    letter-spacing: 0;
    width: 100px;
    margin-left: 110px;
}
.btn-close {
    position: absolute;
    right: 0;
    margin-right: 20px;
    margin-top: 4px;
}
.c-import {
    padding-top: 24px;
    margin-bottom: 40px;
    padding-left: 15px;
    padding-right: 15px;
}
.btn-bottom {
    margin-left: 0px;
    margin-right: 0px;
}
.btn-confirm {
    height: 44px;
    font-size: 17px;
    color: #FFFFFF;
    letter-spacing: 0;
    text-align: center;
}
.btn-back {
    background: #FAFAFA;
    border: 1px solid #E8E9ED;
    border-radius: 4px;
    font-size: 17px;
    color: #4F515E;
    letter-spacing: 0;
    text-align: center;
}
.col-lef {
    padding-right: 10px;
}
.col-rit {
    padding-left: 10px;
}
</style>
