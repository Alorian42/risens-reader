<template>
  <img :src="src" ref="img">
</template>
<script>
export default {
  props: {
    src: {
      type: String,
    },

  },

  data () {
    return {
        
    }
  },
  computed: {
    
  },
  methods: {

  },
  updated () {
    
  },
  mounted () {
    
  }
}
</script>
<style lang="scss">
img {
  cursor: pointer;
  height: 100%;
  max-width: 100vw;
}
</style>
