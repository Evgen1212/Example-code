<template>
  <form class="form" @submit.prevent="onSubmit">
    <slot
      :onFormValidation="onFormValidation"
      :onFormValues="onFormValues"
      :isFormValid="isFormValid"
      :formValues="formValues"
      :recaptchaOnError="recaptchaOnError"
      :recaptchaOnSuccess="recaptchaOnSuccess"
      :recaptchaOnExpired="recaptchaOnExpired"
    />
  </form>
</template>

<script>
export default {
  props: {
    captcha: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      successCaptcha: false,
      formValidation: {},
      formValues: {},
    };
  },
  computed: {
    isFormValid() {
      return Object.values(this.formValidation).every((el) => el === true);
    },
  },
  mounted() {
    if (this.captcha) {
      this.onFormValidation({ successCaptcha: this.successCaptcha });
    }
  },
  methods: {
    recaptchaOnError() {
      this.successCaptcha = false;
      this.onFormValidation({ successCaptcha: this.successCaptcha });
    },
    recaptchaOnSuccess() {
      this.successCaptcha = true;
      this.onFormValidation({ successCaptcha: this.successCaptcha });
    },
    recaptchaOnExpired() {
      this.successCaptcha = false;
      this.onFormValidation({ successCaptcha: this.successCaptcha });
    },
    onSubmit(data) {
      if (!this.isFormValid) return;
      this.$emit('onSubmit', data);
    },
    onFormValidation(data) {
      this.formValidation = {
        ...this.formValidation,
        ...data,
      };
    },
    onFormValues(data) {
      this.formValues = {
        ...this.formValues,
        ...data,
      };
    },
  },
};
</script>
