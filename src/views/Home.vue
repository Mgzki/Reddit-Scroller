<template>
  <!-- Nav -->
  <section>
    <nav class="bg-gray-600 py-4 grid grid-cols-12 gap-2">

      <!-- Hamburger menu icon -->
      <!-- <div class="flex justify-center mx-auto">
          <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="24" height="24" viewBox="0 0 24 24" style=" fill:#000000;" class="my-auto ml-2">
        <path d="M 2 5 L 2 7 L 22 7 L 22 5 L 2 5 z M 2 11 L 2 13 L 22 13 L 22 11 L 2 11 z M 2 17 L 2 19 L 22 19 L 22 17 L 2 17 z"></path>
      </svg>
      </div> -->

      <!-- Subreddit Search -->
      <input v-model="subreddit" v-on:keyup.enter="updateUrlSubreddit" type="text" class="px-2 py-2 sm:py-4 col-start-3 sm:col-start-2 col-span-4 sm:col-span-5 opacity-50 focus:opacity-100 rounded-md shadow-lg" placeholder="r/">
      <!-- User Search -->
      <input v-model="username" v-on:keyup.enter="updateUrlUsername" type="text" class="px-2 py-2 sm:py-4 col-span-4 sm:col-span-5 opacity-50 focus:opacity-100 rounded-md shadow-lg" placeholder="user/">
    </nav>
  </section>

  <!-- Body -->
  <div class="bg-gray-400 overflow-y-scroll" style="height: 91vh">
    <div class="flex max-w-screen-2xl justify-center mx-auto px-2 ">
        <div v-if="posts.length > 0" class="grid grid-cols-6 sm:grid-cols-10 md:grid-cols-9 lg:grid-cols-12 xl:grid-cols-10 grid-flow-row gap-2 pt-6 ">
          <template v-for="post in posts" :key="post.data.id">
            <Post :post="post.data" @fetchFromLink="fetchFromLink($event)"/>
          </template>
        </div>
        <div v-else class="mt-20 bg-gray-100 rounded-xl shadow-lg max-w-2xl ">
            <p class="py-4 px-12">Sorry, no posts here!</p>
        </div>
      </div>
  </div>
  <footer>
    <button @click="loadMore"> More </button>
  </footer>
</template>

<script>
import Post from '../components/post.vue'
export default {
  components: {
    Post,
  },
  data () {
    return {
      posts: [],
      load: [],
      url: 'https://www.reddit.com/',
      urlTail: 'r/wallpapers',
      subreddit: '',
      username: '',
      limit: 50,

    }
  },
  methods: {
    async fetchData () {
      // let temp = []
      let url = this.buildUrl(true)
      try {
        const response = await fetch(url);
        this.posts = await response.json().then(function(json) {
          // Ensures only posts (kind = t3) are sent through
          let children = [];
          for (let child in json.data.children) {
            let fetchedData = json.data.children[child]
            if (fetchedData.kind === 't3' && fetchedData.data.removed_by_category === null) // is a post and hasn't been removed
              children.push(fetchedData)
          }
          return children
        });
      } catch (error) {
        console.log(error);
      }
    },
    async loadMore() {
      let after = this.posts[this.posts.length - 1].data.id
      let url = this.buildAfterUrl()
      try{
        const response = await fetch(url + after);
        this.load = await response.json().then(function(json) {
          return json.data.children
        }); 
        for (const post in this.load) {
          let fetchedData = this.load[post]
          if (fetchedData.kind === 't3' && fetchedData.data.removed_by_category === null) { // is a post and hasn't been removed
            this.posts.push(fetchedData)
          }
        }
      } catch (error) {
        console.log(error);
      }
    },

    // Helper function for creating fetch url
    buildUrl (limit) {
      return limit ? this.url + this.urlTail + '.json?limit=' + this.limit : this.url + this.urlTail + '.json';
    },

    // Helper function for loading more posts
    buildAfterUrl () {
      return this.buildUrl(false) + '?after=t3_'
    },

    updateUrlSubreddit() {
      try {
        this.urlTail = 'r/' + this.subreddit;
        this.username = '';
        this.fetchData();
      } catch (error) {
        console.log('Error finding the subreddit')
        console.log(error)
      }
      // this.urlTail.match(/(user|r)\/.*/) ? this.fetchData() : null
    },

    updateUrlUsername() {
      try {
        this.urlTail = 'user/' + this.username;
        this.subreddit = '';
        this.fetchData();
      } catch (error) {
        console.log('Error finding the user')
        console.log(error)
      }
      // this.urlTail.match(/(user|r)\/.*/) ? this.fetchData() : null
    },

    // Helper for navigation between users/subreddits through clicked links
    fetchFromLink(event){
      event.isAuthor ? this.urlTail = 'user/' + event.url : this.urlTail = 'r/' + event.url
      this.fetchData();
    },
  },
  created() {
    this.fetchData();
  },
}
</script>

<style>
  ::-webkit-scrollbar {
    width: 3px;
  }
  /* Track */
  ::-webkit-scrollbar-track {
    background: var(--darkestgray); 
    
    border-radius: 1px;
  }
  
  /* Handle */
  ::-webkit-scrollbar-thumb {
    background: #333;
    
    border-radius: 1px;
  }
  /* Handle on hover */
  ::-webkit-scrollbar-thumb:hover {
    background: none; 
  }
</style> 

