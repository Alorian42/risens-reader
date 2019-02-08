<template>
  <div id="app">
    <div class="text-center controls head">
      <arrow
      :isRight=false
      @click.native="prevPage">
      </arrow>
      <page-selector
        :pages="links"
        :currentPageProp="currentPage"
      ></page-selector>
      <arrow
      :isRight=true
      @click.native="nextPage">
      </arrow>
    </div>
    <div class="text-center content">
      <page 
          v-for="link in availLinks" 
          v-bind:key="link.id"
          :src="link.link"
          v-show="link.id == currentPage"
          @click.native="imageOnClick">
      </page>
    </div>
    <div class="text-center controls bottom">
      <arrow
      :isRight=false>
      </arrow>
      <page-selector
        :pages="links"
        :currentPageProp="currentPage"
      ></page-selector>
      <arrow
      :isRight=true>
      </arrow>
    </div>
  </div>
</template>

<script>
import 'whatwg-fetch';
import Page from './Page.vue';
import PageSelector from './PageSelector';
import Arrow from './Arrow';

export default {
  name: 'app',
  components: {
    Page,
    PageSelector,
    Arrow
  },
  data () {
    return {
      currentPage: 0,
      links: [],
      availLinks: [],
    }
  },
  computed: {
    numberOfPages: function() {
      return this.links.length;
    }
  },
  mounted() {
    this.$on('ChangePage', page => {
        this.currentPage = page;
        this.computeAvailLinks();
    });


    // TO DO load chapter by ID
    this.loadChapter(1050);
  },
  methods: {
    loadChapter: function(id) {
      //fetch('https://risens.team/risensteam/api/chapter.php?id=' + id)
      fetch('fake.html')
      .then(function(response) {
        if (response.ok) {
          return response.json();
        }
        else {
          //TO DO repeat
        }
      }).then((json) => {
        json.forEach((elem, index) => {
          this.links.push({
            link: elem,
            id: index,
          });
        });
        this.availLinks = this.links.slice(0, 3);
        this.currentPage = this.links[0].id;
      })
    },
    nextPage: function() {
      if (this.currentPage < this.numberOfPages - 1) {
        this.currentPage++;
        this.computeAvailLinks();
        this.smoothScroll();
      } 
      else {
        // TO DO Next Chapter
      }
    },
    prevPage: function() {
      if (this.currentPage > 0) {
        this.currentPage--;
        this.computeAvailLinks();
        this.smoothScroll();
      }
      else {
        // TO DO Prev Chapter
      }
    },
    computeAvailLinks: function() {
      let startAvail = this.currentPage;
      let endAvail = startAvail + 3;
      if (startAvail == 1) {
        startAvail = 0;
        endAvail = 4;
      }
      else if (startAvail != 0) {
        startAvail -= 2;
      }

      this.availLinks = this.links.slice(startAvail, endAvail);
    },
    imageOnClick: function(e) {
      if (e.offsetX === undefined ||
          e.path === undefined ||
          !Array.isArray(e.path) ||
          e.path[0] === undefined) {
        return;
      }
      const xAxisClick = e.offsetX;
      const imageWidth = e.path[0].width;

      if (xAxisClick >= Math.floor(imageWidth / 2)) {
        this.nextPage();
      }
      else {
        this.prevPage();
      }
    },
    smoothScroll: function(){
      let currentScroll = document.documentElement.scrollTop || document.body.scrollTop;
      if (currentScroll > 0) {
          window.requestAnimationFrame(this.smoothScroll);
          window.scrollTo (0,currentScroll - (currentScroll/5));
      }
    },
  },
}
</script>

<style lang="scss">
#app, body {
  padding: 0;
  margin: 0;
  width: 100vw;
}

.ten {
  height: 5vh;
}

.text-center {
  text-align: center;
}

.controls {
  background-color: #eb5424;
}

.bottom {
  width: 100%;
}

.head {
  position: absolute;
  top: 0;
  width: 100%;
}

.content {
  margin-top: 10vh;
}
</style>
