<template>
  <div class="confirmPassword">
    <hd-input
      ref="passwordMain"
      v-model="passwordMain"
      name="passwordMain"
      class="confirm-password__input"
      type="password"
      :icon="icon"
      :label="t.FORM.PASSWORD.LABEL"
      :required="required"/>

    <div class="confirmPassword__strengthMeter" :class="{isVisible: strengthBarVisible}">
      <div class="confirmPassword__strengthMeter__text"
      >{{ t.FORM.VALIDATION.PASSWORD_STRENGTH.LEVELS[strengthIndex] }}</div>
      <div class="confirmPassword__strengthMeter__gauge">
        <div class="confirmPassword__strengthMeter__gauge__bar" :class="barClasses"></div>
      </div>
    </div>

    <hd-input
      ref="passwordConfirm"
      v-model="passwordConfirm"
      name="passwordConfirm"
      class="confirmPassword__input"
      type="password"
      :icon="icon"
      :label="t.FORM.PASSWORD.LABEL_CONFIRM"
      :required="required"/>
  </div>
</template>

<script>
import merge from 'lodash/merge';
import { getMessages } from 'hd-blocks/lang';
import { getPasswordStrength } from 'hd-blocks/services/utils';
import HdInput from 'hd-blocks/components/form/HdInput.vue';

export default {
  name: 'hd-confirm-password',
  components: {
    HdInput,
  },
  props: {
    value: {
      type: String,
      default: '',
    },
    name: {
      type: String,
      default: '',
    },
    required: {
      type: Boolean,
      default: true,
    },
    min: {
      type: Number,
      default: 0,
    },
    withStrength: {
      type: Boolean,
      default: true,
    },
    strengthBarStyle: {
      type: Boolean,
      default: false,
    },
    icon: {
      type: String,
      default: '',
    },
    lang: String,
    texts: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      passwordMain: this.value,
      passwordConfirm: '',
      strengthIndex: null,
      strengthBarVisible: false,
    };
  },
  watch: {
    value(value) {
      this.passwordMain = value;
    },
    passwordMain(value) {
      this.$emit('input', value);
      this.$nextTick(this.validate);
    },
    passwordConfirm() {
      this.$nextTick(this.validate);
    },
  },
  computed: {
    t() {
      return merge(getMessages(this.lang), this.texts);
    },
    barClasses() {
      return {
        [`level--${this.strengthIndex}`]: this.strengthBarVisible && this.strengthIndex >= 0,
      };
    },
  },
  methods: {
    validate() {
      const inputsAreMatching = this.checkPasswordMatching();
      const passwordIsLongEnough = this.checkPasswordLength();

      const isValid = inputsAreMatching && passwordIsLongEnough;

      if (this.withStrength && passwordIsLongEnough) {
        this.updatePasswordStrength();
      } else {
        this.strengthBarVisible = false;
      }
      return isValid;
    },
    checkPasswordLength() {
      if (this.passwordMain.length === 0 && this.required) {
        this.$refs.passwordMain.showError(this.t.FORM.VALIDATION.REQUIRED);
        return false;
      }
      if (this.min > this.passwordMain.length) {
        this.$refs.passwordMain.showError(this.t.FORM.VALIDATION.PASSWORD_SHORT);
        return false;
      }
      this.$refs.passwordMain.hideError();
      return true;
    },
    checkPasswordMatching() {
      if (this.passwordConfirm.length === 0) {
        return false;
      }
      if (this.passwordMain !== this.passwordConfirm) {
        this.$refs.passwordConfirm.showError(this.t.FORM.VALIDATION.PASSWORD_MISMATCH);
        return false;
      }
      this.$refs.passwordConfirm.hideError();
      return true;
    },
    updatePasswordStrength() {
      this.strengthIndex = getPasswordStrength(this.passwordMain, 4);
      if (this.strengthBarStyle) {
        this.strengthBarVisible = true;
      } else {
        const COLORS = [
          'orange',
          'goldenRod',
          'limeGreen',
          'green',
        ];
        const { TITLE, LEVELS } = this.t.FORM.VALIDATION.PASSWORD_STRENGTH;
        const msg = `${TITLE}: <span style='color: ${COLORS[this.strengthIndex]}'
        >${LEVELS[this.strengthIndex]}</span>`;
        this.$refs.passwordMain.showHelper(msg);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.confirmPassword {
  width: 100%;
  position: relative;
  &__strengthMeter {
    height: 14px;
    width: 100%;
    padding-left: $inline-s;
    position: absolute;
    top: 66px; // input height + margin
    display: flex;
    align-items: center;
    visibility: hidden;
    &.isVisible {
      visibility: visible;
    }
    &__text {
      @include font('text-xxsmall');
      color: $regent-gray;
      width: 65px;
    }
    &__gauge {
      height: 2px;
      flex: 1;
      &__bar {
        background: grey;
        width: 0%;
        height: 100%;
        transition: width .5s, background-image 1s;
        position: relative;
        overflow: hidden;
        &:after {
          content: '';
          height: 100%;
          width: 20px;
          background-image: linear-gradient(to right, rgba(255, 255, 255, 0), rgba(255, 255, 255, .7));
          position: absolute;
          right: -30px;
          top: 0;
          opacity: 0;
        }
        &.level {
          &--0 {
            width: 25%;
            background-image: linear-gradient(to right, #f70, #fa0);
          }
          &--1 {
            width: 50%;
            background-image: linear-gradient(to right, #db0, #bd0);
          }
          &--2 {
            width: 75%;
            background-image: linear-gradient(to right, #9d0, #5d1);
          }
          &--3 {
            &:after {
              animation: fadeInLeft 2s;
            }
            width: 100%;
            background-image: linear-gradient(to right, #1d0, #094);
          }
        }
      }
    }
  }
  @keyframes fadeInLeft {
    0% {
      opacity: 0;
      transform: translateX(-300px);
    }
    20% {
      opacity: 1;
    }
    100% {
      opacity: 1;
      transform: translateX(0px);
    }
  }
}
</style>
