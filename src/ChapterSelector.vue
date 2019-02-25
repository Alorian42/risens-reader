<template>
  <v-select 
    maxHeight="250px"
    :onChange="chapterSelected" 
    v-model="currentChapter"
    :options="options"></v-select>
</template>
<script>
import vSelect from 'vue-select'

export default {
  props: {
    chapters: {
      type: Array,
      default: []
    },
    currentChapterProp: {
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
    currentChapter: {
      get: function() {
        if (this.chapters[this.currentChapterProp] !== undefined) {
          return {
            value: this.currentChapterProp,
            label: this.chapters[this.currentChapterProp].name
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

      this.chapters.forEach((elem, index) => {
        array.push({
          value: index,
          label: elem.name
        });
      });

      return array;
    }
  },
  methods: {
    chapterSelected: function(val) {
      if (val === null || val === undefined) {
        val = {
          value: this.chapters.length - 1
        }
      }

      if (val.value != this.currentChapterProp) {
        this.$parent.$emit('ChangeChapter', val.value);
      }
    }
  },
  updated () {
    
  },
  mounted () {
    
  },
  components: {
    vSelect
  }
}
</script>
<style lang="scss">
select {
  padding: 10px;
  margin: 5px;
}

.bottom .v-select {
  width: 150px;
  background-color: white;
}
.bottom .dropdown-menu {
  bottom: 100%;
  top: auto !important;
}
.clear {
  display: none;
}

.dropdown {
  border-radius: 5px;
}
</style>
