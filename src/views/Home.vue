<template>
  <Layout>
    <slot>
      <div class="flex max-w-screen-2xl justify-center mx-auto px-2 ">
        <div class=" grid grid-cols-10 grid-flow-row gap-2 pt-6 overflow-y-scroll">
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
          <Post/>
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
}
</script>

