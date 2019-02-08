<template>
  <select @change="pageSelected()" v-model="currentPage">
    <option 
          v-for="page in pages" 
          v-bind:key="page.id"
          :value="page.id"> {{ page.id + 1 }}</option>
  </select>
</template>
<script>
export default {
  props: {
    pages: {
      type: Array,
      default: []
    },
    currentPageProp: {
      type: Number,
      default: 0,
    }
  },

  data () {
    return {
      tempValue: 0,
    }
  },
  computed: {
    currentPage:   {
      get: function() {
        return this.currentPageProp;
      },
      set: function(newValue) {
        this.tempValue = newValue;
      }
    }
  },
  methods: {
    pageSelected: function(e) {
      if (this.tempValue != this.currentPageProp) {
        this.$parent.$emit('ChangePage', this.tempValue);
      }
    }
  },
  updated () {
    
  },
  mounted () {
    
  }
}
</script>
<style lang="scss">
select {
  padding: 10px;
  margin: 5px;
}
</style>
