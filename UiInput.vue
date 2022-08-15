<template>
  <div class="ui-input">
    <label v-if="label" class="ui-input__label-1">
      {{ label }}
    </label>

    <span
      v-if="rules.min && showLimitLabel"
      class="ui-input-info ui-input-info--min"
      >Min: {{ min }}</span
    >
    <span
      v-if="rules.max && showLimitLabel"
      class="ui-input-info ui-input-info--max"
      >Max: {{ max }}</span
    >

    <span v-if="rules.reserv" class="ui-input-info ui-input-info--reserv"
      >{{ $tc('reserve', 1) }}: {{ reserv }}</span
    >
    <div
      class="d-flex align-center"
      :class="[isTextArea ? 'ui-input__icon--area' : 'ui-input__icon']"
    >
      <template v-if="validateIcon">
        <transition name="fade" leave-absolute>
          <img
            v-if="((isDirty && !isValid) || propsInvalidStatus) && showFailIcon"
            width="16"
            height="16"
            src="/icons/uncheck.svg"
          />
          <img
            v-else-if="isDirty && inputVal && showSuccessIcon"
            width="20"
            height="15"
            src="/icons/check-email.svg"
          />
        </transition>
      </template>
      <template v-if="eye">
        <img
          v-if="!isPasswordVisible"
          style="cursor: pointer; margin-left: 10px"
          src="/icons/notShownPass.svg"
          alt=""
          @click="isPasswordVisible = !isPasswordVisible"
        />
        <img
          v-else
          style="cursor: pointer; margin-left: 10px"
          src="/icons/shownPass.svg"
          alt=""
          @click="isPasswordVisible = !isPasswordVisible"
        />
      </template>
      <div>
        <slot name="icon" />
      </div>
    </div>

    <input
      v-if="!isTextArea"
      v-model="inputVal"
      :disabled="disabled"
      :name="name"
      :maxlength="maxlength"
      :type="eye ? passwordInputState : type"
      :placeholder="placeholder"
      :class="{
        'ui-input__field--error': (isDirty && !isValid) || propsInvalidStatus,
        styleIsClear: styleIsClear,
      }"
      class="ui-input__field"
      @focus="onFocus"
      @input="clearInvalidTrigger"
    />
    <textarea
      v-else
      ref="textArea"
      v-model="inputVal"
      :rows="rows"
      :maxlength="maxlength"
      :name="name"
      :disabled="disabled"
      :placeholder="placeholder"
      class="ui-input__field"
      :class="{
        'ui-input__field--error': isDirty && !isValid,
        styleIsClear: styleIsClear,
      }"
      @keydown="onKeyDown"
      @focus="onFocus"
    />
    <div
      v-if="isDirty && !isValid"
      class="ui-input-error"
      :class="{ 'ui-input-error-static': isErrorTextStatic }"
    >
      {{ errorMessageText }}
    </div>
  </div>
</template>
<script>
export default {
  name: 'UiInput',
  props: {
    maxlength: {
      type: Number,
      default: 3000,
    },
    styleIsClear: {
      type: Boolean,
      default: false,
    },
    eye: {
      type: Boolean,
      default: false,
    },
    isErrorTextStatic: {
      type: Boolean,
      default: false,
    },
    validateIcon: {
      type: Boolean,
      default: false,
    },
    label: {
      type: String,
      default: '',
    },
    name: {
      type: String,
      default: '',
    },
    type: {
      type: String,
      default: 'text',
    },
    placeholder: {
      type: [String, Number],
      default: '',
    },
    errorMessage: {
      type: String,
      default: '',
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    value: {
      type: [String, Number],
      default: '',
    },
    rules: {
      type: Object,
      default: () => {},
    },
    addLabel: {
      type: Object,
      default: null,
    },
    isTextArea: {
      type: Boolean,
      default: false,
    },
    currencyType: {
      type: String,
      default: '',
    },
    exponent: {
      type: Number,
      default: 0,
    },
    rows: {
      type: Number,
      default: 6,
    },
    propsInvalidTrigger: {
      type: Boolean,
      default: false,
    },
    showSuccessIcon: {
      type: Boolean,
      default: true,
    },
    showFailIcon: {
      type: Boolean,
      default: true,
    },
    showLimitLabel: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      localValue: '',
      isValid: false,
      isDirty: false,
      validationData: {},
      isPasswordVisible: false,
      propsInvalidStatus: false,
    };
  },
  computed: {
    passwordInputState: function () {
      if (this.isPasswordVisible) {
        return 'text';
      } else {
        return 'password';
      }
    },
    reserv() {
      return this.isFiat
        ? this.$options.filters.thousNumb(
            (this.rules?.reserv).toFixed(this.exponent)
          )
        : this.rules?.reserv.toFixed(this.exponent);
    },

    min() {
      return this.isFiat
        ? this.$options.filters.thousNumb(this.rules?.min)
        : this.rules?.min;
    },
    max() {
      return this.isFiat
        ? this.$options.filters.thousNumb(this.rules?.max)
        : this.rules?.max;
    },
    isFiat() {
      return this.currencyType === 'FIAT';
    },
    isNotRu() {
      return this.$i18n?.locale !== 'ru';
    },
    errorMessageText() {
      if (this.errorMessage) return this.errorMessage;
      if (this.rules?.required) return this.$t('required_field');
      if (this.rules?.min || this.rules?.max) return this.$t('required_value');
      if (this.rules?.regExp) return this.$t('invalid_email');
      if (this.rules?.repeat !== undefined)
        return this.$t('passwords_not_match');
      // return this.$t('required_field');
    },
    inputVal: {
      get() {
        return this.value || this.localValue;
      },
      set(value) {
        this.localValue = value;
        this.$emit('input', value);
      },
    },
  },

  watch: {
    isValid(val) {
      this.$emit('validate', { [this.name]: val });
    },
    inputVal: {
      handler: function (val) {
        // this.validation();
        // this.isValid = Object.values(this.validationData).every(
        //   (el) => el() === true
        // );
        // this.$emit('validate', { [this.name]: this.isValid });
        this.$emit('value', { [this.name]: this.inputVal });
        if ((this.name == 'to' || this.name == 'from') && this.value != '') {
          if (!this.isDirty) this.isDirty = true;
        }
      },
      immediate: true,
    },
    rules: {
      handler() {
        this.validation();
      },
      deep: true,
    },
    propsInvalidTrigger(val) {
      this.propsInvalidStatus = val;
    },
  },
  mounted() {
    this.$nextTick(() => {
      this.validation();
      this.$emit('validate', { [this.name]: this.isValid });
      this.$emit('value', { [this.name]: this.inputVal });
    });
  },
  methods: {
    clearInvalidTrigger() {
      this.propsInvalidStatus = false;
    },
    validation() {
      let result = true;
      if (this.rules?.required && this.inputVal.length <= 0) {
        result = false;
      }
      if (
        this.rules?.minLength &&
        this.inputVal.length < this.rules.minLength
      ) {
        result = false;
      }
      if (
        this.rules?.repeat !== undefined &&
        this.inputVal !== this.rules.repeat
      ) {
        result = false;
      }
      if (
        this.rules?.regExp &&
        !new RegExp(this.rules.regExp).test(this.inputVal)
      ) {
        result = false;
      }
      if (this.rules?.min && this.inputVal < parseFloat(this.rules.min)) {
        result = false;
      }
      if (this.rules?.max && this.inputVal > parseFloat(this.rules.max)) {
        result = false;
      }
      if (
        this.rules?.reserv &&
        parseFloat(this.inputVal) > parseFloat(this.rules.reserv)
      ) {
        result = false;
      }
      this.isValid = result;
    },
    onFocus(e) {
      if (!this.isDirty) this.isDirty = true;
      this.$emit('focus', e);
    },
    onKeyDown(e) {
      if (this.inputVal.length >= 5000) {
        if (e.keyCode >= 48 && e.keyCode <= 90) {
          e.preventDefault();
          return;
        }
      }
    },
  },
};
</script>
<style lang="scss" scoped>
.ui-input {
  position: relative;
  width: 100%;
  &__icon {
    position: absolute;
    right: 20px;
    top: 20.5px;
    transform: translate(0, -50%);
    display: flex;
    z-index: 2;
    &--area {
      position: absolute;
      right: 20px;
      top: 20%;
      transform: translate(0, -50%);
      display: flex;
      z-index: 2;
    }
  }
  &__field {
    color: white;
    background-color: #524f70;
    width: 100%;
    border: 1px solid transparent;
    padding: 10px 15px;
    border-radius: 5px;
    resize: none;
    overflow: hidden;
    min-height: 30px;
    position: relative;
    z-index: 1;
    &--error {
      border: 1px solid #fc5d47;
      color: #ffffff;
    }
  }
  &-error {
    position: absolute;
    color: #fc5d47;
    font-weight: 500;
    transform: translate(0, -50%);
    padding-top: 28px;
    line-height: 14px;
    @include font-size(14px);
    &-static {
      position: static;
      padding-top: 8px;
      font-weight: 500;
      font-size: 14px;
      line-height: 140%;
      transform: translate(0, 0);
    }
  }
  &__label-1 {
    color: #b1a3bf;
    position: absolute;
    font-weight: 500;
    font-size: 16px;
    top: 0;
    transform: translate(0, -50%);
    padding-bottom: 28px;
  }
  &-info {
    position: absolute;
    @include font-size(1.3rem);
    font-weight: 700;
    &--min {
      right: 0;
      position: absolute;
      top: 0;
      transform: translate(0, -50%);
      padding-bottom: 20px;
    }
    &--max {
      right: 0;
      position: absolute;
      top: 100%;
    }
    &--reserv {
      right: 0;
      position: absolute;
      top: 100%;
      padding-top: 16px;
    }
  }
}
.styleIsClear {
  color: white;
  background-color: #34324e;
  width: 100%;
  border: 1px solid #524f70;
  padding: 10px 15px;
  border-radius: 5px;
  resize: none;
  overflow: hidden;
  min-height: 30px;
  position: relative;
  z-index: 1;
}
::-webkit-input-placeholder {
  color: #b1a3bf;
  font-size: 14px;
}

textarea:focus,
input:focus {
  outline: none;
}
img {
  -moz-user-select: none;
  -webkit-user-select: none;
  -ms-user-select: none;
  user-select: none;
}
</style>
