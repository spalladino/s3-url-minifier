<template>
  <div>
    <h1>Existing shortlinks</h1>
    <button v-on:click.stop.prevent="refresh" :disabled="refreshing">{{ refreshing ? "Refreshing..." : "Refresh" }}</button>
    <p>{{ error }}</p>
    <ul>
      <li v-for="link in links">
        <a v-bind:href="href(link)">{{ link }}</a>
      </li>
    </ul>
  </div>
</template>

<script>
import AWS from 'aws-sdk';
import _ from 'lodash';

export default {
  name: 'listing',
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
      refreshing: false,
      error: null,
      links: []
    }
  },
  created: function() {
    this.s3 = new AWS.S3({credentials: this.credentials, region: 'us-east-1'});
    this.refresh();
  },
  methods: {
    href: function(short) {
      return `http://${this.bucket}/${short}`;
    },
    refresh: function() {
      this.refreshing = true;
      this.error = null;

      this.s3.listObjectsV2({
        Bucket: this.bucket,
      }, (err, data) => {
        this.refreshing = false;
        if (err) {
          this.error = `Error loading links: ${err}`;
        } else {
          this.links = _(data.Contents).map('Key').reject(key => (key == 'index.html' || _.startsWith(key, 'admin/'))).value();
        }
      });
    }
  }
}
</script>
