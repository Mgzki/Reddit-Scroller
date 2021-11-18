<template>
  <div class="my-2 col-span-10 sm:col-span-5 md:col-span-3 xl:col-span-2 bg-gray-100 rounded-xl shadow-lg" >
      <!-- Post Title -->
      <div class="flex py-1 bg-gray-200 h-16 rounded-t-lg text-center px-2">
        <a :href="toLink" target="_blank" class="font-semibold truncate my-auto">{{title}}</a>
      </div>

      <!-- Content Display -->
        <!-- Gallery -->
        <div v-if="contentType === 'gallery'" class="flex justify-center overflow-hidden flex-row relative" style="height:24rem">
          <!-- Back Button -->
          <BackButton v-if="imageIndex > 0" @click="prevGalleryImage"/>
          <!-- Image -->
          <a :href="mediaLink" target="_blank"><img :src="gallery[imageIndex]" class="flex-1 h-full"></a>
          <!-- Next Button -->
          <NextButton v-if="imageIndex < gallery.length - 1" @click="nextGalleryImage" />
          <!-- gallery length overlay -->
          <div class="absolute bottom-0 bg-gray-800 border-gray-500 border text-white opacity-70 px-2 mb-1">
            {{imageIndex + 1}} / {{gallery.length}}
          </div>
        </div>

        <!-- Image -->
        <div v-else-if="contentType === 'image'" class="flex justify-center overflow-hidden flex-row" style="height:24rem">
          <a :href="mediaLink" target="_blank"><img :src="content" class="flex-1 h-full"></a>
        </div>

        <!-- Video -->
        <div v-else-if="isVideo" class="flex justify-center overflow-hidden flex-row" style="height:24rem">
          <!-- <button v-show="!isClickedToLoad" @click="loadVideo" width="max" class="w-full" style="height:24rem">
            <img :src="this.post.preview.images[0].source.url.replace(/amp;/g,'')" class="w-full" height="100%">
          </button> -->
          <video :src="content" controls>
            <p>Your browser doesn't support HTML5 video. Here is a <a :href="mediaLink">link to the video</a> instead.</p>
          </video>
        </div>

        <!-- Embedded videos -->
        <div v-else-if="contentType === 'embedded'" class="flex justify-center overflow-hidden flex-row">

          <!-- No Thumbnail -->
          <template v-if="content === 'no thumbnail'">
            <button v-show="!isClickedToLoad" @click="loadVideo" width="max" class="w-full flex justify-center items-center bg-gray-500" style="height:24rem">
              <img src="https://www.svgrepo.com/show/13672/play-button.svg" width="50" height="100%">
            </button>
            <iframe v-show="isClickedToLoad" :src="this.getTwitchClip" frameborder="0" allowfullscreen="true" scrolling="no" height="384" width="100%"></iframe>
          </template>

          <!-- Has Thumbnail -->
          <template v-else>
            <button  v-show="!isClickedToLoad" @click="loadVideo" width="max" class="w-full" style="height:24rem">
              <img :src="this.post.preview.images[0].resolutions[numPreviewResolutions - 1].url.replace(/amp;/g,'')" class="w-full" height="100%">
            </button>

            <!-- Video frame for thumbnail-->
            <iframe v-show="isClickedToLoad" :src="content" frameborder="0" allowfullscreen="true" scrolling="no" height="384" width="100%"></iframe>

          </template>
        </div>

        <!-- NonEmbeddable Video -->
        <!-- Display thumbnail and link to video -->
        <div v-else-if="content === 'nonEmbeddable'" class="flex justify-center overflow-hidden flex-row" style="height:24rem">
          <a :href="this.mediaLink" target="_blank">
            <img :src="this.post.thumbnail">
          </a>
        </div>

        <!-- Tweet -->
        <div v-else-if="contentType === 'tweet'" class="overflow-y-scroll text-left px-1 flex" style="height:24rem">
          <a :href="content[1]" target="_blank" class="text-sm my-auto mx-auto py-4 px-4 bg-blue-200 rounded-xl">Tweet by {{content[0][0]}}</a>
        </div>

        <!-- Text -->
        <div v-else class="overflow-y-scroll text-left px-1 " style="height:24rem">
          <a :href="medialink" target="_blank" class="text-sm my-auto ">{{selfText}}</a>
        </div>

      <!-- Author in Subreddit -->
      <p class="text-sm align-center bg-gray-200 h-12 rounded-b-lg pt-1">
        <Link @click="fetchFromLink(true)" class="cursor-pointer">{{author}}</Link>
        in <Link @click="fetchFromLink(false)" class="cursor-pointer">r/{{subreddit}}</Link>
      </p>
  </div>
</template>

<script>
import Link from '../components/link.vue'
import BackButton from '../components/back-button.vue'
import NextButton from '../components/next-button.vue'
export default {
  components: {
    Link,
    BackButton,
    NextButton,
  },
  props: {
    post: Object
  },
  data () {
    return {
      contentType: null, // video, selftext, link (to video, image, gallery)
      isClickedToLoad: false,
      content: null,
      imageIndex: 0,
      gallery: null,
      selfText: null,
      video: null,
      title: '',
      author: '',
      subreddit: '',
      mediaLink: '',
      preview: '',
      // test: '',
      domain: 'localhost', // helper for when &parent=domain is required for an embedded video
    }
  },
  methods: {
    populateData() {
      this.title = this.post.title;
      this.author = this.post.author;
      this.subreddit = this.subredditStr;
      this.mediaLink = this.post.url;
      if (this.isSelfText) {
        this.selfText = this.getSelfText
      } else {
        this.selfText = false;
        let temp = this.getContentType
        this.contentType = temp[1]; // for determining how/what to display
        this.content = temp[0];
        this.gallery = this.isGallery ? this.getGallery : false
        this.video = this.isVideo;
      }
    },

    fetchFromLink(author) {
      author ? this.$emit("fetchFromLink", {url: this.author, isAuthor: author}) : this.$emit("fetchFromLink", {url: this.subreddit, isAuthor:author})
    },

    nextGalleryImage() {
      if (this.imageIndex < this.gallery.length - 1) {
        this.imageIndex++;
      }
    },

    prevGalleryImage() {
      if (this.imageIndex > 0) {
        this.imageIndex--;
      }
    },

    loadVideo() {
      // this.test = this.content;
      this.isClickedToLoad = !this.isClickedToLoad
    },

    // Helper function for certain posts that don't have media/preview data
    hasVideoExtension(url) {
      url.match(/.*.(mp4|gifv|h.264|AAC|webm)/i)
      return ''
    },
  },

  created () {
    this.populateData();
  },

  computed: {
    getContentType: function () {
      // handle videos by domain and 
      
      let type = null;
      try {
      if (this.isSelfText) return this.getSelfText;
      else if (this.isGallery) {
        return [this.getGallery,'gallery'];
      }
      else if(this.post.domain === "i.redd.it") {
        
        return [this.post.preview.images[0].resolutions[this.numPreviewResolutions - 1].url.replace(/amp;/g,''), 'image']
      }
      
      else if (this.post.domain === "imgur.com" | this.post.domain === "i.imgur.com") {
        return this.hasPreviewProperty 
          ? [this.post.preview.images[0].resolutions[this.numPreviewResolutions - 1].url.replace(/amp;/g,''), 'image'] 
          : [this.post.url, 'video']
      }
      else if (this.post.domain === "clips.twitch.tv") {
        if (this.hasPreviewProperty) {
          return [this.getTwitchClip,'embedded']
        } 
        return ['no thumbnail','embedded']
      }
      else if (this.post.domain === "twitter.com") {
            return this.getTwitterPost
      }
      
      
      else if (Object.prototype.hasOwnProperty.call(this.post,'post_hint')) {
        type = this.post.post_hint
        if (type === 'link' | type === 'rich:video' | type === 'hosted:video') {
          // Video
          
          if (this.post.domain === "youtube.com" | this.post.domain === "youtu.be") {
            let url = this.getYoutubeUrl
            if (url ==='nonEmbeddable') return ['', 'nonEmbeddable']
            return [url, 'embedded'];
          }
          
          // if (Object.prototype.hasOwnProperty.call(this.post.preview, 'reddit_video_preview') | (Object.prototype.hasOwnProperty.call(this.post.media, 'reddit_video'))){
        
          // if (Object.prototype.hasOwnProperty.call(this.post, 'media')){
          //   //Linked Video
          //   if (Object.prototype.hasOwnProperty.call(this.post.media, 'reddit_video')){
          //     return [this.post.media.reddit_video.fallback_url,'video'];
          //   }
          // }
          // Post has preview
          if (Object.prototype.hasOwnProperty.call(this.post, 'preview')) {

            // Post has video preview
            if (Object.prototype.hasOwnProperty.call(this.post.preview, 'reddit_video_preview')) {
              return [this.post.preview.reddit_video_preview.fallback_url,'video'];
            }

            // Post has image preview and is enabled
            // else if (this.post.preview.enabled) {
            //   return [this.post.preview.images[0].resolutions[this.numPreviewResolutions - 1].url.replace(/amp;/g,''),'image']
            // }
          }
          
          if (Object.prototype.hasOwnProperty.call(this.post, 'preview')) {
            if (Object.prototype.hasOwnProperty.call(this.post.preview, 'images')) {
              return [this.post.preview.images[0].resolutions[this.numPreviewResolutions - 1].url.replace(/amp;/g,''), 'image']
            } 
            // return 0
          }

      }}
      
      if (Object.prototype.hasOwnProperty.call(this.post, 'thumbnail')) {
        if (this.post.thumbnail !== 'default') return [this.post.thumbnail, 'image'] 
      } // assume it's an image for now
       // assume image, only gets here when no other image data is available
      } catch (error){console.log(error, this.post.title, this.post.id)}
      return ['https://i.stack.imgur.com/y9DpT.jpg','image']
    },

    hasPreviewProperty: function () {
      return Object.prototype.hasOwnProperty.call(this.post, 'preview')
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

    isVideo: function() {
      let videoTypes = ['rich:video', 'video', 'hosted:video'];
      for (let type in videoTypes) {
        if (videoTypes[type] === this.contentType) return true;
      } return false
    },

    getPreview: function () {
      if (this.gallery) {
        return this.gallery
      }
      else if (Object.prototype.hasOwnProperty.call(this.post, "preview")) {
        if (this.post.preview.enabled) {
          //takes lowest resolution image to fit display div with reasonable quality
          return this.post.preview.images[0].resolutions[this.numPreviewResolutions - 1].url.replace(/amp;/g,'')
        } else if (this.isVideo && Object.prototype.hasOwnProperty.call(this.post.preview, "reddit_video_preview")) {
          return this.post.preview.reddit_video_preview.fallback_url
        }
      }
      return this.post.thumbnail
    },

    getTwitchClip: function () {
      // pulls media source and converts into iframe embeddable format
      let clipId = this.post.url.match(/tv\/.*/)[0].replace(/tv\//,'')
      return 'https://clips.twitch.tv/embed?clip=' + clipId + '&parent=' + this.domain
    },

    getTwitterPost: function () {
      // get user 
      let user = this.post.media.oembed.author_url.toLowerCase().match(/[^/]+(?=\/$|$)/g)
      return [[user,this.post.media.oembed.url],'tweet'];
    },

    getYoutubeUrl: function () {
      let domain = this.post.domain
      let vidId = '';
      if (this.post.url.match(/clip\/.*/)) {
        return 'nonEmbeddable' // for youtube.com/clips/ links which have no embeddable information
      }
      domain === 'youtu.be' 
        ? vidId = this.post.url.match(/be\/.*/)[0].replace(/be\//,'') // youtu.be domain videos
        : vidId = this.post.url.match(/v=.*/)[0].replace(/v=/,'') // youtube.com domain videos
      return 'https://youtube.com/embed/' + vidId + '?feature=oembed&enablejsapi=1'
    },

    // Returns number of preview resolutions for a post
    numPreviewResolutions: function () {
      return this.post.preview.images[0].resolutions.length
    },

    // Returns number of preview resolutions for an image in a gallery
    numGalleryImageResolutions: function () {
      let ids = this.getGalleryImageIds
      let len = ids.length - 1
      return this.post.media_metadata[ids[len]].p.length 
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

    // Returns array of unique image ids in a gallery
    // Helper function for pulling media_metadata by unique id
    getGalleryImageIds: function () {
      let ids = []
      for (let item in this.post.gallery_data.items) {
        ids.push(this.post.gallery_data.items[item].media_id)
      } return ids
    },

    // Returns the gallery
    getGallery: function () {
      let images = [];
      // Get image ID from gallery data to pull url for each image from media_metadata
      let ids = this.getGalleryImageIds;
      for (let id in ids) {
        let realId = ids[id]
        // Push to gallery if image was not deleted
        try {
          images.push(this.post.media_metadata[realId].p[this.numGalleryImageResolutions - 1].u.replace(/amp;/g,''))
        } catch (error) {
          console.log(this.post.title)
          console.log(error)
        }
        
      }
      return images;
    },
      
  }
}
</script>

<style>

</style>