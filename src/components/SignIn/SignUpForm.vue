<template>
  <div class="sign-up">
    <h3>Create Account</h3>
    <q-form @submit.stop="signUp">
      <label for="username">What's your name?</label>
      <input v-model="username" ref="input" id="username" autocomplete="false" />
      <button type="submit" class="btn btn-primary" :disabled="!validUsername">Create</button>
    </q-form>
  </div>
</template>

<script>
import {useAppStore} from 'stores/App'
import {useNostrStore} from 'src/nostr/NostrStore'
import {useSettingsStore} from 'stores/Settings'
import {generatePrivateKey} from 'nostr-tools'
import EventBuilder from 'src/nostr/EventBuilder'

export default {
  name: 'SignUpForm',
  emits: ['complete'],
  data() {
    return {
      username: null,
    }
  },
  computed: {
    validUsername() {
      return this.username && this.username.length > 0
    },
  },
  methods: {
    async signUp() {
      if (!this.validUsername) return

      const privkey = generatePrivateKey()

      const settings = useSettingsStore()
      const account = settings.addAccount({privkey})
      const app = useAppStore()
      app.switchAccount(account.pubkey)

      const event = EventBuilder.metadata(account.pubkey, {name: this.username}).build()
      await app.signEvent(event)
      if (await useNostrStore().publish(event)) {
        this.$emit('complete', {
          pubkey: account.pubkey,
          name: this.username
        })
      } else {
        this.$q.notify({
          message: 'Failed to create profile',
          color: 'negative',
        })
      }
    }
  },
  mounted() {
    this.$refs.input.focus()
  }
}
</script>

<style lang="scss" scoped>
@import "assets/theme/colors.scss";

.sign-up {
  margin: auto;
  label {
    display: block;
  }
  input {
    color: #fff;
    height: 50px;
    width: 100%;
    padding: 12px 20px;
    border-radius: 999px;
    margin: 1rem auto;
    background-color: rgba($color: $color-dark-gray, $alpha: 0.3);
    border: 1px solid rgba($color: $color-dark-gray, $alpha: 0);
    transition: border 150ms ease;
    &:focus {
      border: 1px solid rgba($color: $color-primary, $alpha: 1);
      outline: none;
    }
  }
  button {
    margin: auto;
  }
}
</style>
