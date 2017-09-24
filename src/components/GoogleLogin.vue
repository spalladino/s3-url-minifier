<template>
  <div>
    <div id="google-signin"></div>
  </div>
</template>

<script>

export default {
  name: 'login',
  props: {
    onLogin: { type: Function, required: true }
  },
  data () {
    return {
      error: null
    }
  },
  created: function() {
    gapi.load('auth2', () => {
      console.log(this)
      gapi.auth2.init({
        client_id: process.env.GOOGLE_CLIENT_ID,
        hosted_domain: process.env.GOOGLE_DOMAIN,
        ux_mode: 'popup'
      }).then(() => {
        gapi.signin2.render('google-signin', {
          onsuccess: this.handleLogin,
          onfailure: this.handleFailure,
          longtitle: true
        });
      });
    });
  },
  methods: {
    handleFailure: function() {
      this.error = `Please log in with your ${this.domain} user`;
    },
    handleLogin: function(googleUser) {
      const token = googleUser.getAuthResponse().id_token;
      AWS.config.region = 'us-east-1';
      const credentials = new AWS.CognitoIdentityCredentials({
        IdentityPoolId: process.env.AWS_IDENTITY_POOL_ID,
        Logins: {
          'accounts.google.com': token
        }
      });
      AWS.config.credentials = credentials;
      this.onLogin(credentials);
    }
  }
}
</script>
