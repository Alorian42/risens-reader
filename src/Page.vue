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
  max-width: 100vw;
  position: relative;
}

.staticMode img{
  max-height: calc(100vh - 100px);
}
</style>
