<template>
<div class="xs-flex xs-flex-justify-space-around full-height browse" :style="`margin-top:${navbarheight}px`">
 

<div class="xs-flex xs-flex-justify-space-around xs-flex-wrap blog-wrapper clearfix col lg-col-9 md-col-8 sm-col-7">

      <article v-if="items2[pi]" v-for="(p,pi) in items2" :key="p.pi" class="xs-border xs-p2 col xs-col-6 xs-pr1 article-wrapper">

        <div class="item clearfix gutters">

          <div class="article-info col lg-col-4 md-col-6 sm-col-12">
            <h2 class="xs-text-1 md-text-2 lg-text-1 xs-mb2 slab"><nuxt-link :to="p._path">{{p.title}}</nuxt-link></h2>
            <div class="article-meta"></div>
            <p class="lg-text-4 md-text-4" v-if="pi < 1">{{p.teaser}}</p>

            <nuxt-link :to="p._path" class="xs-block xs-text-2 xs-my2">Read More</nuxt-link>
          </div>

          <nuxt-link class="xs-text-center col lg-col-8 md-col-6" :to="p._path">           
            <img style="width:100%;" v-if="p.thumbnail" :src="require('~/static'+p.thumbnail)">             
          </nuxt-link>

        </div>

      </article>

</div>

<div class="sidebar-wrapper xs-flex-1 col lg-col-3 md-col-4 sm-col-5">Sidebar</div>








</div>
</template>
<script>

  import MdWrapper from "~/components/MdWrapper";

export default {
  props: [],
  data() {
    return {
      currentPage: null,
      pageNumbers: [],
      pageNumberCount: 0,
      items2: [],
      query: 1,
      busy: false,
      count: 0
    };
  },
  methods: {
    pageCheck() {
      if (this.allitems.length > 12) {
        this.$store.commit("paginateOn", true);
        this.$store.commit("resultsLength", this.allitems.length);
      } else if (this.allitems.length < 12) {
        this.$store.commit("paginateOff", false);
      } else {
        this.$store.commit("paginateOff", false);
      }
    },

    loadMore() {
      this.count = this.offset;
      if (this.total > this.count && this.busy == false) {
        this.busy = true;

     
          this.items2.splice(0);
          for (var i = 0, j = 12; i < j; i++) {
            let api = this.allitems[this.count];

            this.items2.push(api);
            this.count++;
          }

          this.busy = false;
        
      }
    },

    onResize(event) {
      this.navHeight();
    },

    navHeight() {
      if (process.browser) {
        var height = document.getElementById("navbar").clientHeight;

        this.$store.commit("SET_NAVHEIGHT", height - 1);
      }
    }
  },
  watch: {
    // whenever question changes, this function will run
    $route({ params, query }) {
      if (this.$route.query.page > 1) {
        this.loadMore();
        this.navHeight();
        this.pageCheck();
      } else if (this.$route.query.page == null) {
        this.$route.query.page = 1;
        this.loadMore();
          this.navHeight();
        this.pageCheck();
      } else {
        this.loadMore();
          this.navHeight();
        this.pageCheck();
      }
    },
    queryParam: function() {
      this.loadMore();
    }
  },
  computed: {
    allitems() {
      return this.$store.state.blogPosts;
    },
    getLayout() {
      if (this.$store.state.siteInfo.altlayout == false ) {
        return 'BaelGrid'
      } else if (this.$store.state.siteInfo.altlayout == true ) {
        return 'FullGrid'
      }
    },
    offset() {
      if (this.queryParam > 1) {
        return Number(this.queryParam - 1) * 12;
      } else {
        return 0;
      }
    },
    prevpage() {
      var prev = Number(this.queryParam) - 1;
      return prev;
    },
    nextpage() {
      var next = Number(this.queryParam) + 1;
      return next;
    },
    navbarheight() {
      return this.$store.state.navheight;
    },
    total() {
      return this.allitems.length;
    },

    queryParam() {
      if (this.$route.query.page == null) {
        return 1;
      } else {
        return Number(this.$route.query.page);
      }
    }
  },
  components: {
    MdWrapper
  },
  updated() {
    this.$nextTick(() => {
      this.pageCheck();
      this.navHeight();
      this.$store.commit("SET_GRIDOFFSET", this.offset);
    });
  },
  mounted() {
    if (process.browser) {
      this.loadMore();

      this.$nextTick(() => {
        this.navHeight();
        this.pageCheck();
        window.addEventListener("resize", this.onResize);
      });
    }
  },
  beforeDestroy() {
    // Unregister the event listener before destroying this Vue instance
    window.removeEventListener("resize", this.onResize);
  }
};
</script>

<style sdoped>

.article-wrapper {
flex-basis:50%;
}
.article-wrapper:first-of-type {
flex-basis:100%;
}

.article-wrapper > div > a {
flex-direction:column;
}
</style>
