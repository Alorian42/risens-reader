<template>
  <span class="text-center arrow" v-bind:style="arrowColor">
    <i class="fas" v-bind:class="arrow" v-bind:title="alt"></i>
  </span>
</template>
<script>
export default {
  props: {
    isRight: {
      type: Boolean,
    },
    disabled: {
      type: Boolean
    },
    doubleArrows: {
      type: Boolean
    },
    alt: {
      type: String
    },

  },

  data () {
    return {
      
    }
  },
  computed: {
    arrow: function() {
      return this.isRight ?
        this.doubleArrows ? 'fa-angle-double-right' : 'fa-angle-right' : 
        this.doubleArrows ? 'fa-angle-double-left' : 'fa-angle-left';
    },
    arrowColor: function() {
      return {
        color: this.disabled ? '#e67b7b' : 'white',
        cursor: this.disabled ? 'not-allowed' : 'pointer',
      };
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
