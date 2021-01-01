<template>
  <div id="app">
    <VueGPicker
      :clientId="'394905518339-n3bgefs1sd3jhl768cgefpqbidm80p7p.apps.googleusercontent.com'"
      :developerKey="'AIzaSyBRBgQOXuwS1JisN3JmgspjMnlkwTT9wIA'"
      :scope="['https://www.googleapis.com/auth/drive.file']"
      @change="onChange"
      @authenticated="onAuthenticated"
      :multiselect="true"
      :navHidden="false"
      :authImmediate="false"
      :upload="true"
      :mimeTypes="['image/png', 'image/jpeg', 'image/jpg']"
      :viewId="'DOCS'">
      <button>VueGPicker</button>
    </VueGPicker>
  </div>
</template>

<script>
import VueGPicker from './components/VueGPicker.vue'

export default {
  name: 'app',
  methods: {
    onChange (data) {
      window.console.log('on change:', data)
    },
    onAuthenticated (token) {
      window.console.log('oauth token:', token)
    }
  },
  components: {
    VueGPicker
  }
}
</script>

