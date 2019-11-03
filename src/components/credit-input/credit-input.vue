<template>
  <div>
    <input class="masked" data-type="masked" :data-id="uniqkey" :placeholder="this.mask" ref="mask" />
    <div>
      <card-info :data="cardInfo" class="card-info"></card-info>
    </div>
  </div>
</template>

<script>
import CardInfo from "../card-info/card-info";
import { throttle } from "../../helpers";

export default {
  components: { CardInfo },
  props: {
    mask: {
      type: String,
      default: "#### #### #### ####"
    },
    uniqkey: {
      type: [String, Number],
      default: 0
    }
  },
  data() {
    return {
      maskedNumber: "#Xx",
      maskedLetter: "_",
      cardInfo: {}
    };
  },
  methods: {
    fetchCardData(value) {
      const payload = value.replace(/\s/g, "");
      fetch(`https://lookup.binlist.net/${payload}`)
        .then(response => response.json())
        .then(data => {
          this.cardInfo = { ...data };
        })
        .catch(error => {
          this.cardInfo = {};
        });
    },
    setUpMask(input) {
      this.createShell(input);
    },
    createShell(input) {
      let text = "",
        placeholder = input.getAttribute("placeholder");

      input.setAttribute("maxlength", placeholder.length);
      input.setAttribute("data-placeholder", placeholder);

      text =
        '<span class="shell">' +
        '<span aria-hidden="true" style="display: none" id="' +
        input.dataset.id +
        'Mask"><i></i>' +
        placeholder +
        "</span>" +
        input.outerHTML +
        "</span>";
      input.outerHTML = text;
    },
    setValueOfMask: throttle(function(e) {
      const value = e.target.value,
        placeholder = e.target.getAttribute("data-placeholder");
      this.fetchCardData(value);
      return value;
    }, 1500),
    activateMasking(inputs) {
      const self = this;
      inputs[0].addEventListener("keyup", e => {
        self.handleValueChange(e);
      });
    },
    handleValueChange(e) {
      const id = e.target.getAttribute("data-id");

      switch (e.keyCode) {
        case 20: // CapsLock
        case 17: // Ctrl
        case 18: // Options
        case 16: // Shift
        case 37: // Arrows
        case 38:
        case 39:
        case 40:
        case 9: // tab (let blur handle tab)
          return;
      }
      console.log(id);
      document.querySelectorAll('[data-id]')[0].value = this.handleCurrentValue(e);
      document.getElementById(`${id}Mask`).innerHTML = this.setValueOfMask(e);
    },
    handleCurrentValue(e) {
      let isCharsetPresent = e.target.getAttribute("data-charset"),
        placeholder =
          isCharsetPresent || e.target.getAttribute("data-placeholder"),
        value = e.target.value,
        l = placeholder.length,
        newValue = "",
        isInt,
        isLetter,
        strippedValue;

      strippedValue = isCharsetPresent
        ? value.replace(/\W/g, "")
        : value.replace(/\D/g, "");

      for (let i = 0, j = 0; i < l; i++) {
        let x;
        let isInt = !isNaN(parseInt(strippedValue[j]));
        let isLetter = strippedValue[j]
          ? strippedValue[j].match(/[A-Z]/i)
          : false;
        let matchesNumber = this.maskedNumber.includes(placeholder[i]);
        let matchesLetter = this.maskedLetter.includes(placeholder[i]);

        if (
          (matchesNumber && isInt) ||
          (isCharsetPresent && matchesLetter && isLetter)
        ) {
          newValue += strippedValue[j++];
        } else if (
          (!isCharsetPresent && !isInt && matchesNumber) ||
          (isCharsetPresent &&
            ((matchesLetter && !isLetter) || (matchesNumber && !isInt)))
        ) {
          this.errorOnKeyEntry();
          return newValue;
        } else {
          newValue += placeholder[i];
        }

        if (strippedValue[j] === undefined) {
          break;
        }
      }
      if (e.target.getAttribute("data-valid-example")) {
        return this.validateProgress(e, newValue);
      }
      return newValue;
    },
    validateProgress(e, value) {
      let validExample = e.target.getAttribute("data-valid-example"),
        pattern = new RegExp(e.target.getAttribute("pattern")),
        placeholder = e.target.getAttribute("data-placeholder"),
        l = value.length,
        testValue = "";

      for (i = l; i >= 0; i--) {
        testValue = value + validExample.substr(value.length);
        if (pattern.test(testValue)) {
          return value;
        } else {
          value = value.substr(0, value.length - 1);
        }
      }

      return value;
    },
    errorOnKeyEntry() {
      console.log("some error");
    },
    init() {
      const maskedInput = this.$refs["mask"];
      const masking = {
        maskedInput
      };
      this.setUpMask(maskedInput);
      masking.maskedInput = document.querySelectorAll("[data-type='masked']");
      this.activateMasking(masking.maskedInput);
    }
  },

  mounted() {
    this.init();
  }
};
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
</style>
