<template>
  <div class="prev" ref="prev"></div>
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
    let h = this.$refs['prev'].parentElement.clientHeight;
    if (h == 0) {
      h = "100%";
    }
    else {
      h = h + 'px';
    }

    this.$refs['prev'].style.height = h;
  },
  mounted () {
    let h = this.$refs['prev'].parentElement.clientHeight;
    this.$refs['prev'].style.height = h + 'px';

    if (h == 0) {
      this.$forceUpdate();
    }
  }
}
</script>
<style lang="scss">
.prev {
  cursor: pointer;
  background-color: red;
  opacity: 0.6;
  width: 50%;
  height: 100%;
  position: absolute;
  top: 0%;
  left: 0%;
}
</style>
