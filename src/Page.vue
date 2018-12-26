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
    this.$refs['img'].onload = () => {
      console.log(this.$refs['img'].width, this.$refs['img'].height);
      if (this.$refs['img'].width > this.$refs['img'].height) {
        this.$refs['img'].style.width = "100%";
      }
      else {
        this.$refs['img'].style.height = "100%";
      }
      console.log(this.$refs['img'].width, this.$refs['img'].height);
    };
  }
}
</script>
<style lang="scss">
img {
  
}
</style>
