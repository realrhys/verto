<template>
  <q-page class="column flex-center text-white bg-black">
    <q-card class="q-pa-sm bg-black" flat style="max-width: 500px; width: 100%;">
      <q-card-section class=" text-center text-uppercase">
        <q-icon class="float-left" name="help_outline" size="2.5rem" color="white" @click.native="$documentationManger.openDocumentation('settings/index')">
          <q-tooltip>{{ $t('SettingsView.help') }}</q-tooltip>
        </q-icon>
        <big class="titillium">Settings</big>
      </q-card-section>
    </q-card>
    <q-card class="q-pa-sm bg-black" flat style="max-width: 500px; width: 100%;">
      <q-card-section class="text-center settings-border">
        <div class="q-pa-sm">
          <big class="titillium text-uppercase">Version</big>
          <br>
          <span class="text-h6 q-pa-lg">
            {{ version }}
          </span>
        </div>
        <hr>
        <div class="q-pa-sm">
          <big class="titillium text-uppercase">Config Path</big>
          <br>
          <span class="break-word text-h6 q-pa-lg">
            {{ configPath }}
          </span>
        </div>
        <!--
        <hr>
        <div class="q-pa-sm">
          <big class="titillium text-uppercase">vDex URL</big>
            <br>
            <span class="break-word text-h6 q-pa-lg">
              {{ vdexUrl }}

                <q-input
                  v-model="vdexUrl"
                  type="url"
                  dark
                  color="green"
                  :label="$t('SettingsView.vdex_url')"
                />
            </span>
        </div>
        -->
        <hr>
        <div class="q-pa-sm">
          <big class="titillium text-uppercase">Network</big>
          <br>
          <br>
          <q-btn-toggle
            v-model="network"
            glossy
            @input="setNetwork"
            :options="[
              {label: 'Main Net', value: 'mainnet'},
              {label: 'Test Net', value: 'testnet'}
            ]"
          />
        </div>
        <hr>
        <div class="q-pa-sm">
          <big class="titillium text-uppercase">vDex URL</big>
          <br>
          <span class="break-word text-h6 q-pa-lg">
            {{ vdexUrl }}
          </span>
        </div>
        <div class='q-pa-md'>
          <q-btn class='full-width'
            outline
            glossy
            dense
            @click.native="$router.push({name:'edit-vdex-url', params: {}})"
          >
            <span class="text-h5">Edit</span>
          </q-btn>
        </div>
        <hr>
        <div class='q-pa-md'>
          <q-btn class='full-width'
            outline
            glossy
            dense @click.native="goChangePassword"
          >
            <span class="text-h5">{{ $t('SettingsView.change_password') }}</span>
          </q-btn>
        </div>
        <div class='q-pa-md'>
          <q-btn class='full-width'
            outline
            glossy
            dense
            @click.native="backupConfig"
          >
            <span class="text-h5">{{ $t('SettingsView.backup_config') }}</span>
            </q-btn>
        </div>
        <div class='q-pa-md'>
          <q-btn class='full-width'
            outline
            glossy
            dense
            @click.native="$router.push({name:'restore-wallet', params: {returnto: 'settings'}})"
          >
            <span class="text-h5">{{ $t('SettingsView.restore_config') }}</span>
          </q-btn>
        </div>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
import configManager from '@/util/ConfigManager'
import { version } from '../../../package.json'
let platformTools = require('../../util/platformTools')
if (platformTools.default) platformTools = platformTools.default

export default {
  data () {
    return {
      pword: '',
      minimizedModal: false,
      message: '',
      version: {},
      network: this.$store.state.settings.network,
      configPath: '',
      vdexUrl: ''
    }
  },
  mounted () {
    this.version = version
    this.setupPlatformPath()
    this.getVdexUrl()
    console.log(JSON.stringify(this.$store.state.currentwallet.config, null, 4))
  },
  methods: {
    async setupPlatformPath () {
      this.configPath = await platformTools.filePath()
    },
    async getVdexUrl () {
      const data = this.$store.state.currentwallet.config.data
      this.vdexUrl = 'NONE'
      if (data) {
        if (data.vdex) {
          this.vdexUrl = data.vdex.url
        }
      }
    },
    goChangePassword: function () {
      this.$router.push({ path: '/change-password' })
    },
    setNetwork: function () {
      this.$store.dispatch('settings/toggleNetwork', this.network)
      this.$q.notify({ message: `Network changed to ${this.network}`, color: 'positive' })
    },
    async backupConfig () {
      try {
        await configManager.backupConfig()
        if (this.$q.platform.is.android) {
          this.$q.notify({ color: 'positive', message: 'Config Saved' })
        }
      } catch (e) {
        // TODO: Exception handling
      }
    }
  }
}
</script>
<style>
.settings-border {
  border: solid thin white !important;
  border-style: solid;
  border-width: thin;
}
.break-word {
  word-wrap: break-word;
}
</style>
