<template>
  <div class="pageBody customStyles">
    <Banner :id="tab.key" :page="thisPage[0]"/>
    <div class="navBanners">
      <NavBanner class="leftNavBanner" useCustomStyles="true" />
      <div class="navSpacer" />
      <NavBanner class="rightNavBanner" useCustomStyles="true" />
    </div>
    <div class="pageFrame">
      <div class="pageContent leftPage">
          <div class="mediaContent">
              <h2 class="pageTitle" v-text="thisPage[0].title" />
              <div class="media">
                  <Media v-for="url in pageMedia[0]" :key="url" :url="url" class="panel"/>
              </div>
          </div>      
          <div class="textContent">
              <TextContent :key="thisPage[0].pageId" :content="thisPage[0].content"/>
              <PageNav :thisPage="thisPage[0]" :nextPages="nextPagesArray[0]" ref="pageNav1"/>
          </div>
      </div>
      <div class="pageContent rightPage">
          <div class="mediaContent">
              <h2 class="pageTitle" v-html="thisPage[1].title" />
              <div class="media">
                  <Media v-for="url in pageMedia[1]" :key="url" :url="url" class="panel"/>
              </div>
          </div>      
          <div class="textContent">
              <TextContent :key="thisPage[1].pageId" :content="thisPage[1].content"/>
              <PageNav :thisPage="thisPage[1]" :nextPages="nextPagesArray[1]" ref="pageNav2"/>
          </div>
      </div>
    </div>
    <PageFooter pageWidth="1660px" />
  </div>
</template>

<script>
// @ is an alias to /src
import NavBanner from '@/components/UIElements/NavBanner.vue'
import Banner from '@/components/Page/PageBanner.vue'
import Media from '@/components/UIElements/MediaEmbed.vue'
import TextContent from '@/components/Page/PageText.vue'
import PageNav from '@/components/SpecialPages/x2ComboNav.vue'
import PageFooter from '@/components/Page/PageFooter.vue'

export default {
  name: 'x2Combo',
  props: [
    'tab', 'routeParams'
  ],
  components: {
    NavBanner, Banner, Media, TextContent, PageNav, PageFooter
  },
  data: function() {
    return {
      preload: []
    }
  },
  computed: {
    pageNum() {
      return this.$isVizBase(this.routeParams.base) ? this.$vizToMspa(this.routeParams.base, this.routeParams.p).p : this.routeParams.p
    },
    storyNum() {
      return this.$getStory(this.pageNum)
    },
    thisPage() {
      let thisPageId = this.pageNum
      let leftPageId, rightPageId
      if (parseInt(thisPageId) % 2 == 0) {
        leftPageId = thisPageId
        rightPageId = this.$archive.mspa.story[thisPageId].next[0]
      }
      else {
        leftPageId = this.$archive.mspa.story[thisPageId].previous
        rightPageId = thisPageId
      }

      let page = [this.$archive.mspa.story[leftPageId], this.$archive.mspa.story[rightPageId]]
      return page
    },
    pageMedia() {
      this.preload = []
      let preloadPages = [
        this.nextPagesArray[1][0],
        this.$archive.mspa.story[this.nextPagesArray[1][0].next[0]]
      ]
      preloadPages.forEach(page => {
        page.media.forEach(media => {
          if (/(gif|png)$/i.test(media)) {
            let img = new Image()
            img.src = this.$mspaURL(media)
            this.preload.push(img)
          }
        })
      })
      return [this.thisPage[0].media, this.thisPage[1].media]
    },
    nextPagesArray() {
      console.log(`${this.tab.url} - ${this.thisPage[1].title}`)
      let nextPages = [[], []]
      this.thisPage[0].next.forEach(nextID => {
        nextPages[0].push(this.$archive.mspa.story[nextID])
      })
      this.thisPage[1].next.forEach(nextID => {
        nextPages[1].push(this.$archive.mspa.story[nextID])
      })
      return nextPages
    }
  },
  methods:{
    keyNavEvent(dir) {
      if (dir == 'left' && this.$parent.$el.scrollLeft == 0) this.$pushURL(this.$refs.pageNav1.backUrl)
      else if (dir == 'right' && this.$parent.$el.scrollLeft + this.$parent.$el.clientWidth == this.$parent.$el.scrollWidth) this.$pushURL(this.$refs.pageNav2.nextUrl(this.nextPagesArray[1][0]))
    }
  }
}
</script>

<style scoped lang="scss">
.pageBody {
  color: var(--font-default);
  background: var(--page-pageBody);
  
  margin: 0 auto;

  > img {
    align-self: center;
  }

  .navBanners{
    width: 1660px;
    display: flex;
    flex-flow: row;
    justify-content: center;
    background: var(--nav-bg);
    .navSpacer {
      padding: 0 50px;
    }
    ::v-deep nav {
      width: auto;
    }
  }
  .pageFrame {
    background: var(--page-pageFrame);

    padding-top: 7px;
    padding-bottom: 23px;
    width: 1660px !important;

    flex: 0 1 auto;
    display: flex;
    justify-content: center;
    align-items: flex-start;

    .pageContent {
      background: var(--page-pageContent);
      
      max-width: 950px;
      display: flex;
      flex: 0 1 auto;
      align-items: center;
      flex-flow: column;
      &.leftPage {
        margin-right: 30px;
      }

      .mediaContent {
        display: flex;
        align-items: center;
        flex-flow: column;

        h2.pageTitle {
          max-width: 590px;
          text-align: center;
          line-height: 1.1;
          font-size: 32px;
          padding: 15px 0;
        }

        .media{
          display: flex;
          align-items: center;
          flex-flow: column;

          .panel {
            ::v-deep {
              image-rendering: pixelated;
            }
            &:not(:last-child) {
              margin-bottom: 17px;
            }
          }            
        }
      }

      .textContent{
        margin-top: 30px;
        width: 600px;
        display: flex;
        flex-direction: column;
      }
    }	
  }

}


</style>

