<template>
  <div class="home">

    <!--new account-->
    <div class="new_account">
      <b-button variant="success" @click="newAccount">newAccount</b-button>
      <div>
        {{this.newAccountInfo}}
      </div>
    </div>

    <!--import account-->
    <div class="import_account">
      <b-form @submit="onSubmit">
        <b-form-group label="key:" label-for="keyInput" description="We'll never share your key with anyone else.">
          <b-form-input id="keyInput" type="text" v-model="importKeyForm.key" required
                        placeholder="Please enter your private key"/>
        </b-form-group>
        <b-button type="submit" variant="primary">importAccount</b-button>
      </b-form>
      <div>
        {{this.importAccountInfo}}
      </div>
    </div>

    <!-- transfer-->
    <div class="transfer">
      <b-button type="success" variant="primary" @click="transfers">transfer</b-button>
    </div>

  </div>
</template>

<script>
  import nuls from 'nuls-sdk-js';
  import utils from 'nuls-sdk-js/lib/utils/utils';
  //const nuls = require('nuls-sdk-js');

  export default {
    data() {
      return {
        newAccountInfo: '',

        importKeyForm: {
          key: '',
        },
        importAccountInfo: '',
      }
    },
    components: {},
    mounted() {

    },
    methods: {
      newAccount() {
        let passWord = "";
        this.newAccountInfo = nuls.newAddress(passWord);
      },

      /**
       * Import private key
       *  导入的私钥生成的地址和以前的地址不一样  The address generated by the imported private key is different from the previous address
       * @param evt
       */
      onSubmit() {
        //console.log(evt);
        //导入地址
        const key = this.importKeyForm.key;
        this.importAccountInfo = nuls.importByKey(key);
      },

      transfers() {

        let pri = '407d5cd9b5d62ab633c52dfb45542622b06c05004a0314c312390a32b5d06234';
        let pub = '032dd7aaff8d2c3ae6597877b67f87702f44f5998b3da4459ddeb6eec8d39171c9';
        let fromAddress = 'TTaqFxuD1xc6gpixUiMVQsjMZ5fdYJ2o';
        let toAddress = 'TTakMrubBXi998CZgaYdTy2Nrqwd2ptq';
        let amount = 8880000; //8
        let remark = 'transfer test';

        //转账功能 trustUrl
        async function transfer(pri, pub, fromAddress, toAddress, amount, remark) {
          const inputUtxoInfo = await nuls.getInputUtxo(fromAddress, amount);
          let inputOwner = [];
          let totalValue = 0;
          let fee = 0;
          //判断是否零钱过多
          if (inputUtxoInfo.length >= 6000) {
            return {success: false, data: "Too much change to consume"}
          } else {
            //计算手续费 （124 + 50  * inputs.length + 38 * outputs.length + remark.bytes.length ）/1024
            fee = Math.ceil((124 + 50 * inputUtxoInfo.length + 38 * 2 + +utils.stringToByte(remark).length) / 1024) * 100000;
          }
          //计算转账金额需要的inputUtxo
          for (let item of inputUtxoInfo) {
            totalValue = totalValue + item.value;
            inputOwner.push({owner: item.owner, na: item.value, lockTime: item.lockTime});
          }
          let outputOwner = [
            {owner: toAddress, na: amount, lockTime: 0}
          ];
          //计算多余的金额并返回
          if (totalValue - amount > 0) {
            outputOwner.push({owner: fromAddress, na: totalValue - amount - fee, lockTime: 0})
          }
          let hashOrSignature = nuls.transferTransaction(pri, pub, inputOwner, outputOwner, remark);
          //验证交易
          let valiTransactions = await nuls.valiTransaction(hashOrSignature.signature);
          //验证交易成功
          if (valiTransactions.data.success) {
            //广播交易
            const broadcastInfo = await nuls.broadcast(hashOrSignature.signature);
            console.log(broadcastInfo.data)
            //return broadcastInfo.data
          } else {
            return {success: false, data: "verify transaction failure"}
          }
        }

        //测试开始
        transfer(pri, pub, fromAddress, toAddress, amount, remark)
        /* transfer(pri, pub, fromAddress, toAddress, amount, remark).then((response) => {
           console.log(response)
         }).catch((error) => {
           console.log(error)
         });*/

      },

    }
  }
</script>

<style lang="less">
  .home {
    width: 1024px;
    margin: 0 auto;
    .new_account {
      border: 1px solid #c1c1c1;
      margin: 10px 0 0 0;
      padding: 20px;
    }
    .import_account {
      border: 1px solid #c1c1c1;
      margin: 10px 0 0 0;
      padding: 20px;
    }
    .transfer {
      border: 1px solid #c1c1c1;
      margin: 10px 0 0 0;
      padding: 20px;
    }

  }
</style>

