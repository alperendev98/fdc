<template>
  <div>
    <fdc-sign-in v-if="displayMap.showSignIn" v-bind:signInConfig="options.signInConfig"></fdc-sign-in>
    <fdc-sign-up v-if="displayMap.showSignUp" v-bind:signUpConfig="options.signUpConfig"></fdc-sign-up>
    <amplify-confirm-sign-up v-if="displayMap.showConfirmSignUp" v-bind:confirmSignUpConfig="options.confirmSignUpConfig"></amplify-confirm-sign-up>
    <amplify-confirm-sign-in v-if="displayMap.showConfirmSignIn" v-bind:confirmSignInConfig="options.confirmSignInConfig"></amplify-confirm-sign-in>
    <amplify-forgot-password v-if="displayMap.showForgotPassword" v-bind:forgotPasswordConfig="options.forgotPasswordConfig"></amplify-forgot-password>
  </div>
</template>

<script>
import { AmplifyEventBus } from 'aws-amplify-vue'
import { Auth} from 'aws-amplify';
import SignIn from './SignIn'
import SignUp from './SignUp'
export default {
  name: 'Authenticator',
  props: ['authOptions'],
  components: {
    'fdc-sign-in': SignIn,
    'fdc-sign-up': SignUp
  },
  data () {
    return {
        user: {
          email: null
        },
        displayMap: {showSignIn:true},
        error: '',
        logger: {}
    }
  },
  computed: {
    options() {
      const defaults = {
        signInConfig: {},
        signUpConfig: {},
        confirmSignUpConfig: {},
        confirmSignInConfig: {},
        forgotPasswordConfig: {}
      };
      return Object.assign(defaults, this.authOptions || {})
    }
  },
  async mounted() {
    AmplifyEventBus.$on('localUser', user => {
      this.user = user;
      this.options.signInConfig.email = this.user.email;
      this.options.confirmSignInConfig.user = this.user;
      this.options.confirmSignUpConfig.email = this.user.email;
    });
    AmplifyEventBus.$on('authState', data => {
      this.displayMap = this.updateDisplayMap(data)
    });
    Auth.currentAuthenticatedUser().then((user) => {
      if (!user) {
        this.displayMap = this.updateDisplayMap('signedOut')
        return;
      }
      this.user = user;
      this.displayMap = this.updateDisplayMap('signedIn');
    }).catch(e => new Error(e));

  },
  methods: {
    updateDisplayMap: state => {
      return {
        showSignIn: state === 'signedOut',
        showSignUp: state === 'signUp',
        showConfirmSignUp: state === 'confirmSignUp',
        showConfirmSignIn: state === 'confirmSignIn',
        showForgotPassword: state === 'forgotPassword',
        showSignOut: state === 'signedIn',
        showMfa: state === 'setMfa'
      }
    },
    setError: function(e) {
      this.error = e.message || e;
      this.logger.error(this.error)
    }
  }
}
</script>