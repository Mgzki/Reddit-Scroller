<template>
  <a :href="link" class="text-blue-600 underline inline-flex">
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
