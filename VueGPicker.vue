<template>
  <div @click="onChoose">
    <slot/>
    <button v-if="!this.$slots.default">Open google chooser</button>
  </div>
</template>


<script>
import loadScript from 'load-script'

const GOOGLE_SDK_URL = 'https://apis.google.com/js/api.js'

export default {
  props: {
    clientId: {
      type: String,
      required: true
    },
    developerKey: String,
    scope: {
      type: Array,
      default: function () {
        return ['https://www.googleapis.com/auth/drive.readonly']
      }
    },
    viewId: {
      type: String,
      default: 'DOCS' 
    },
    origin: String,
    createPicker: Function,
    authImmediate: {
      type: Boolean,
      default: false
    },
    multiselect: {
      type: Boolean,
      default: false
    },
    navHidden: {
      type: Boolean,
      default: false
    },
    upload: {
      type: Boolean,
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false
    },
    mimeTypes: {
      type: Array
    }
  },
  data: () => ({
    scriptLoadingStarted: false
  }),
  mounted () {
    if(this.isGoogleReady()) {
      // google api is already exists
      // init immediately
      this.onApiLoad()
    } else if (!this.scriptLoadingStarted) {
      // load google api and the init
      this.scriptLoadingStarted = true
      loadScript(GOOGLE_SDK_URL, this.onApiLoad)
    } else {
      // is loading
    }
  },
  methods: {
    isGoogleReady () {
      return !!window.gapi
    },
    isGoogleAuthReady () {
      return !!window.gapi.auth
    },
    isGooglePickerReady () {
      return !!window.google.picker
    },
    onApiLoad () {
      window.gapi.load('auth')
      window.gapi.load('picker')
    },
    doAuth (callback) {
      window.gapi.auth.authorize({
          client_id: this.clientId,
          scope: this.scope,
          immediate: this.authImmediate
        },
        callback
      )
    },
    onChoose () {
      if (!this.isGoogleReady() || !this.isGoogleAuthReady() || !this.isGooglePickerReady() || this.disabled) {
        return
      }

      const token = window.gapi.auth.getToken()
      const oauthToken = token && token.access_token

      if (oauthToken) {
        this.defaultCreatePicker(oauthToken)
      } else {
        this.doAuth(({ access_token }) => this.defaultCreatePicker(access_token))
      }
    },
    pickerCallback (data) {
      this.$emit('change', data)
      switch (data.action) {
        case 'loaded':
          this.$emit('loaded')
          break
        case 'cancel':
          this.$emit('cancel')
          break
        case 'picked':
          this.$emit('picked', data.docs)
          break
        default: 
          break
      }
    },
    onAuthenticated (oauthToken) {
      this.$emit('authenticated', oauthToken)
    },
    defaultCreatePicker (oauthToken) {

      this.onAuthenticated(oauthToken)

      if(this.createPicker){
        return this.createPicker(window.google, oauthToken)
      }

      const googleViewId = window.google.picker.ViewId[this.viewId]
      const view = new window.google.picker.View(googleViewId)

      if (this.mimeTypes) {
        view.setMimeTypes(this.mimeTypes.join(','))
      }

      if (!view) {
        throw new Error('Can\'t find view by viewId')
      }

      const picker = new window.google.picker.PickerBuilder()
        .addView(view)
        .setOAuthToken(oauthToken)
        .setDeveloperKey(this.developerKey)
        .setCallback(this.pickerCallback)

      if (this.origin) {
        picker.setOrigin(this.origin)
      }

      if (this.navHidden) {
        picker.enableFeature(window.google.picker.Feature.NAV_HIDDEN)
      }

      if (this.multiselect) {
        picker.enableFeature(window.google.picker.Feature.MULTISELECT_ENABLED)
      }

      if (this.upload) {
        picker.addView(new window.google.picker.DocsUploadView())
      }

      picker.build()
        .setVisible(true)
    }
  }
}
</script>
