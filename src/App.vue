<template>
  <div id="app" v-bind:class="{ night: forceNightMode, staticMode: staticMode }">
    <loading :active.sync="isLoading" :can-cancel="false" :is-full-page="true"></loading>
    <div class="text-center controls head">
      <arrow
        alt="Предыдущая страница"
        :isRight="false"
        :disabled="this.currentPage === 0 && this.chapters[this.currentChapter - 1] === undefined"
        @click.native="prevPage"
      ></arrow>
      <page-selector :pages="links" :currentPageProp="currentPage"></page-selector>
      <div @click="toggleMode">
        <i title="Сменить режим" class="fas fa-search download-icon"></i>
      </div>
      <arrow
        alt="Следующая страница"
        :isRight="true"
        :disabled="this.currentPage === this.numberOfPages - 1 && this.chapters[this.currentChapter + 1] === undefined"
        @click.native="nextPage"
      ></arrow>
    </div>
    <div class="text-center content">
      <page
        v-for="link in availLinks"
        v-bind:key="link.id"
        :src="link.link"
        v-show="link.id == currentPage"
        @click.native="imageOnClick"
      ></page>
    </div>
    <div class="text-center controls bottom">
      <arrow
        alt="Предыдущая глава"
        :isRight="false"
        :doubleArrows="true"
        :disabled="this.chapters[this.currentChapter - 1] === undefined"
        @click.native="prevChapter"
      ></arrow>

      <div @click="alertError">
        <i title="Сообщить об ошибке" class="fas fa-exclamation-triangle alert-icon"></i>
      </div>
      <chapter-selector :chapters="chapters" :currentChapterProp="currentChapter"></chapter-selector>
      <div @click="downloadChapter">
        <i title="Скачать" class="fas fa-cloud-download-alt download-icon"></i>
      </div>
      <arrow
        alt="Следующая глава"
        :isRight="true"
        :doubleArrows="true"
        :disabled="this.chapters[this.currentChapter + 1] === undefined"
        @click.native="nextChapter"
      ></arrow>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import "whatwg-fetch";
import VueRouter from "vue-router";
import Page from "./Page.vue";
import PageSelector from "./PageSelector";
import ChapterSelector from "./ChapterSelector";
import Arrow from "./Arrow";
import JSZip from "jszip";
import { saveAs } from "file-saver";
import Loading from "vue-loading-overlay";
import "vue-loading-overlay/dist/vue-loading.css";
import Swal from "sweetalert2";
import VueCookies from "vue-cookies";

Vue.use(VueRouter);
Vue.use(VueCookies);
var router = new VueRouter({
  mode: "history",
  routes: []
});

export default {
  name: "app",
  router,
  components: {
    Page,
    PageSelector,
    ChapterSelector,
    Arrow,
    Loading,
    Swal
  },
  data() {
    return {
      currentPage: 0,
      links: [],
      availLinks: [],
      id: false,
      currentChapter: 0,
      chapters: [],
      isLoading: true,
      isVip: false,
      forceNightMode: false,
      staticMode: true,
    };
  },
  computed: {
    numberOfPages: function() {
      return this.links.length;
    }
  },
  mounted() {
    this.$on("ChangePage", page => {
      this.currentPage = page;
      this.computeAvailLinks();
    });
    this.$on("ChangeChapter", chapter => {
      this.currentChapter = parseInt(chapter);
      if (this.chapters[this.currentChapter] !== undefined) {
        this.loadChapter(this.chapters[this.currentChapter].id);
      }
    });

    window.addEventListener("message", event => {
      if (event.origin != "https://risens.team") {
        return;
      }

      if (event.data == "night") {
        this.checkNightMode();
      }
    });

    const n = $cookies.get("night");
    this.forceNightMode = n == "yes";

    window.onkeyup = e => {
      let key = e.keyCode ? e.keyCode : e.which;

      if (key == 39) {
        this.nextPage();
      } else if (key == 37) {
        this.prevPage();
      }
    };

    fetch("https://risens.team/risensteam/api/get_info.php")
      .then(response => {
        return response.json();
      })
      .then(json => {
        if (json.group == 1 || json.group == 6 || json.group == 10) {
          this.isVip = true;
        }
      });

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
    checkNightMode: function() {
      const n = $cookies.get("night");
      this.forceNightMode = n == "yes";
    },
    loadChapter: function(id) {
      this.isLoading = true;
      fetch("https://risens.team/risensteam/api/chapter.php?id=" + id)
        .then(function(response) {
          if (response.ok) {
            return response.json();
          } else {
            setTimeout(() => this.loadChapter(this.id), 2000);
          }
        })
        .then(json => {
          this.links = [];

          json.forEach((elem, index) => {
            this.links.push({
              link: elem,
              id: index
            });
          });

          this.availLinks = this.links.slice(0, 3);
          this.currentPage = this.links[0].id;
          this.isLoading = false;
          this.statAction();
        });
    },
    nextPage: function() {
      if (this.currentPage < this.numberOfPages - 1) {
        this.currentPage++;
        this.computeAvailLinks();
        this.smoothScroll();
      } else {
        this.nextChapter();
      }
    },
    prevPage: function() {
      if (this.currentPage > 0) {
        this.currentPage--;
        this.computeAvailLinks();
        this.smoothScroll();
      } else {
        this.prevChapter();
      }
    },
    computeAvailLinks: function() {
      let startAvail = this.currentPage;
      let endAvail = startAvail + 3;
      if (startAvail == 1) {
        startAvail = 0;
        endAvail = 4;
      } else if (startAvail != 0) {
        startAvail -= 2;
      }

      this.availLinks = this.links.slice(startAvail, endAvail);
    },
    imageOnClick: function(e) {
      if (
        (e.offsetX === undefined && e.layerX) ||
        (e.path === undefined && !e.composedPath && !e.composedPath())
      ) {
        this.nextPage();
        return;
      }
      const path = e.path || (e.composedPath && e.composedPath());
      const xAxisClick = e.offsetX === 0 ? e.layerX : e.offsetX;
      const imageWidth = path[0].width;
      console.log(xAxisClick, imageWidth);

      if (xAxisClick >= Math.floor(imageWidth / 2)) {
        this.nextPage();
      } else {
        this.prevPage();
      }
    },
    smoothScroll: function() {
      let currentScroll =
        document.documentElement.scrollTop || document.body.scrollTop;
      if (currentScroll > 0) {
        window.requestAnimationFrame(this.smoothScroll);
        window.scrollTo(0, currentScroll - currentScroll / 5);
      }
    },
    getChapters: function(id, chapterId = false) {
      this.isLoading = true;
      fetch("https://risens.team/risensteam/api/manga.php?id=" + id)
        .then(function(response) {
          if (response.ok) {
            return response.json();
          } else {
            setTimeout(() => this.getChapters(this.id), 2000);
          }
        })
        .then(json => {
          let tempChapters = [];
          json.forEach((elem, index) => {
            tempChapters.push({
              name: elem.name,
              id: elem.id,
              order: parseFloat(elem.chapter)
            });
          });

          tempChapters.sort((a, b) => {
            if (a.order < b.order) {
              return 1;
            }
            if (a.order > b.order) {
              return -1;
            }

            return 0;
          });

          this.chapters = tempChapters.reverse();

          if (chapterId !== false) {
            const tempIndex = this.chapters.findIndex(x => x.id == chapterId);
            if (tempIndex !== -1) {
              this.currentChapter = tempIndex;
            } else {
              this.currentChapter = this.chapters.length - 1;
            }
          } else {
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
      if (!this.isVip) {
        Swal.fire({
          type: "error",
          title: "Ошибка!",
          text: "Скачка доступна только для наших патронов!",
          footer:
            '<a href="https://patreon.risens.team" target="_blank">Поддержите нас на Патреоне...</a>'
        });

        return;
      }

      this.isLoading = true;
      let urls = [];
      let zipName = this.chapters[this.currentChapter].name + ".zip";

      this.links.forEach(elem => {
        urls.push(elem.link);
      });

      var zip = new JSZip();

      var promise = function(url, index) {
        return new Promise(function(resolve) {
          let myHeaders = new Headers();
          myHeaders.set("Origin", "https://risens.team/");

          const myInit = {
            method: "GET",
            headers: myHeaders,
            mode: "cors",
            cache: "default"
          };

          url = url.replace(
            "https://storage.yandexcloud.net/rt-manga/Manga%2F",
            "https://risens.team/risensteam/vipdl_manga.php?path="
          );

          fetch(url, myInit)
            .then(function(response) {
              return response.blob();
            })
            .then(function(blob) {
              let reader = new FileReader();
              reader.onloadend = function() {
                let res = this.result;
                let ext = "jpg";
                if (res.includes("data:image/png;base64")) {
                  ext = "png";
                }
                res = res.split(",", 2)[1];

                let stringIndex = index + 1;

                stringIndex =
                  stringIndex <= 9 ? "0" + stringIndex : stringIndex;
                zip.file(stringIndex + "." + ext, res, { base64: true });
                resolve();
              };

              reader.readAsDataURL(blob);
            });
        });
      };

      Promise.all(
        urls.map((url, index) => {
          return promise(url, index);
        })
      ).then(() => {
        zip
          .generateAsync({
            type: "blob"
          })
          .then(content => {
            saveAs(content, zipName);
            this.isLoading = false;
          });
      });
    },
    alertError: function() {
      Swal.fire({
        input: "textarea",
        title: "С какой ошибкой вы столкнулись?",
        inputPlaceholder: "Опишите вашу проблему...",
        showCancelButton: true
      }).then(response => {
        const message = response.value;
        if (message) {
          fetch(
            "https://risens.team/risensteam/api/message.php?message=" +
              encodeURIComponent(message) +
              "&chapter_id=" +
              this.chapters[this.currentChapter].id +
              "&manga_id=" +
              this.id
          ).then(function(response) {
            alert("Спасибо!");
          });
        }
      });
    },
    statAction: function() {
      fetch(
        "https://risens.team/risensteam/api/stat.php?type=3&id=" +
          this.chapters[this.currentChapter].id
      );
    },
    toggleMode: function() {
      this.staticMode = !this.staticMode;
    }
  }
};
</script>

<style lang="scss">
#app,
body {
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
  position: fixed;
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

.night .controls {
  background-color: #212529;
}

.night i {
  color: white;
}
</style>
