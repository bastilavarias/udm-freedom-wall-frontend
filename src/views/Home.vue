<template>
  <v-app>
    <v-app-bar app flat color="primary" dark>
      <v-container>
        <v-row justify="space-between" align="center">
          <v-toolbar-title>
            <v-avatar :size="40" class="mr-2">
              <v-img :src="require('@/assets/udm_logo.png')"></v-img>
            </v-avatar>
            <span class="font-weight-bold"> UDM Freedom Wall </span>
          </v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn
            color="secondary"
            depressed
            light
            @click="isWritePostDialogOpen = true"
          >
            <span class="text-capitalize mr-2">Post</span>
            <v-icon>mdi-send</v-icon>
          </v-btn>
        </v-row>
      </v-container>
    </v-app-bar>
    <v-main>
      <v-container>
        <v-row
          v-masonry
          origin-left="true"
          horizontal-order="true"
          transition-duration="0.3s"
          item-selector=".item"
          gutter="0"
        >
          <v-col
            v-masonry-tile
            class="item"
            v-for="(post, index) in posts"
            :key="index"
            xs="3"
            sm="6"
            md="4"
          >
            <v-card outlined>
              <v-card-subtitle class="caption">
                Posted {{ formatDate(post.created_at) }}
              </v-card-subtitle>
              <v-card-text>
                <div :class="`${post.flair ? 'mb-5' : ''} body-1`">
                  {{ post.text }}
                </div>
                <v-chip
                  color="primary"
                  small
                  class="text-uppercase"
                  v-if="post.flair"
                >
                  {{ post.flair }}
                </v-chip>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
        <infinite-loading @infinite="getPosts" :identifier="scrollOptions.id">
          <template v-slot:spinner>
            <div class="py-10">
              <v-progress-circular
                color="secondary"
                indeterminate
                :size="50"
              ></v-progress-circular>
            </div>
          </template>
          <template v-slot:no-results><span></span></template>
          <template v-slot:no-more><span></span></template>
        </infinite-loading>
      </v-container>
    </v-main>
    <home-post-writer-dialog-component
      :is-open.sync="isWritePostDialogOpen"
      :posts.sync="posts"
    ></home-post-writer-dialog-component>
  </v-app>
</template>

<script>
import axios from "axios";
import HomePostWriterDialogComponent from "@/components/home/post-writer-dialog";
import moment from "moment";

export default {
  components: { HomePostWriterDialogComponent },
  data() {
    return {
      posts: [],
      isWritePostDialogOpen: false,
      scrollOptions: {
        page: 1,
        id: +new Date(),
      },
    };
  },

  methods: {
    async getPosts($state) {
      try {
        const response = await axios.get(
          `${process.env.VUE_APP_BACKEND_URL}/api/post?page=${this.scrollOptions.page}`
        );
        const result = response.data;
        const posts = result.data.data;
        if (posts.length === 10) {
          this.posts = [...this.posts, ...posts];
          this.scrollOptions.page++;
          $state.loaded();
        } else {
          this.posts = [...this.posts, ...posts];
          $state.complete();
        }
      } catch (_) {
        $state.complete();
        this.posts = [];
      }
    },

    formatDate(date) {
      return moment(date).format("MMMM DD YYYY");
    },
  },
};
</script>
