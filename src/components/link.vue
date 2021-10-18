<template>
  <a :href="link" class="text-blue-600 underline">
    <slot/>
  </a>
</template>

<script>
export default {
  props: {
      link: String,
  }
}
</script>
