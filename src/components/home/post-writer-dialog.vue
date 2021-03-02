<template>
  <v-dialog width="480" persistent v-model="isOpenLocal">
    <v-card>
      <v-card-title>
        <span>Write your Post</span>
        <v-spacer></v-spacer>
        <v-btn icon @click="isOpenLocal = false">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-card-title>
      <v-card-text>
        <v-row dense>
          <v-col cols="12">
            <v-textarea
              filled
              rounded
              placeholder="Write your post here *"
              :counter="300"
              :height="250"
              v-model="text"
            ></v-textarea>
          </v-col>
          <v-col cols="12">
            <v-select
              :items="flairs"
              filled
              rounded
              placeholder="Flair"
              v-model="flair"
              clearable
              @click:clear="flair = null"
            >
              <template v-slot:selection="{ item }">
                <v-chip small color="primary" class="text-uppercase">
                  #{{ item }}
                </v-chip>
              </template>
              <template v-slot:item="{ item }">
                <v-chip small color="primary" class="text-uppercase">
                  #{{ item }}
                </v-chip>
              </template>
            </v-select>
          </v-col>
        </v-row>
      </v-card-text>
      <v-card-actions>
        <span class="font-italic caption">Write at least 2 characters.</span>
        <v-spacer></v-spacer>
        <v-btn
          color="primary"
          :disabled="!isFormValid"
          @click="createPost"
          :loading="isCreatePostStart"
        >
          <span class="text-capitalize mr-2">Send Post</span>
          <v-icon>mdi-send</v-icon>
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import axios from "axios";

export default {
  name: "home-post-writer-dialog-component",

  props: {
    isOpen: {
      type: Boolean,
      required: true,
    },

    posts: {
      type: Array,
      required: true,
    },
  },

  data() {
    return {
      isOpenLocal: this.isOpen,
      text: null,
      flair: null,
      isCreatePostStart: false,
      postsLocal: this.posts,
    };
  },

  computed: {
    flairs() {
      return ["Opinion", "Concern", "Confession"];
    },

    isFormValid() {
      return this.text && this.text.length >= 2 && this.text.length <= 300;
    },
  },

  watch: {
    isOpen(value) {
      this.isOpenLocal = value;
    },

    isOpenLocal(value) {
      this.$emit("update:isOpen", value);
    },

    posts(value) {
      this.postsLocal = value;
    },

    postsLocal(value) {
      this.$emit("update:posts", value);
    },
  },

  methods: {
    async createPost() {
      this.isCreatePostStart = true;
      const formData = new FormData();
      formData.append("text", this.text.trim());
      if (this.flair) formData.append("flair", this.flair);
      try {
        const response = await axios.post(
          "https://udm-fw-development.herokuapp.com/api/post",
          formData
        );
        const { success, data } = response.data;
        if (success) {
          this.postsLocal = [data, ...this.postsLocal];
          this.text = null;
          this.flair = null;
          this.isOpenLocal = false;
        }
      } catch (_) {
        console.log(_);
      }
      this.isCreatePostStart = false;
    },
  },
};
</script>
