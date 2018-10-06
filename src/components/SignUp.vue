<template>
<div id="app">
  <div v-bind:class="amplifyUI.formSection">
    <div v-bind:class="amplifyUI.sectionHeader">Sign Up</div>
    <div v-bind:class="amplifyUI.sectionBody">
      <div v-bind:class="amplifyUI.formField" 
          v-for="signUpField in orderBy(this.options.signUpFields, 'displayOrder')" 
          :signUpField="signUpField" 
          v-bind:key="signUpField.key"
        >
        <div v-bind:class="amplifyUI.inputLabel">{{signUpField.label}} {{signUpField.required ? '*': ''}}</div>
        <input 
            :type = "signUpField.type" 
            v-bind:class="[amplifyUI.input, signUpField.invalid ? 'invalid': '']" 
            v-model="signUpField.value" 
            :placeholder="signUpField.label"
            v-on:change="clear(signUpField)" 
          />
        
      </div>
    </div>
    <div v-bind:class="amplifyUI.sectionFooter">
      <span v-bind:class="amplifyUI.sectionFooterPrimaryContent">
        <button v-bind:class="amplifyUI.button" v-on:click="signUp">Create Account</button>
      </span>
      <span v-bind:class="amplifyUI.sectionFooterSecondaryContent">
        Have an Account?
        <a v-bind:class="amplifyUI.a" v-on:click="signIn">Sign In</a>
      </span>
    </div>
    <div class="error" v-if="error1">
      {{ error1 }}
    </div>
  </div>
  </div>
</template>

<script>
import Vue from 'vue'
import { AmplifyEventBus } from 'aws-amplify-vue'
import { Auth } from 'aws-amplify'
import * as AmplifyUI from '@aws-amplify/ui'
export default {
  name: 'SignUp',
  async beforeCreate() {
    try {
      await Auth.currentAuthenticatedUser()
      this.signedIn = true
    } catch (err) {
      this.signedIn = false
    }
    AmplifyEventBus.$on('authState', info => {
      if (info === 'signedIn') {
        this.signedIn = true
      } else {
        this.signedIn = false
      }
    });
  },
  data () {
    return {
      amplifyUI: AmplifyUI,
      error1: '',
      signedIn: false
      
    }
  },
  computed: {
    options() {
      const defaults = {
        header: 'Sign Up',
        signUpFields: [
          
          
          {
            label: 'Email',
            key: 'email',
            required: true,
            type: 'string',
            displayOrder: 1
          },{
            label: 'Password',
            key: 'password',
            required: true,
            type: 'password',
            displayOrder: 2,
          },
        ]
      }
      return Object.assign(defaults, this.signUpConfig || {})
    }
  
  },
  methods: {
    signUp: function() {
      if (!this.validate()) {
        return null;
      }
      let user = {
        attributes: {},
      };
      this.options.signUpFields.forEach((e) => {
        
        if(e.key ==='email') {
          user.username = e.value
        }else if (e.key === 'password') {
          user.password = e.value
        } else if (e.key === 'phone_number') {
          user.attributes.phone_number = `+${this.countryCode}${e.value}`
        } else {
          user.attributes[e.key] = e.value;
        }
      })
      
       this.$Amplify.Auth.signUp(user)
            .then(data => {
              
              AmplifyEventBus.$emit('localUser', data.user)
              if (data.userConfirmed === false){
                return AmplifyEventBus.$emit('authState', 'confirmSignUp');
              }
              return AmplifyEventBus.$emit('authState', 'signedOut')
            })
            .catch(e => this.setError(e));
    },
    validate: function() {
      let allValid = true;
      this.options.signUpFields.map((el) => {
        if (el.required && !el.value) {
          allValid = false;
          Vue.set(el, 'invalid', true);
        }
        return el;
      })
      return allValid;
    },
    signIn: function() {
      AmplifyEventBus.$emit('authState', 'signedOut')
    },
    clear(field) {
      if (field && field.invalid && field.value) {
        Vue.set(field, 'invalid', false)
      }
    },
    setError: function(e) {
      this.error1 = e.message || e
      // console.log(e.message)
      // Logger.error(e.message) 
    },
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