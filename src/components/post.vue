<template>
  <div class="my-2 col-span-10 sm:col-span-5 md:col-span-3 xl:col-span-2 bg-gray-100 rounded-xl shadow-lg" >
      <!-- Post Title -->
      <div class="flex py-1 bg-gray-200 h-16 rounded-t-lg text-center px-2">
        <a :href="toLink" class="font-semibold truncate my-auto">{{title}}</a>
      </div>

      <!-- Content Display -->
        <!-- Non Text Content -->
        <div v-if="gallery" class="flex justify-center overflow-hidden flex-row relative" style="height:24rem">
          <button @click="prevGalleryImage" class="absolute left-0 text-red-500 bg-blue-300" style="top:50%">prev</button>
          <a :href="mediaLink"><img :src="preview[imageIndex]" class="flex-1 h-full"></a>
          <button @click="nextGalleryImage" class="absolute right-0 text-red-500 bg-blue-300" style="top:50%">next</button>
        </div>
        <!-- Image -->
        <div v-else-if="!isSelfText && !video" class="flex justify-center overflow-hidden flex-row" style="height:24rem">
          <a :href="mediaLink"><img :src="preview" class="flex-1 h-full"></a>
        </div>
        <!-- Video -->
        <div v-else-if="video" class="flex justify-center overflow-hidden flex-row" style="height:24rem">
          <video :src="preview" controls>
            <p>Your browser doesn't support HTML5 video. Here is a <a :href="mediaLink">link to the video</a> instead.</p>
          </video>
        </div>
        <!-- Text -->
        <div v-else class="overflow-y-scroll text-left px-1 " style="height:24rem">
          <p class="text-sm my-auto ">{{selfText}}</p>
        </div>
        
      
      <!-- Author in Subreddit -->
      <p class="text-sm align-center bg-gray-200 h-8 rounded-b-lg pt-1">
        
        By <Link :link="toAuthor">{{author}}</Link>
        in <Link :link="toSubreddit">{{subreddit}}</Link>
        
        {{contentType}}
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
      contentType: null, // video, selftext, link (to video, image, gallery)
      imageIndex: 0,
      gallery: null,
      selfText: null,
      video: null,
      title: '',
      author: '',
      subreddit: '',
      mediaLink: '',
      preview: '',
    }
  },
  methods: {
    populateData() {
      this.contentType = this.getContentType;

      this.title = this.post.title;
      this.author = this.post.author;
      this.subreddit = this.subredditStr;
      this.gallery = this.isGallery ? this.getGallery : false
      this.selfText = this.isSelfText ? this.getSelfText : false
      this.mediaLink = this.post.url;
      this.video = this.isVideo;
      this.preview = this.getPreview;
    },
    nextGalleryImage: function () {
      if (this.imageIndex < this.gallery.length - 1) {
        this.imageIndex++;
      }
    },
    prevGalleryImage: function () {
      if (this.imageIndex > 0) {
        this.imageIndex--;
      }
    }
  },

  created () {
    this.populateData();
  },

  computed: {
    // Returns ContentType (Video, Link, Image)
    // post_hint doesn't exist for galleries?
    // post_hint sometimes isn't there for self-posts either
    
    // if not selfText, check contentType, if not Video or image, check type of link (gallery, video, image)
    // handle if gallery
    getContentType: function () {
      return this.post.post_hint
    },

    // Returns the subreddit with the first character capitalized
    subredditStr: function () {
      return this.post.subreddit[0].toUpperCase() + this.post.subreddit.substring(1);
    },

    // Returns link to the author
    toAuthor: function() {
      return "https://reddit.com/u/" + this.author;
    },

    // Returns link to the subreddit
    toSubreddit: function() {
      return "https://reddit.com/r/" + this.subreddit;
    },

    // Returns link to the post
    toLink: function() {
      return "https://reddit.com" + this.post.permalink;
    },
    isVideo: function(){
      //this.post.preview.reddit_video_preview.fallback_url
      //this.post.media.reddit_video.fallback_url
      if (Object.prototype.hasOwnProperty.call(this.post, "preview")) {
        if (!this.post.preview.enabled && !this.selfText) {
          return true
        }
      } return false
    },
    getPreview: function () {
      if (this.gallery) {
        return this.gallery
      }
      else if (Object.prototype.hasOwnProperty.call(this.post, "preview")) {
        if (this.post.preview.enabled) {
          //takes lowest resolution image to fit display div with reasonable quality
          return this.post.preview.images[0].resolutions[this.numResolutions - 1].url.replace(/amp;/g,'')
        } else if (this.isVideo && Object.prototype.hasOwnProperty.call(this.post.preview, "reddit_video_preview")) {
          return this.post.preview.reddit_video_preview.fallback_url
        }
        return "https://images.unsplash.com/photo-1508921912186-1d1a45ebb3c1?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=687&q=80"
      }
      return this.post.thumbnail
    },

    // Returns number of preview resolutions for a post
    numResolutions: function () {
      return this.post.preview.images[0].resolutions.length
    },
    
    // Returns boolean for if the post is a selfText
    isSelfText: function () {
      return this.post.is_self
    },

    // Returns the value of the selfText
    getSelfText: function () {
      return this.post.selftext;
    },

    // Returns boolean for if the post is a gallery
    isGallery: function () {
      return (Object.prototype.hasOwnProperty.call(this.post, "is_gallery"))
    },

    // Returns the gallery
    getGallery: function () {
      let images = [];
      // Get image ID from gallery data to pull url for each image from media_metadata
      for (let item in this.post.gallery_data.items) {
        let id = this.post.gallery_data.items[item].media_id
        images.push(this.post.media_metadata[id].p[3].u.replace(/amp;/g,''))
      }
      return images;
    },

    // Checks for valid embeddable video file extension within preview
    hasVideoExtension: function () {
      // match(/.*.(mp4|gifv|h.264|AAC|webm)/i)
      // console.log(this.post.preview.images[0].source.url.match(/.*.(mp4|gifv|h.264|AAC|webm)/i),this.permalink)
      // return this.post.preview.images[0].source.url.match(/.*.(mp4|gifv|h.264|AAC|webm)/i)
      return 0
    },

    
  }
}
</script>

<style>

</style>