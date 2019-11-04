<template>
  <div>
    <input class="credit-input" ref="input" @input="inputChange"/>
    <div>
      <card-info :data="cardInfo" class="card-info"></card-info>
    </div>
  </div>
</template>

<script>
  import CardInfo from "../card-info/card-info";
  import { throttle } from "../../helpers";

  export default {
    components: {CardInfo},
    props: {
      mask: {
        type: String,
        default: "#### #### #### ####"
      }
    },
    data() {
      return {
        cardInfo: {}
      }
    },
    methods: {
      updateCardInfo: throttle(function (value) {
        if (value.length < 2) return;
        fetch(`https://lookup.binlist.net/${value}`)
          .then(response => response.json())
          .then(data => {
            this.cardInfo = data;
          })
          .catch(error => {
            this.cardInfo = {};
          })
      }, 1500),
      inputChange() {
        let value = this.$refs['input'].value.replace(/\D/g, '');
        if (!value.length) this.cardInfo = {};
        let res = '';
        for (let vi = 0, mi = 0; vi < value.length && mi < this.mask.length; mi++) {
          switch (this.mask[mi]) {
            case '#':
              res += value[vi];
              vi++;
              break;
            default:
              res += this.mask[mi]
          }
        }
        this.$refs['input'].value = res;
        this.updateCardInfo(res.replace(/\D/g, ''));
      }
    }
  }
</script>

<style scoped lang="scss">
  .masked {
    border-style: solid;
    border-width: 0 0 1px 0;
    border-color: #6a6262;
    border-radius: 0;
    background: #fff;
    font-family: "Courier New", monospace;
    font-size: 1.6em;
    padding: 0.3em 0.5em;
    outline: none;
    transition: border-color 0.2s;
    margin-left: auto;
    margin-right: auto;
    text-align: center;
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
  }

  .card-info {
    border: 1px solid #736d6d;
    border-radius: 10px;
    margin-top: 10px;
    padding: 10px;
  }

  .credit-input {
    border-style: solid;
    border-width: 0 0 1px 0;
    border-color: #6a6262;
  }
</style>
