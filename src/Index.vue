<template lang="pug">
div
  div.navBar-fixed
    nav.teal
      div.nav-wrapper
        a.brand-logo {{ name }}
          small 合計 {{ totalSupply }} (Wei)
        ul.right
          li 収支バランス: {{ balance }} (Wei)

  div.container
    div.row
      div.col.s12.m6.l6
        h2 トークン購入
        form(v-on:submit.prevent="onBuyToken")
          label(for="buyAmount") 金額 (Wei)
          input(id="buyAmount" ref="buyAmount" placeholder="1000000000000000000" type="text")

          input.btn(type="submit" value="購入")

      div.col.s12.m6.l6
        h2 トークン送信
        form(v-on:submit.prevent="onTransferToken")
          label(for="recipient") 送信先アドレス
          input(id="recipient" ref="recipient" placeholder="0xXXXXXXXXXXXXXXXXXXXX" type="text")
        
          label(for="amount") 金額 (Wei)
          input(id="amount" ref="amount" placeholder="1000000000000000000" type="text")

          input.btn(type="submit" value="送金")

  div
    ul.collection.with-header
      li.collection-header
        h4 トランザクションリスト
      li.collection-item(v-if="transactions.length === 0") 取引なし.
      li.collection-item(v-for="tx in transactions")
        a(href="#") {{ tx }}

</template>

<script lang="ts">
import Vue from 'vue';
import Component from 'vue-class-component';

import Web3 from 'web3';
import ContractInfo from './contractInfo';
import contractInfo from './contractInfo';
import { Contract } from 'web3/types';
import { setInterval } from 'timers';

let web3: Web3;
let token: Contract;
let address: string;

const gasLimit = 3000000;
const gasPrice = 10000000000;

@Component
export default class Index extends Vue {
  title = 'kawaCoin Exchanged';
  totalSupply = '0';
  symbol = 'XXX';
  balance = '0.0';
  transactions: string[] = [];

  onBuyToken() {
    const amount = this.$refs.buyAmount as HTMLInputElement;
    token.methods
      .buyToken()
      .send({
        from: address,
        value: amount.value,
        gas: gasLimit,
        gasPrice: gasPrice,
      })
      .on('transactionHash', txHash => {
        this.transactions.push(txHash);
      });
  }

  onTransferToken() {
    const to = this.$refs.transferTo as HTMLInputElement;
    const value = this.$refs.transferValue as HTMLInputElement;
    token.methods.buyTransfer(to.value, value.value).send({
      from: address,
      gas: gasLimit,
      gasPrice: gasPrice,
    });
  }

  // HTML要素を配置し終わったら呼ばれる
  mounted() {
    // Metamaskの古い0.20から自分で用意した1.0.0に変換している.

    web3 = new Web3(window.web3.currentProvider);
    web3.eth.getAccounts().then(result => {
      address = result[0];

      token = new web3.eth.Contract(
        contractInfo.Token.abi,
        contractInfo.Token.address
      );
      token.methods
        .symbol()
        .call({ from: address }) //水色のボタン
        .then(result => {
          this.symbol = result;
        });
      token.methods
        .totalSupply()
        .call({ from: address }) //水色のボタン
        .then(result => {
          this.totalSupply = result;
        });
      token.methods
        .balanceOf(address)
        .call({ from: address }) //水色のボタン
        .then(result => {
          this.balance = result;
        });
    });
  }
}
</script>

<style scoped>
/* CSS can be added here if you need. */
</style>
