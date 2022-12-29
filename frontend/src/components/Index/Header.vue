<template>
    <div id="header" class="logo">
      <div v-html="core.description"></div>
    </div>
</template>

<script>
export default {
  name: 'Header',
    computed: {
      core() {
          return this.$store.getters.core
      }
    }
}
</script>
