<template>
  <div class="next" ref="next"></div>
</template>
<script>
export default {
  props: {
    page: {
      type: Number,
      default: 0
    }
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
    let h = this.$refs['next'].parentElement.clientHeight;
    if (h == 0) {
      h = "100%";
    }
    else {
      h = h + 'px';
    }
    
    this.$refs['next'].style.height = h;
  },
  mounted () {
    let h = this.$refs['next'].parentElement.clientHeight;
    console.log(h);
    this.$refs['next'].style.height = h + 'px';
    if (h == 0) {
      this.$forceUpdate();
    }
  }
}
</script>
<style lang="scss">
.next {
  cursor: pointer;
  background-color: green;
  opacity: 0.6;
  width: 50%;
  height: 100%;
  position: absolute;
  top: 0%;
  left: 50%;
}
</style>
