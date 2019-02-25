<template>
  <v-select-p 
    maxHeight="250px"
    :onChange="pageSelected" 
    v-model="currentPage"
    :options="options"></v-select-p>
</template>
<script>
import vSelectP from 'vue-select'

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
        if (this.pages[this.currentPageProp] !== undefined) {
          return {
            value: this.currentPageProp,
            label: this.pages[this.currentPageProp].id + 1
          };
        }
        else {
          return 0;
        }
      },
      set: function(newValue) {
        this.tempValue = newValue;
      }
    },
    options: function() {
      let array = [];

      this.pages.forEach((elem, index) => {
        array.push({
          value: index,
          label: elem.id + 1
        });
      });

      return array;
    }
  },
  methods: {
    pageSelected: function(val) {
      if (val === null || val === undefined) {
        val = {
          value: 0
        }
      }

      if (val.value != this.currentPageProp) {
        this.$parent.$emit('ChangePage', val.value);
      }
    }
  },
  updated () {
    
  },
  mounted () {
    
  },
  components: {
    vSelectP
  }
}
</script>
<style lang="scss">
select {
  padding: 10px;
  margin: 5px;
}

.head .v-select {
  width: 100px;
  background-color: white;
}
</style>
