<template>
  <div>
    <h1>Create a minified URL</h1>
    <form v-on:submit.prevent="createRedirect">
      <input type="text" :disabled="creating" v-model.trim="short" placeholder="Shortlink" v-on:input="checkShortlink" />
      <input type="text" :disabled="creating" v-model.trim="url" placeholder="Target URL" />
      <button type="submit" :disabled="creating">{{ creating ? "Creating..." : "Create" }}</button>
      <p>{{ error }}</p>
      <p>{{ notice }}</p>
    </form>
  </div>
</template>

<script>
import AWS from 'aws-sdk';
import _ from 'lodash';

export default {
  name: 'minifier',
  props: {
    credentials: {
      type: Object,
      required: true
    },
    bucket: {
      type: String,
      required: true
    }
  },
  data () {
    return {
      short: null,
      url: null,
      error: null,
      notice: null,
      creating: false
    }
  },
  created: function() {
    this.s3 = new AWS.S3({credentials: this.credentials, region: 'us-east-1'});
  },
  methods: {
    checkShortlink: _.debounce(function() {
      const short = this.short;
      this.s3.getObject({Bucket: this.bucket, Key: short}, (err, data) => {
        if (!err && short == this.short) {
          this.notice = `Shortlink ${short} already exists to ${data.WebsiteRedirectLocation}`
        } else {
          this.notice = null;
        }
      })
    }, 1000),
    createRedirect: function() {
      this.creating = true;
      this.notice = null;

      const target = this.url;
      const short = this.short;

      this.s3.putObject({
        ACL: 'public-read',
        Body: "",
        Bucket: this.bucket,
        Key: short,
        ContentLength: 0,
        WebsiteRedirectLocation: target
      }, (err, success) => {
        this.creating = false;
        if (err) {
          this.error = `Error creating redirect: ${err}`;
        } else {
          this.notice = `Created ${this.bucket}/${short} to ${target}`;
          this.url = null;
          this.short = null;
        }
      });
    }
  }
}
</script>
