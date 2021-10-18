<template>
  <div class="my-2 col-span-10 sm:col-span-5 md:col-span-3 xl:col-span-2 bg-gray-100 rounded-xl shadow-lg" >
      <!-- Post Title -->
      <div class="flex py-1 bg-gray-200 h-16 rounded-t-lg text-center px-2">
        <a :href="toLink" class="font-semibold truncate my-auto">{{title}}</a>
      </div>

      <!-- Content Display -->
      <container>
        <!-- Non Text Content -->
        <div v-if="!isSelfText" class="flex justify-center overflow-hidden flex-row" style="height:24rem">
          <a :href="mediaLink"><img :src="preview" class="flex-1 h-full"></a>
        </div>
        <!-- Text Content -->
        <div v-else class="overflow-y-scroll" style="height:24rem">
          <p class="text-sm">{{selfText}}</p>
        </div>
      </container>
      
      <!-- Author in Subreddit -->
      <p class="text-sm align-center bg-gray-200 h-8 rounded-b-lg pt-1">
        By <Link :link="toAuthor">{{author}}</Link>
        in <Link :link="toSubreddit">{{subreddit}}</Link>
      </p>
  </div>
</template>

<script>
import Link from '../components/link.vue'
export default {
  components: {
    Link,
  },
  props: {
    post: Object
  },
  data () {
    return {
      media: null,
      gallery: null,
      selfText: null,
      title: '',
      thumbnail: '',
      author: '',
      subreddit: '',
      mediaLink: '',
      preview: '',
    }
  },
  methods: {
    populateData() {
      this.title = this.post.title;
      this.thumbnail = this.post.thumbnail;
      this.author = this.post.author;
      // Capitalize first letter of subreddit
      this.subreddit = this.subredditStr;
      this.preview = this.getPreview;
      this.mediaLink = this.post.url;

      this.selfText = this.isSelfText;
    }
  },
  created () {
    this.populateData();
  },
  computed: {
    subredditStr: function () {
      return this.post.subreddit[0].toUpperCase() + this.post.subreddit.substring(1);
    },
    toAuthor: function() {
      return "https://reddit.com/u/" + this.author;
    },
    toSubreddit: function() {
      return "https://reddit.com/r/" + this.subreddit;
    },
    toLink: function() {
      return "https://reddit.com" + this.post.permalink;
    },
    getThumbnail: function () {
      return 0
    },
    test: function () {
      return (this.post.preview) ? this.post.preview.images[0].source.url : 'aaaaa'
    },
    getPreview: function () {
      if (Object.prototype.hasOwnProperty.call(this.post, "preview")) {
        if (this.post.preview.enabled) {
          //takes lowest resolution image to fit display div with reasonable quality
          return this.post.preview.images[0].resolutions[this.numResolutions - 3].url.replace(/amp;/g,'')
        } else if (!this.post.preview.enabled) {
          return this.post.preview.reddit_video_preview.fallback_url
        }
        return "https://images.unsplash.com/photo-1508921912186-1d1a45ebb3c1?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=687&q=80"
      }
      // video preview (this.post.preview.enabled = false)
      // this.post.preview.reddit_video_preview.fallback_url

      // const str = this.post.media_metadata.ph5om3maw6u71.s.u;
      // return str.replace(/amp;/g,'')
      return this.post.thumbnail
    },
    numResolutions: function () {
      return this.post.preview.images[0].resolutions.length
    },
    
    isSelfText: function () {
      if (this.post.is_self) {
        return this.post.selftext;
      } return null
    }
  }
}
</script>

<style>

</style>