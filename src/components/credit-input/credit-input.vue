<template>
  <div>
    <input class="masked" :placeholder="this.mask" id="cc" ref="mask"/>
    <div v-if="cardInfo">
      {{cardInfo.bank.name}}
    </div>
  </div>
</template>

<script>
  import { throttle } from '../../helpers';

  var lookup = require('binlookup')();

  export default {
    props: {
      mask: {
        type: String,
        default: '#### #### #### ####'
      }
    },
    data() {
      return {
        cardInfo: null
      }
    },
    methods: {
      showCreditNumber() {
        const maskValue = document.getElementById('ccMask').innerText.replace(/\s/g, '');
        console.log(Number(maskValue))
      },
      fetchCardData(value) {
        lookup(value.replace(/\s/g, ''), (err, data) => {
          if (err) {
            console.log(err);
            return err;
          }
          this.cardInfo = {...data};
        });
      }
    },
    mounted() {
      const maskedInput = [this.$refs['mask']];
      console.log(maskedInput);
      const self = this;
      const masking = {
        maskedInputs: maskedInput,
        maskedNumber: '#Xx',
        maskedLetter: '_',

        init: function () {
          masking.setUpMasks(masking.maskedInputs);
          masking.maskedInputs = document.querySelectorAll('.masked');
          masking.activateMasking(masking.maskedInputs);
        },

        setUpMasks: function (inputs) {
          let i, l = inputs.length;

          for (i = 0; i < l; i++) {
            masking.createShell(inputs[i]);
          }
        },

        createShell: function (input) {
          let text = '',
            placeholder = input.getAttribute('placeholder');

          input.setAttribute('maxlength', placeholder.length);
          input.setAttribute('data-placeholder', placeholder);
          input.removeAttribute('placeholder');

          text = '<span class="shell">' +
            '<span aria-hidden="true" style="display: none" id="' + input.id +
            'Mask"><i></i>' + placeholder + '</span>' +
            input.outerHTML +
            '</span>';
          input.outerHTML = text;
        },

        setValueOfMask: throttle(function (e) {
          const value = e.target.value,
            placeholder = e.target.getAttribute('data-placeholder');
          console.log(value);
          self.fetchCardData(value);
          return value;
        }, 300),

        activateMasking: function (inputs) {
          let i, l;

          for (i = 0, l = inputs.length; i < l; i++) {
            masking.maskedInputs[i].addEventListener('keyup', function (e) {
              masking.handleValueChange(e);
            }, false);
          }
        },

        handleValueChange: function (e) {
          const id = e.target.getAttribute('id');

          switch (e.keyCode) {
            case 20: // CapsLock
            case 17: // Ctrl
            case 18: // Options
            case 16: // Shift
            case 37: // Arrows
            case 38:
            case 39:
            case 40:
            case  9: // tab (let blur handle tab)
              return;
          }

          document.getElementById(id).value = masking.handleCurrentValue(e);
          document.getElementById(id + 'Mask').innerHTML = masking.setValueOfMask(e);

        },

        handleCurrentValue: function (e) {
          let isCharsetPresent = e.target.getAttribute('data-charset'),
            placeholder = isCharsetPresent || e.target.getAttribute('data-placeholder'),
            value = e.target.value, l = placeholder.length, newValue = '',
            i, j, isInt, isLetter, strippedValue;

          strippedValue = isCharsetPresent ? value.replace(/\W/g, "") : value.replace(/\D/g, "");

          for (i = 0, j = 0; i < l; i++) {
            let x;
            let isInt = !isNaN(parseInt(strippedValue[j]));
            let isLetter = strippedValue[j] ? strippedValue[j].match(/[A-Z]/i) : false;
            let matchesNumber = masking.maskedNumber.indexOf(placeholder[i]) >= 0;
            let matchesLetter = masking.maskedLetter.indexOf(placeholder[i]) >= 0;

            if ((matchesNumber && isInt) || (isCharsetPresent && matchesLetter && isLetter)) {

              newValue += strippedValue[j++];

            } else if ((!isCharsetPresent && !isInt && matchesNumber) || (isCharsetPresent && ((matchesLetter && !isLetter) || (matchesNumber && !isInt)))) {
              masking.errorOnKeyEntry();
              return newValue;

            } else {
              newValue += placeholder[i];
            }
            // break if no characters left and the pattern is non-special character
            if (strippedValue[j] === undefined) {
              break;
            }
          }
          if (e.target.getAttribute('data-valid-example')) {
            return masking.validateProgress(e, newValue);
          }
          return newValue;
        },

        validateProgress: function (e, value) {
          let validExample = e.target.getAttribute('data-valid-example'),
            pattern = new RegExp(e.target.getAttribute('pattern')),
            placeholder = e.target.getAttribute('data-placeholder'),
            l = value.length, testValue = '';

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

        errorOnKeyEntry: function () {
          console.log('some error');
        }
      };

      masking.init();
    }
  }
</script>

<style scoped lang="scss">


</style>