<template>
  <div>
    <h1>Create a minified URL</h1>
    <form v-on:submit.prevent="createRedirect">
      <input type="text" :disabled="creating" v-model.trim="short" placeholder="Shortlink" />
      <input type="text" :disabled="creating" v-model.trim="url" placeholder="Target URL" />
      <button type="submit" :disabled="creating">{{ creating ? "Creating..." : "Create" }}</button>
      <p>{{ error }}</p>
      <p>{{ notice }}</p>
    </form>
  </div>
</template>

<script>
import AWS from 'aws-sdk';

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
  methods: {
    s3: function() {
      const config = new AWS.Config({ credentials: this.credentials, region: 'us-east-1' });
      const s3 = new AWS.S3(config);
      return s3;
    },
    createRedirect: function() {
      this.creating = true;
      this.notice = null;

      const target = this.url;
      const short = this.short;
      const bucket = this.bucket;

      this.s3().putObject({
        ACL: 'public-read',
        Body: "",
        Bucket: bucket,
        Key: short,
        ContentLength: 0,
        WebsiteRedirectLocation: target
      }, (err, success) => {
        this.creating = false;
        if (err) {
          this.error = `Error creating redirect: ${err}`;
        } else {
          this.notice = `Created ${bucket}/${short} to ${target}`;
          this.url = null;
          this.short = null;
        }
      });
    }
  }
}
</script>
