<template>
  <div>
    <h1>Enter your AWS credentials</h1>
    <form v-on:submit.prevent="login">
      <input type="text" name="key" v-model.trim="key" placeholder="API Key" />
      <input type="password" name="secret" v-model.trim="secret" placeholder="API Secret" />
      <button :disabled="loggingIn" type="submit">{{this.loggingIn ? "Logging in..." : "Log in"}}</button>
      <p>{{ error }}</p>
    </form>
  </div>
</template>

<script>
import AWS from 'aws-sdk';

export default {
  name: 'login',
  props: {
    onLogin: {
      type: Function,
      required: true
    }
  },
  data () {
    return {
      key: "",
      secret: "",
      loggingIn: false,
      error: null
    }
  },
  methods: {
    login: function() {
      this.loggingIn = true;
      const credentials = new AWS.Credentials(this.key, this.secret);
      const region = 'us-east-1';
      const config = new AWS.Config({ credentials, region });
      const s3 = new AWS.S3(config);

      // Get index to test credentials
      s3.getObject({Bucket: 'zpl.in', Key: 'index.html'}, (err, data) => {
        if (err) {
          this.error = "Invalid credentials";
          this.loggingIn = false;
        } else {
          this.onLogin(credentials);
        }
      })
    }
  }
}
</script>
