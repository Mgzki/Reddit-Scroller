<template>
  <div class="my-2 col-span-10 sm:col-span-5 md:col-span-3 xl:col-span-2 bg-gray-100 rounded-xl shadow-lg" >
      <!-- Post Title -->
      <div class="flex py-1 bg-gray-200 h-16 rounded-t-lg text-center px-2">
        <a :href="toLink" class="font-semibold truncate my-auto">{{title}}</a>
      </div>

      <!-- Content Display -->
        <!-- Gallery -->
        <div v-if="contentType === 'gallery'" class="flex justify-center overflow-hidden flex-row relative" style="height:24rem">
          <button v-if="imageIndex > 0" @click="prevGalleryImage" class="absolute left-0 text-red-500 bg-blue-300" style="top:50%">prev</button>
          <a :href="mediaLink"><img :src="preview[imageIndex]" class="flex-1 h-full"></a>
          <button v-if="imageIndex < gallery.length - 1" @click="nextGalleryImage" class="absolute right-0 text-red-500 bg-blue-300" style="top:50%">next</button>
        </div>
        <!-- Image -->
        <div v-else-if="contentType === 'image'" class="flex justify-center overflow-hidden flex-row" style="height:24rem">
          <a :href="mediaLink"><img :src="preview" class="flex-1 h-full"></a>
        </div>
        <!-- Video -->
        <div v-else-if="isVideo" class="flex justify-center overflow-hidden flex-row" style="height:24rem">
          
          <video :src="content" controls>
            <p>Your browser doesn't support HTML5 video. Here is a <a :href="mediaLink">link to the video</a> instead.</p>
          </video>
        </div>
        <!-- Twitch -->
        <div v-else-if="contentType === 'twitch'" class="flex justify-center overflow-hidden flex-row">
          <button v-show="!isClickedToLoad" @click="loadVideo" width="max" class="w-full" style="height:24rem">
            <img :src="this.post.thumbnail" class="w-full" height="100%">
          </button>
          <iframe v-show="isClickedToLoad" :src="test" frameborder="0" allowfullscreen="true" scrolling="no" height="384" width="100%"></iframe>
        </div>
        <!-- Tweet -->
        <div v-else-if="contentType === 'tweet'" class="overflow-y-scroll text-left px-1 flex" style="height:24rem">
          <a :href="content[1]" class="text-sm my-auto mx-auto py-4 px-4 bg-blue-200 rounded-xl">Tweet by {{content[0][0]}}</a>
        </div>
        <!-- Text -->
        <div v-else class="overflow-y-scroll text-left px-1 " style="height:24rem">
          <p class="text-sm my-auto ">{{selfText}}</p>
        </div>
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
      test: '',
    }
  },
  methods: {
    populateData() {
      let temp = this.getContentType
      this.contentType = temp[1]; //tweet, video, gallery, image, twitch, rich:video
      this.content = temp[0];
      
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
    },
    loadVideo: function () {
      this.test = this.content;
      this.isClickedToLoad = !this.isClickedToLoad
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
    // getContentType: function () {
    //   let type = null;
    //   if (this.isSelfText) return 'selfText';
    //   else if (this.isGallery) {
    //     return 'gallery'
    //   }
    //   else if (Object.prototype.hasOwnProperty.call(this.post,'post_hint')) {
    //     type = this.post.post_hint
    //     if (type === 'link') {
    //       // Video
    //       // Embedded Twitch
    //       if (this.post.domain === "clips.twitch.tv") {
    //         return 'twitch'
    //       }
    //       // if (Object.prototype.hasOwnProperty.call(this.post.preview, 'reddit_video_preview') | (Object.prototype.hasOwnProperty.call(this.post.media, 'reddit_video'))){
    //       if (Object.prototype.hasOwnProperty.call(this.post, 'preview') | (Object.prototype.hasOwnProperty.call(this.post, 'media'))){
    //         //Linked Video
    //         if (Object.prototype.hasOwnProperty.call(this.post.preview, 'reddit_video_preview') | (Object.prototype.hasOwnProperty.call(this.post.media, 'reddit_video'))){
    //           return 'linkedVideo'
    //         }
    //       }
    //       if (Object.prototype.hasOwnProperty.call(this.post, 'preview')) {
    //         return 'linkedImage'
    //       }
    //       // photo link
    //     }
    //   } return type
    // },
    getContentType: function () {
      let type = null;
      if (this.isSelfText) return this.getSelfText;
      else if (this.isGallery) {
        return [this.getGallery,'gallery']
      }
      else if (Object.prototype.hasOwnProperty.call(this.post,'post_hint')) {
        type = this.post.post_hint
        if (type === 'link') {
          // Video
          // Embedded Twitch
          if (this.post.domain === "clips.twitch.tv") {
            return [this.getTwitchClip,'twitch'];
          }
          if (this.post.domain === "twitter.com") {
            return this.getTwitterPost
          }
          // if (Object.prototype.hasOwnProperty.call(this.post.preview, 'reddit_video_preview') | (Object.prototype.hasOwnProperty.call(this.post.media, 'reddit_video'))){
        
          if (Object.prototype.hasOwnProperty.call(this.post, 'media')){
            //Linked Video
            if (Object.prototype.hasOwnProperty.call(this.post.media, 'reddit_video')){
              return [this.post.media.reddit_video.fallback_url,'video'];
            }
          }
          // Post has preview
          if (Object.prototype.hasOwnProperty.call(this.post, 'preview')) {

            // Post has video preview
            if (Object.prototype.hasOwnProperty.call(this.post.preview, 'reddit_video_preview')) {
              return [this.post.preview.reddit_video_preview.fallback_url,'video'];
            }

            // Post has image preview and is enabled
            else if (this.post.preview.enabled) {
              return [this.post.preview.images[0].resolutions[this.numResolutions - 1].url.replace(/amp;/g,''),'image']
            }
          }
          
          if (Object.prototype.hasOwnProperty.call(this.post, 'preview')) {
            return [this.post.thumbnail, 'image']
          }
          // photo link
        }
      } return ['this.post.thumbnail','image']
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
      let videoTypes = ['rich:video', 'video'];
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
          return this.post.preview.images[0].resolutions[this.numResolutions - 1].url.replace(/amp;/g,'')
        } else if (this.isVideo && Object.prototype.hasOwnProperty.call(this.post.preview, "reddit_video_preview")) {
          return this.post.preview.reddit_video_preview.fallback_url
        }
      }
      return this.post.thumbnail
    },

    getTwitchClip: function () {
      // pulls media source and converts into iframe embeddable format
      let slashIndex = 15; //index of the end of 'clips.twitch.tv' for use with inserting '/' to create valid url
      let url = this.post.secure_media_embed.content.replace(/amp;/g,'').match(/url=https[A-z./?=%0-9-&;]*/)[0].split('https')[1].slice(0,-7).replace(/%[0-9][A-z]/g,'');
      return 'https://' + [url.slice(0,slashIndex),'/embed?clip=',url.replace(/%[0-9][A-z]/g,'').slice(slashIndex)].join('') + '&parent=localhost'

    },

    getTwitterPost: function () {
      // get user 
      let user = this.post.media.oembed.author_url.toLowerCase().match(/[^/]+(?=\/$|$)/g)
      return [[user,this.post.media.oembed.url],'tweet'];
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
  }
}
</script>

<style>

</style>