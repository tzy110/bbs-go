<template>
  <section class="main">
    <div class="container main-container left-main">
      <div class="left-container">
        <div class="main-content">
          <topics-nav :nodes="nodes" :current-node-id="node.nodeId" />
          <topic-list :topics="topicsPage.results" :show-ad="true" />
          <pagination
            :page="topicsPage.page"
            :url-prefix="'/topics/node/' + node.nodeId + '?p='"
          />
        </div>
      </div>
      <div class="right-container">
        <site-notice />
        <tweets-widget :tweets="newestTweets" />
        <score-rank :score-rank="scoreRank" />
        <friend-links :links="links" />
      </div>
    </div>
  </section>
</template>

<script>
import SiteNotice from '~/components/SiteNotice'
import ScoreRank from '~/components/ScoreRank'
import FriendLinks from '~/components/FriendLinks'
import TopicsNav from '~/components/TopicsNav'
import TopicList from '~/components/TopicList'
import TweetsWidget from '~/components/TweetsWidget'
import Pagination from '~/components/Pagination'

export default {
  components: {
    SiteNotice,
    ScoreRank,
    FriendLinks,
    TopicsNav,
    TopicList,
    TweetsWidget,
    Pagination
  },
  async asyncData({ $axios, params, query }) {
    const [
      node,
      nodes,
      topicsPage,
      scoreRank,
      links,
      newestTweets
    ] = await Promise.all([
      $axios.get('/api/topic/node?nodeId=' + params.nodeId),
      $axios.get('/api/topic/nodes'),
      $axios.get('/api/topic/node/topics', {
        params: {
          nodeId: params.nodeId,
          page: query.p || 1
        }
      }),
      $axios.get('/api/user/score/rank'),
      $axios.get('/api/link/toplinks'),
      $axios.get('/api/tweet/newest')
    ])
    return {
      node,
      nodes,
      topicsPage,
      scoreRank,
      links,
      newestTweets
    }
  },
  methods: {
    twitterCreated(data) {
      if (this.topicsPage) {
        if (this.topicsPage.results) {
          this.topicsPage.results.unshift(data)
        } else {
          this.topicsPage.results = [data]
        }
      }
    }
  },
  head() {
    return {
      title: this.$siteTitle(this.node.name + ' - 话题'),
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: this.$siteDescription()
        },
        { hid: 'keywords', name: 'keywords', content: this.$siteKeywords() }
      ]
    }
  }
}
</script>

<style lang="scss" scoped></style>
