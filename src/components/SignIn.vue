<template>
  <div v-bind:class="amplifyUI.formSection">
    <div v-bind:class="amplifyUI.sectionHeader">{{options.header}}</div>
    <div v-bind:class="amplifyUI.sectionBody">
      <div v-bind:class="amplifyUI.formField">
        <div v-bind:class="amplifyUI.inputLabel">Email *</div>
        <input v-bind:class="amplifyUI.input"  v-model="options.email" placeholder="Email" autofocus v-on:keyup.enter="signIn" />
      </div>
      <div v-bind:class="amplifyUI.formField">
        <div v-bind:class="amplifyUI.inputLabel">Password *</div>
        <input  v-bind:class="amplifyUI.input" v-model="password" type="password" placeholder="Password" v-on:keyup.enter="signIn" />
        <div v-bind:class="amplifyUI.hint">
          Forgot your password?
          <a v-bind:class="amplifyUI.a" v-on:click="forgot">Reset</a>
        </div>
      </div>
    </div>
    <div v-bind:class="amplifyUI.sectionFooter">
      <span v-bind:class="amplifyUI.sectionFooterPrimaryContent">
        <button v-bind:class="amplifyUI.button" v-on:click="signIn">Sign In</button>
      </span>
      <span v-bind:class="amplifyUI.sectionFooterSecondaryContent">
        No Account?
        <a v-bind:class="amplifyUI.a" v-on:click="signUp">Sign Up</a>
      </span>
    </div>
    <div class="error" v-if="error">
      {{ error }}
    </div>
  </div>
</template>

<script>
// import Auth from '@aws-amplify/auth';
import { AmplifyEventBus } from 'aws-amplify-vue'
import * as AmplifyUI from '@aws-amplify/ui';
import { Auth } from 'aws-amplify'

export default {
  name: 'SignIn',
  props: ['signInConfig'],
  data () {
    return {
        password: '',
        error: '',
        amplifyUI: AmplifyUI,
    }
  },
  computed: {
    options() {
      const defaults = {
        header: 'Sign In',
        email: ''
      }
      return Object.assign(defaults, this.signInConfig || {})
    }
  },
  mounted() {
    
  },
  methods: {
    signIn: function() {
      Auth.signIn(this.options.email, this.password)
        .then(data => {
          if (data.challengeName) {
            AmplifyEventBus.$emit('localUser', data);
            return AmplifyEventBus.$emit('authState', 'confirmSignIn')
          } 
          return AmplifyEventBus.$emit('authState', 'signedIn')
        })
        .catch(e => this.setError(e));
    },
    forgot: function() {
      AmplifyEventBus.$emit('authState', 'forgotPassword')
    },
    signUp: function() {
      AmplifyEventBus.$emit('authState', 'signUp')
    },
    setError: function(e) {
      this.error = e.message || e;
    }
  }
}
</script>