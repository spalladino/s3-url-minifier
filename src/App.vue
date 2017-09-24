<template>
  <div id="app">
    <div v-if="loggedIn">
      <Minifier :credentials="credentials" :bucket="bucket" />
      <hr />
      <Listing :credentials="credentials" :bucket="bucket" />
    </div>
    <Login v-else :on-login="handleLogin" />
  </div>
</template>

<script>
import Login from './components/GoogleLogin.vue';
import Minifier from './components/Minifier.vue';
import Listing from './components/Listing.vue';

export default {
  name: 'app',
  components: {
    Login,
    Minifier,
    Listing
  },
  data: function() {
    return {
      loggedIn: false,
      credentials: null,
      bucket: process.env.AWS_S3_BUCKET
    }
  },
  methods: {
    handleLogin: function(credentials) {
      this.loggedIn = true;
      this.credentials = credentials;
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
