<template>
  <div id="app">
    <div class="ten">
      11324
    </div>
    <div class="eighty">
      <page 
          v-for="link in availLinks" 
          v-bind:key="link.id"
          :src="link.link"
          v-show="link.id == currentPage">
      </page>
      
      <div class="absolute" ref="arrowsRef">
        <prev @click.native="prevPage" :page="currentPage"></prev>
        <next @click.native="nextPage" :page="currentPage"></next>
      </div>
    </div>
    <div class="ten">
      11324
    </div>
  </div>
</template>

<script>
import 'whatwg-fetch';
import Page from './Page.vue';
import Next from './Next.vue';
import Prev from './Prev.vue';

export default {
  name: 'app',
  components: {
    Page,
    Next,
    Prev,
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
    // TO DO load chapter by ID
    this.loadChapter(1050);

    window.onscroll = (e) => {
      let vertical_position = 0;
      if (pageYOffset)//usual
        vertical_position = pageYOffset;
      else if (document.documentElement.clientHeight)//ie
        vertical_position = document.documentElement.scrollTop;
      else if (document.body)//ie quirks
        vertical_position = document.body.scrollTop;
      
      this.$refs['arrowsRef'].style.top = vertical_position + 'px';
    }
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
      } 
      else {
        // TO DO Next Chapter
      }
    },
    prevPage: function() {
      if (this.currentPage > 0) {
        this.currentPage--;
        this.computeAvailLinks();
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
    }
  },
}
</script>

<style lang="scss">
#app, body {
  padding: 0;
  margin: 0;
  height: 100vh;
}
.absolute {
  position: absolute;
  top: 5vh;
  left: 0;
  height: 90vh;
  width: 100vw;
}
.ten {
  height: 5vh;
}
.eighty {
  height: 90vh;
  text-align: center;
}
</style>
