<template>
    <div id="app">
      <div v-if="!signedIn">
         <v-Authenticator></v-Authenticator>
      </div>
      <div v-if="signedIn">
        <amplify-sign-out class="signout"></amplify-sign-out>
        <h1>Hello {{this.userInfo.attributes.email}}</h1>
      </div>
    </div>
</template>

<script>
import { AmplifyEventBus } from 'aws-amplify-vue'
import { Auth } from 'aws-amplify'
import Authenticator from './components/Authenticator'

export default {
  name: 'app',
  components: {
    'v-Authenticator': Authenticator
  },
  async beforeCreate() {
    try {
      this.userInfo = await Auth.currentAuthenticatedUser()
      this.signedIn = true
    } catch (err) {
      this.signedIn = false
    }
    AmplifyEventBus.$on('authState', async info => {
      if (info === 'signedIn') {
        this.signedIn = true
        this.userInfo = await Auth.currentAuthenticatedUser()
      } else {
        this.signedIn = false
      }
    });
  },
  data () {
    return {
      signedIn: false,
      userInfo : {}
    }
  }
}
</script>

<style>
body {
  margin: 0
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
.signout {
  background-color: #ededed;
  margin: 0;
  padding: 11px 0px 1px;
}
</style>