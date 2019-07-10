<template>
<q-page class="column flex-center text-white bg-black">
  <q-card flat class="bg-black" style="width: 100%; max-width: 500px;">
    <q-card-section class="text-weight-bold text-center text-uppercase">
      <q-icon class="float-left" name="help_outline" size="2.5rem" color="white" @click.native="$documentationManger.openDocumentation('settings/editVdexUrl')">
        <q-tooltip>{{ $t('SettingsView.help') }}</q-tooltip>
      </q-icon>
      <big class="titillium q-pa-xl">{{ $t('SettingsView.edit_vdex_url') }}</big>
      <q-icon class="float-right" name="close" size="2.5rem" color="white" @click.native="$router.push({name:'settings'})"/>
    </q-card-section>
  </q-card>
  <q-card flat class="bg-black" style="width: 100%; max-width: 500px;">
      <q-inner-loading :visible="spinnervisible">
        <q-spinner size="50px" color="primary" />
      </q-inner-loading>
      <q-stepper done-color="green" active-color="green" ref="stepper" alternative-labels animated v-model="step">
        <q-step :name="1" :done="step>1" title="Select Config File" class=" bg-black workflow-step">
          <q-card-section class="text-center text-white"  >
              <q-input
                  v-model="vdexUrl"
                  type="url"
                  dark
                  color="green"
                  @input="checkVdexUrlHasValue"
                  :label="$t('SettingsView.vdex_url')"
                />
            <div
              v-show="showNextButtonToPassword"
              class="text-center q-pa-sm"
              @click="showThePasswordScreen"
            >
              <q-icon name="navigate_next" size="3.2rem" color="green"   >
                <q-tooltip>{{ $t('next') }}</q-tooltip>
              </q-icon>
            </div>
          </q-card-section>
        </q-step>
        <q-step :name="2" :done="step>2" title="Verto Password" class=" bg-black workflow-step">
          <q-card-section class=" text-white text-center"  >
            <div>
                <q-input
                  dark
                  v-model="vertoPassword"
                  type="password"
                  color="green"
                  @input="showSubmitKey"
                  @keyup.enter="saveVdexUrl"
                  v-bind:label="$t('CreateVertoPassword.vertopassword')"
                />
            </div>
            <div v-show="incorrectPassword" class="text-h6 text-uppercase text-red q-pa-lg">
              {{ $t('Welcome.incorrect') }}
            </div>
            <div v-show="unknownError" class="text-h6 text-uppercase text-red q-pa-lg">
              Unknown Error
            </div>
            <div v-show="submitKey" class="q-pa-md" @click="saveVdexUrl">
              <q-icon name="navigate_next" size="3.2rem" color="green" >
                <q-tooltip>{{ $t('next') }}</q-tooltip>
              </q-icon>
            </div>
            <q-btn color="white" flat @click="$refs.stepper.previous()" label="Back" />
          </q-card-section>
        </q-step>
      </q-stepper>
  </q-card>
</q-page>
</template>

<script>
import configManager from '@/util/ConfigManager'
import { userError } from '@/util/errorHandler'

export default {
  name: 'RestoreConfig',
  data () {
    return {
      step: 1,
      vdexUrl: '',
      spinnervisible: false,
      contractable: true,
      submitKey: false,
      unknownError: false,
      incorrectPassword: false,
      showNextButtonToPassword: false,
      vertoPassword: ''
    }
  },
  created () {
    this.returnto = this.$route.params.returnto
  },
  mounted () {
    this.getVdexUrl()
  },
  methods: {
    async getVdexUrl () {
      const data = this.$store.state.currentwallet.config.data
      if (data) {
        if (data.vdex) {
          this.vdexUrl = data.vdex.url
        }
      }
    },
    goback () {
      if (this.returnto === 'settings') {
        this.$router.push('/settings')
      } else {
        this.$router.push(this.returnto)
      }
    },
    showThePasswordScreen () {
      if (this.showNextButtonToPassword) {
        this.$refs.stepper.next()
      }
    },
    showSubmitKey () {
      this.resetErrors()
      if (this.vertoPassword.length > 2) {
        this.submitKey = true
      } else {
        this.submitKey = false
      }
    },
    resetErrors () {
      this.incorrectPassword = false
      this.unknownError = false
    },
    checkVdexUrlHasValue: function () {
      if (!this.vdexUrl) {
        this.showNextButtonToPassword = false
      } else {
        this.showNextButtonToPassword = true
      }
    },
    async saveVdexUrl () {
      this.incorrectPassword = false
      if (!this.submitKey) {
        return
      }
      this.submitKey = false
      const config = this.$store.state.currentwallet.config
      if (!config.data) {
        config.data = {}
      }
      if (!config.data.vdex) {
        config.data.vdex = {}
      }
      config.data.vdex.url = this.vdexUrl
      try {
        const results = await configManager.updateConfig(this.vertoPassword, config)
        if (results.message === 'bad_password') {
          this.incorrectPassword = true
          this.spinnervisible = false
          this.vertoPassword = ''
          this.submitKey = false
        } else {
          this.$q.notify({ color: 'positive', message: 'Application refreshing' })
          const that = this
          setTimeout(function () {
            that.spinnervisible = false
            that.$router.push({ name: 'settings' })
          }, 200)
        }
      } catch (e) {
        this.incorrectPassword = true
        this.spinnervisible = false
        this.vertoPassword = ''
        userError(e)
      }
    }
  }
}
</script>

<style lang="stylus" scoped>
a
  cursor: pointer
</style>
