<script>
import { useVuelidate } from '@vuelidate/core';
import { required, minLength, email } from '@vuelidate/validators';
import { useAlert } from 'dashboard/composables';
import NextButton from 'dashboard/components-next/button/Button.vue';

export default {
  components: {
    NextButton,
  },
  props: {
    show: {
      type: Boolean,
      default: false,
    },
    currentChat: {
      type: Object,
      default: () => ({}),
    },
  },
  emits: ['cancel', 'update:show'],
  setup() {
    return { v$: useVuelidate() };
  },
  data() {
    return {
      email: '',
      selectedType: '',
      isSubmitting: false,
    };
  },
  validations: {
    email: {
      required,
      email,
      minLength: minLength(4),
    },
  },
  computed: {
    localShow: {
      get() {
        return this.show;
      },
      set(value) {
        this.$emit('update:show', value);
      },
    },
    sentToOtherEmailAddress() {
      return this.selectedType === 'other_email_address';
    },
    isFormValid() {
      if (this.selectedType) {
        if (this.sentToOtherEmailAddress) {
          return !!this.email && !this.v$.email.$error;
        }
        return true;
      }
      return false;
    },
    selectedEmailAddress() {
      const { meta } = this.currentChat;
      switch (this.selectedType) {
        case 'contact':
          return meta.sender.email;
        case 'assignee':
          return meta.assignee.email;
        case 'other_email_address':
          return this.email;
        default:
          return '';
      }
    },
  },
  methods: {
    onCancel() {
      this.$emit('cancel');
    },
    async onSubmit() {
      this.isSubmitting = false;
      try {
        await this.$store.dispatch('sendEmailTranscript', {
          email: this.selectedEmailAddress,
          conversationId: this.currentChat.id,
        });
        useAlert(this.$t('EMAIL_TRANSCRIPT.SEND_EMAIL_SUCCESS'));
        this.onCancel();
      } catch (error) {
        useAlert(this.$t('EMAIL_TRANSCRIPT.SEND_EMAIL_ERROR'));
      } finally {
        this.isSubmitting = false;
      }
    },
  },
};
</script>

<template>
  <woot-modal v-model:show="localShow" :on-close="onCancel">
    <div class="flex flex-col h-auto overflow-auto">
      <woot-modal-header
        :header-title="$t('EMAIL_TRANSCRIPT.TITLE')"
        :header-content="$t('EMAIL_TRANSCRIPT.DESC')"
      />
      <form class="w-full" @submit.prevent="onSubmit">
        <div class="w-full">
          <div
            v-if="currentChat.meta.sender && currentChat.meta.sender.email"
            class="flex items-center gap-2"
          >
            <input
              id="contact"
              v-model="selectedType"
              type="radio"
              name="selectedType"
              value="contact"
            />
            <label for="contact">{{
              $t('EMAIL_TRANSCRIPT.FORM.SEND_TO_CONTACT')
            }}</label>
          </div>
          <div v-if="currentChat.meta.assignee" class="flex items-center gap-2">
            <input
              id="assignee"
              v-model="selectedType"
              type="radio"
              name="selectedType"
              value="assignee"
            />
            <label for="assignee">{{
              $t('EMAIL_TRANSCRIPT.FORM.SEND_TO_AGENT')
            }}</label>
          </div>
          <div class="flex items-center gap-2">
            <input
              id="other_email_address"
              v-model="selectedType"
              type="radio"
              name="selectedType"
              value="other_email_address"
            />
            <label for="other_email_address">{{
              $t('EMAIL_TRANSCRIPT.FORM.SEND_TO_OTHER_EMAIL_ADDRESS')
            }}</label>
          </div>
          <div v-if="sentToOtherEmailAddress" class="w-[50%] mt-1">
            <label :class="{ error: v$.email.$error }">
              <input
                v-model="email"
                type="text"
                :placeholder="$t('EMAIL_TRANSCRIPT.FORM.EMAIL.PLACEHOLDER')"
                @input="v$.email.$touch"
              />
              <span v-if="v$.email.$error" class="message">
                {{ $t('EMAIL_TRANSCRIPT.FORM.EMAIL.ERROR') }}
              </span>
            </label>
          </div>
        </div>
        <div class="flex flex-row justify-end w-full gap-2 px-0 py-2">
          <NextButton
            faded
            slate
            type="reset"
            :label="$t('EMAIL_TRANSCRIPT.CANCEL')"
            @click.prevent="onCancel"
          />
          <NextButton
            type="submit"
            :label="$t('EMAIL_TRANSCRIPT.SUBMIT')"
            :disabled="!isFormValid"
          />
        </div>
      </form>
    </div>
  </woot-modal>
</template>
