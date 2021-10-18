<template>
  <Layout>
    <slot>
      <div class="flex max-w-screen-2xl justify-center mx-auto px-2 ">
        <div class="grid grid-cols-6 sm:grid-cols-10 md:grid-cols-9 lg:grid-cols-12 xl:grid-cols-10 grid-flow-row gap-2 pt-6 overflow-y-scroll">
          <template v-for="post in posts" :key="post.data.id">
            <Post :post="post.data"/>
          </template>
        </div>
      </div>
    </slot>
  </Layout>
</template>

<script>
import Layout from '../layouts/default.vue'
import Post from '../components/post.vue'
export default {
  components: {
    Layout,
    Post
  },
  data () {
    return {
      posts: [],
    }
  },
  methods: {
    async fetchData () {
      try {
        const response = await fetch('https://www.reddit.com/r/wallpapers/.json');
        this.posts = await response.json().then(function(json) {
          return json.data.children
        });
      } catch (error) {
        console.log(error);
      }
    }
  },
  created() {
    this.fetchData();
  }
}
</script>

