<template>
  <b-modal :id="'txInfoModal_' + transType + modalId"
           ref="infoModal"
           class="tx-modal"
           lazy
           centered
           hide-header
           hide-footer>
    <div class="md-content">
      <button
        class="close btn-close"
        @click="closeModal">
        <img src="../../../assets/imgs/icons/operate/ic_close.svg">
      </button>
      <div class="tx-title">
        <div class="tx-icon">
          <img v-if="txIcon==='sent'"
               src="../../../assets/imgs/icons/wallet/ic_sent.svg"
               width="60px"
               height="60px">
          <img v-else-if="txIcon==='received'"
               src="../../../assets/imgs/icons/wallet/ic_received.svg"
               width="60px"
               height="60px">
          <img v-else-if="txIcon==='leased in'"
               src="../../../assets/imgs/icons/wallet/ic_leasing_reverse.svg"
               width="60px"
               height="60px">
          <img v-else-if="txIcon==='leased out'"
               src="../../../assets/imgs/icons/wallet/ic_leasing.svg"
               width="60px"
               height="60px">
          <img v-else-if="txIcon==='leased out canceled'"
               src="../../../assets/imgs/icons/wallet/ic_leasing_cancel.svg"
               width="60px"
               height="60px">
          <img v-else-if="txIcon==='leased in canceled'"
               src="../../../assets/imgs/icons/wallet/ic_leasing_cancel_in.svg"
               width="60px"
               height="60px">
        </div>
        <div :class="txClass + '-amount'">{{ txIcon === 'sent' ? '-' : txIcon === 'received' ? '+' : '' }}{{ formatter(txAmount) }} VEE</div>
      </div>
      <div class="tx-address"
           v-if="!txIcon==='self'">
        <label>{{ (txIcon === 'received' || txIcon === 'leased in' || txIcon === 'leased in canceled') ? 'From' : 'To' }}</label>
        <span>{{ txAddress }}</span>
      </div>
      <div class="tx-block">
        <label>Timestamp</label>
        <span>{{ new Date(txTime / 1000000).toLocaleString() }}</span>
      </div>
      <div class="tx-fee">
        <label>Fee</label>
        <span>{{ formatter(txFee || 0) }} VEE</span>
      </div>
      <div class="tx-attachment"
           v-if="txIcon === 'sent' || txIcon === 'received'">
        <label>Attachment</label>
        <span>{{ txAttachment }}</span>
      </div>
      <div class="tx-attachment">
        <label>ID</label>
        <span>{{ modalId }}</span>
      </div>
      <div class="tx-attachment">
        <label>Block Height</label>
        <span>{{ txBlock }}</span>
      </div>
    </div>
  </b-modal>
</template>

<script>
import browser from '../../../utils/browser'

export default {
    name: 'TxInfoModal',
    props: {
        modalId: {
            type: String,
            default: ''
        },
        txIcon: {
            type: String,
            default: ''
        },
        txAddress: {
            type: String,
            default: ''
        },
        txAmount: {
            type: Number,
            default: 0
        },
        txFee: {
            type: Number,
            default: 0
        },
        txTime: {
            type: Number,
            default: 0
        },
        txAttachment: {
            type: String,
            default: ''
        },
        transType: {
            type: String,
            default: 'payment'
        },
        txBlock: {
            type: Number,
            default: 0
        }
    },
    computed: {
        txClass() {
            return this.txIcon.replace(/\s+/g, '')
        }
    },
    methods: {
        closeModal() {
            this.$refs.infoModal.hide()
        },
        formatter(num) {
            return browser.numberFormatter(num)
        }
    }
}
</script>

<style scoped lang="less">
.tx-modal {
    text-align: center;
    .tx-title {
        padding: 24px 0;
        border-bottom: 1px solid #1111;
        .tx-icon {
            margin-bottom: 10px;
        }
    }
    .tx-address {
        text-align: left;
        border-bottom: 1px solid #E8E9ED;
        height: 48px;
        padding-top: 15px;
        span {
            float:right;
            font-size: 15px;
            color: #4F515E;
            letter-spacing: 0;
            text-align: right;
        }
        label {
            font-size: 15px;
            color: #9091A3;
            letter-spacing: 0;
        }
    }
    .tx-id {
        text-align: left;
        border-bottom: 1px solid #E8E9ED;
        height: 48px;
        padding-top: 15px;
        span {
            float:right;
            font-size: 15px;
            color: #4F515E;
            letter-spacing: 0;
            text-align: right;
        }
        label {
            font-size: 15px;
            color: #9091A3;
            letter-spacing: 0;
        }
    }
    .tx-block {
        text-align: left;
        border-bottom: 1px solid #E8E9ED;
        height: 48px;
        padding-top: 15px;
        span {
            float:right;
            font-size: 15px;
            color: #4F515E;
            letter-spacing: 0;
            text-align: right;
        }
        label {
            font-size: 15px;
            color: #9091A3;
            letter-spacing: 0;
        }
    }
    .tx-fee {
        text-align: left;
        border-bottom: 1px solid #E8E9ED;
        height: 48px;
        padding-top: 15px;
        span {
            float:right;
            font-size: 15px;
            color: #4F515E;
            letter-spacing: 0;
            text-align: right;
        }
        label {
            font-size: 15px;
            color: #9091A3;
            letter-spacing: 0;
        }
    }
    .tx-attachment {
        text-align: left;
        border-bottom: 1px solid #E8E9ED;
        height: 48px;
        padding-top: 15px;
        span {
            float:right;
            font-size: 15px;
            color: #4F515E;
            letter-spacing: 0;
            text-align: right;
        }
        label {
          font-size: 15px;
          color: #9091A3;
          letter-spacing: 0;
        }
    }
    .sent-amount {
        font-size: 28px;
        color: #F5354B;
        letter-spacing: 0;
        text-align: center;
    }
    .received-amount {
        font-size: 28px;
        color: #23A28C;
        letter-spacing: 0;
        text-align: center;
    }
    .leasedin-amount {
        font-size: 28px;
        color: #86BEF7;
        letter-spacing: 0;
        text-align: center;
    }
    .leasedout-amount {
        font-size: 28px;
        color: #73CC5A;
        letter-spacing: 0;
        text-align: center;
    }
    .leasedoutcanceled-amount {
        font-size: 28px;
        color: #FF7A8A;
        letter-spacing: 0;
        text-align: center;
    }
    .leasedincanceled-amount {
        font-size: 28px;
        color: #FFD192;
        letter-spacing: 0;
        text-align: center;
    }
    .md-content {
        padding: 40px 24px;
        padding-top: 0;
    }
}
.btn-close {
    position: absolute;
    right: 0;
    margin-right: 20px;
    margin-top: 4px;
}
</style>
