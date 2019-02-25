<template>
  <div id="app">
    <div class="text-center controls head">
      <arrow
        alt="Предыдущая страница"
        :isRight=false
        :disabled="this.currentPage === 0 && this.chapters[this.currentChapter - 1] === undefined"
        @click.native="prevPage">
      </arrow>
      <page-selector
        :pages="links"
        :currentPageProp="currentPage"
      ></page-selector>
      <arrow
        alt="Следующая страница"
        :isRight=true
        :disabled="this.currentPage === this.numberOfPages - 1 && this.chapters[this.currentChapter + 1] === undefined"
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
        alt="Предыдущая глава"
        :isRight=false
        :doubleArrows=true
        :disabled="this.chapters[this.currentChapter - 1] === undefined"
        @click.native="prevChapter">
      </arrow>
      
      <div
        @click="alertError"
      >
        <i title="Сообщить об ошибке" class="fas fa-exclamation-triangle alert-icon"></i>
      </div>
      <chapter-selector
        :chapters="chapters"
        :currentChapterProp="currentChapter"
      ></chapter-selector>
      <!--<div
        @click="downloadChapter"
      >
        <i title="Скачать" class="fas fa-cloud-download-alt download-icon"></i>
      </div>-->
      <arrow
        alt="Следующая глава"
        :isRight=true
        :doubleArrows=true
        :disabled="this.chapters[this.currentChapter + 1] === undefined"
        @click.native="nextChapter">
      </arrow>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import 'whatwg-fetch';
import VueRouter from 'vue-router'
import Page from './Page.vue';
import PageSelector from './PageSelector';
import ChapterSelector from './ChapterSelector';
import Arrow from './Arrow';
import JSZip from 'jszip';
import { saveAs } from 'file-saver';

Vue.use(VueRouter);
var router = new VueRouter({
    mode: 'history',
    routes: []
});

export default {
  name: 'app',
  router,
  components: {
    Page,
    PageSelector,
    ChapterSelector,
    Arrow
  },
  data () {
    return {
      currentPage: 0,
      links: [],
      availLinks: [],
      id: false,
      currentChapter: 0,
      chapters: [],
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
    this.$on('ChangeChapter', chapter => {
        this.currentChapter = parseInt(chapter);
        if (this.chapters[this.currentChapter] !== undefined) {
          this.loadChapter(this.chapters[this.currentChapter].id);
        }
    });

    
    window.onkeyup = e => {
      let key = e.keyCode ? e.keyCode : e.which;
      
      if (key == 39) {
        this.nextPage();
      } 
      else if (key == 37) {
        this.prevPage();
      }
    }

    const query = this.$route.query;
    let chapterId = false;

    if (query.chapter_id !== null && query.chapter_id !== undefined) {
      chapterId = query.chapter_id;
    }

    if (query.id !== null && query.id !== undefined) {
      this.id = query.id;
      this.getChapters(this.id, chapterId);
    }
  },
  methods: {
    loadChapter: function(id) {
      fetch('https://risens.team/risensteam/api/chapter.php?id=' + id)
      .then(function(response) {
        if (response.ok) {
          return response.json();
        }
        else {
          setTimeout(() => this.loadChapter(this.id), 2000);
        }
      }).then((json) => {
        this.links = [];

        json.forEach((elem, index) => {
          this.links.push({
            link: elem,
            id: index,
          });
        });
        
        this.availLinks = this.links.slice(0, 3);
        this.currentPage = this.links[0].id;
      });
    },
    nextPage: function() {
      if (this.currentPage < this.numberOfPages - 1) {
        this.currentPage++;
        this.computeAvailLinks();
        this.smoothScroll();
      } 
      else {
        this.nextChapter();
      }
    },
    prevPage: function() {
      if (this.currentPage > 0) {
        this.currentPage--;
        this.computeAvailLinks();
        this.smoothScroll();
      }
      else {
        this.prevChapter();
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
    smoothScroll: function() {
      let currentScroll = document.documentElement.scrollTop || document.body.scrollTop;
      if (currentScroll > 0) {
          window.requestAnimationFrame(this.smoothScroll);
          window.scrollTo (0,currentScroll - (currentScroll/5));
      }
    },
    getChapters: function(id, chapterId = false) {
      fetch('https://risens.team/risensteam/api/manga.php?id=' + id)
      .then(function(response) {
        if (response.ok) {
          return response.json();
        }
        else {
          setTimeout(() => this.getChapters(this.id), 2000);
        }
      }).then((json) => {
        let tempChapters = [];
        json.forEach((elem, index) => {
          tempChapters.push({
            name: elem.name,
            id: elem.id,
            order: parseFloat(elem.chapter),
          });
        });

        tempChapters.sort((a, b) => {
          if (a.order < b.order) {
            return -1;
          }
          if (a.order > b.order) {
            return 1;
          }

          return 0;
        });

        this.chapters = tempChapters;

        if (chapterId !== false) {
          const tempIndex = this.chapters.findIndex(x => x.id == chapterId);
          if (tempIndex !== -1) {
            this.currentChapter = tempIndex;
          }
          else {
            this.currentChapter = this.chapters.length - 1;
          }
        }
        else {
          this.currentChapter = this.chapters.length - 1;
        }

        this.loadChapter(this.chapters[this.currentChapter].id);
      });
    },
    nextChapter: function() {
      let next = this.currentChapter + 1;
      if (this.chapters[next] !== undefined) {
        this.currentChapter = next;
        this.loadChapter(this.chapters[this.currentChapter].id);
      }
    },
    prevChapter: function() {
      let next = this.currentChapter - 1;
      
      if (this.chapters[next] !== undefined) {
        this.currentChapter = next;
        this.loadChapter(this.chapters[this.currentChapter].id);
      }
    },
    downloadChapter: function() {
      let urls = [];
      let zipName = this.chapters[this.currentChapter].name + '.zip';

      this.links.forEach(elem => {
        urls.push(elem.link);
      });

      var zip = new JSZip();

      var promise = function(url, index) {
        return new Promise(function(resolve) {
          var httpRequest = new XMLHttpRequest();
          httpRequest.open("GET", url);
          httpRequest.responseType = 'blob';

          httpRequest.onload = function() {
            let reader = new FileReader();
            reader.onloadend = function() {
              let res = this.result;
              let ext = 'jpg';
              if (res.includes('data:image/png;base64')) {
                  ext = 'png';
              }
              res = res.split(',', 2)[1];

              let stringIndex = index + 1;

              stringIndex = (stringIndex <= 9) ? '0' + stringIndex : stringIndex;
              zip.file(stringIndex + '.' + ext, res, {base64: true});
              resolve();
            }
            reader.readAsDataURL(this.response);
          }

          httpRequest.send();
        });
      };

      Promise.all(urls.map(function(url, index) {
          return promise(url, index)
        }.bind(this)))
        .then(function() {
          zip.generateAsync({
              type: "blob"
          })
          .then(function(content) {
            saveAs(content, zipName);
          });
      });
    },
    alertError: function() {
      const message = prompt('С какой ошибкой вы столкнулись');
      if (message === null) {
        return;
      }
      
      fetch('https://risens.team/risensteam/api/message.php?message=' + encodeURIComponent(message) +
            '&chapter_id=' + this.chapters[this.currentChapter].id + '&manga_id=' + this.id)
      .then(function(response) {
        alert('Спасибо!');
      });
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
  height: 50px;

  display: flex;
  justify-content: center;
  align-items: center;
}

.bottom {
  position: absolute;
  bottom: 0;
  width: 100%;
}

.head {
  position: absolute;
  top: 0;
  width: 100%;
}

.content {
  margin-top: 50px;
  height: calc(100vh - 100px);
}

.alert-icon {
  margin-right: 10px;
  color: white;
  font-size: 25px;
  cursor: pointer;
}

.download-icon {
  margin-left: 10px;
  color: white;
  font-size: 25px;
  cursor: pointer;
}
</style>
