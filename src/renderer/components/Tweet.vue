<template>
<li class="list-group-item" >
  <div class="user-content pull-left">
    <img class="img-circle media-object user-thumb" :src="userData.profile_image_url" width="48" height="48" @click="showUserDialog">
    <div class="retweet-user" v-if="retweetUserData">
      <img class="img-circle media-object user-thumb" :src="retweetUserData.profile_image_url" width="24" height="24" @click="showRetweetUserDialog"><span class="icon icon-retweet"></span>
    </div>
  </div>
  <div class="media-body">
    <strong>{{ userData.name }}</strong><span>@{{ userData.screen_name }}</span>
    <tweet-text :text="status.text" :urls="status.getUrls()"></tweet-text>
    <div>
      <ul class="media-list">
        <li class="media" v-for="media in mediaList">
          <img v-bind:src="media.media_thumb" @click="showMediaDialog(media.media_small)">
        </li>
      </ul>
    </div>
    <div>
      {{ dateSt }}
    </div>
    <div class="action-area">
      <span class="icon icon-reply reaction" @click="reply"></span>
      <span class="icon icon-retweet reaction" @click="retweet"></span>
      <span class="icon reaction" v-bind:class="{ 'icon-heart': status.favorited, 'icon-heart-empty': !status.favorited }" @click="like"></span>
    </div>
  </div>
</li>
</template>

<script>
import TweetText from './TweetText';
import moment from 'moment';
import Loading from './Loading';
import TweetStatus from '../data/TweetStatus';
import User from '../data/User';

export default {
  props: ['status'],
  components: {
    Loading: Loading,
    TweetText: TweetText
  },
  computed: {
    userData: function () {
      return this.status.getUser();
    },
    retweetUserData: function () {
      const userData = this.status.getRetweetUser();
      if (userData instanceof User) {
        return userData;
      } else {
        return;
      }
    },
    dateSt: function () {
      return moment(new Date(this.status.created_at)).format("YYYY-MM-DD HH:mm:ss a");
    },
    mediaList: function () {
      return this.status.getMedia();
    }
  },
  methods: {
    showDetail: function () {

    },
    reply: function () {
      this.$store.dispatch('showTweetDialog', { status: this.status });
    },
    retweet: function () {
      this.$store.dispatch('showRetweetDialog', {
        profile_image_url: this.status.getUser().profile_image_url,
        name: this.status.getUser().name,
        screen_name: this.status.getUser().screen_name,
        text: this.status.text,
        date: moment(new Date(this.status.created_at)).format("YYYY-MM-DD HH:mm:ss a"),
        mediaList: this.status.getMedia(),
        target_tweet_id: this.status.id
      });
    },
    like: function () {
      if (this.status.favorited) {
        this.$store.dispatch('destroyLike', { status: this.status });
      } else {
        this.$store.dispatch('createLike', { status: this.status });
      }
    },
    showMediaDialog: function (image_url) {
      this.$store.dispatch('showMediaDialog', image_url);
    },
    showUserDialog: function () {
      this.$store.dispatch('showUserDialog', this.status.getUser());
    },
    showRetweetUserDialog: function () {
      this.$store.dispatch('showUserDialog', this.status.getRetweetUser());
    }
  }
}
</script>

<style lang="scss">
.user-content {
  text-align: center;
  padding-right: 10px;
}

.retweet-user {
  span, img {
    font-size: 20px;
    vertical-align: middle;
  }
}

.user-thumb {
  cursor: pointer;
}

.media-list {
  padding: 0px;

  img {
    cursor: pointer;
  }
}

.action-area {
  font-size: 20px;

  .reaction {
    cursor: pointer;
  }
}
</style>
