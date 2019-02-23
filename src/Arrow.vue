<template>
  <span class="text-center arrow">{{ arrow }}</span>
</template>
<script>
export default {
  props: {
    isRight: {
      type: Boolean,
    },

  },

  data () {
    return {
        isRight: false,
    }
  },
  computed: {
    arrow: function() {
      return this.isRight ? '⮕' : '⬅';
    }
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
.arrow {
  display: inline-block;
  width: 20vw;
  font-size: 45px;
  height: 100%;
  line-height: 100%;
  vertical-align: middle;
  cursor: pointer;
  color: white;
}
</style>
