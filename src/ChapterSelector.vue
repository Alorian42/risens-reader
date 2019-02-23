<template>
  <select @change="chapterSelected()" v-model="currentChapter">
    <option 
          v-for="(chapter, index) in chapters" 
          v-bind:key="chapter.id"
          :value="index"> {{ chapter.name }}</option>
  </select>
</template>
<script>
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
        return this.currentChapterProp;
      },
      set: function(newValue) {
        this.tempValue = newValue;
      }
    }
  },
  methods: {
    chapterSelected: function(e) {
      if (this.tempValue != this.currentChapterProp) {
        this.$parent.$emit('ChangeChapter', this.tempValue);
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
