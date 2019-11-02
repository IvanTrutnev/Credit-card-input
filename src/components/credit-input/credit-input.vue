<template>
  <div>
    <input class="masked" :placeholder="this.mask" id="cc" ref="mask" />
    <div>
      <card-info :data="cardInfo" class="card-info"></card-info>
    </div>
  </div>
</template>

<script>
import CardInfo from "../card-info/card-info";
import { throttle } from "../../helpers";

const lookup = require("binlookup")();

export default {
  components: { CardInfo },
  props: {
    mask: {
      type: String,
      default: "#### #### #### ####"
    }
  },
  data() {
    return {
      cardInfo: {}
    };
  },
  methods: {
    fetchCardData(value) {
      fetch(`https://lookup.binlist.net/${value.replace(/\s/g, "")}`)
        .then(response => response.json())
        .then(data => {
          this.cardInfo = { ...data };
        })
        .catch(error => {
          //console.log(error);
        });
    }
  },
  mounted() {
    const maskedInput = this.$refs["mask"];
    const self = this;
    const masking = {
      maskedInput: maskedInput,
      maskedNumber: "#Xx",
      maskedLetter: "_",

      init: function() {
        masking.setUpMasks(masking.maskedInput);
        masking.maskedInput = document.querySelectorAll(".masked");
        masking.activateMasking(masking.maskedInput);
      },

      setUpMasks: function(input) {
        masking.createShell(input);
      },

      createShell: function(input) {
        let text = "",
          placeholder = input.getAttribute("placeholder");

        input.setAttribute("maxlength", placeholder.length);
        input.setAttribute("data-placeholder", placeholder);
        input.removeAttribute("placeholder");

        text =
          '<span class="shell">' +
          '<span aria-hidden="true" style="display: none" id="' +
          input.id +
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
        self.fetchCardData(value);
        return value;
      }, 1500),

      activateMasking: function(inputs) {
        inputs[0].addEventListener(
          "keyup",
          function(e) {
            masking.handleValueChange(e);
          },
          false
        );
      },

      handleValueChange: function(e) {
        const id = e.target.getAttribute("id");

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

        document.getElementById(id).value = masking.handleCurrentValue(e);
        document.getElementById(id + "Mask").innerHTML = masking.setValueOfMask(
          e
        );
      },

      handleCurrentValue: function(e) {
        let isCharsetPresent = e.target.getAttribute("data-charset"),
          placeholder =
            isCharsetPresent || e.target.getAttribute("data-placeholder"),
          value = e.target.value,
          l = placeholder.length,
          newValue = "",
          i,
          j,
          isInt,
          isLetter,
          strippedValue;

        strippedValue = isCharsetPresent
          ? value.replace(/\W/g, "")
          : value.replace(/\D/g, "");

        for (i = 0, j = 0; i < l; i++) {
          let x;
          let isInt = !isNaN(parseInt(strippedValue[j]));
          let isLetter = strippedValue[j]
            ? strippedValue[j].match(/[A-Z]/i)
            : false;
          let matchesNumber = masking.maskedNumber.indexOf(placeholder[i]) >= 0;
          let matchesLetter = masking.maskedLetter.indexOf(placeholder[i]) >= 0;

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
            masking.errorOnKeyEntry();
            return newValue;
          } else {
            newValue += placeholder[i];
          }

          if (strippedValue[j] === undefined) {
            break;
          }
        }
        if (e.target.getAttribute("data-valid-example")) {
          return masking.validateProgress(e, newValue);
        }
        return newValue;
      },

      validateProgress: function(e, value) {
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

      errorOnKeyEntry: function() {
        console.log("some error");
      }
    };

    masking.init();
  }
};
</script>

<style scoped lang="scss">
.masked {
  border-style: solid;
  border-width: 0 0 1px 0;
  border-color: #eee;
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
