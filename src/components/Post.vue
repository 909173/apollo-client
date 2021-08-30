<template>
  <v-app>
    <v-main>
      <div id="block">
        <v-dialog v-model="dialog" max-width="500px">
          <v-card>
            <h2 v-if="isCreate">本棚に追加する</h2>
            <h2 v-if="!isCreate">本棚を更新する</h2>
            <v-form ref="form" v-model="valid">
              <v-text-field
                v-model="post.title"
                :rules="titleRules"
                :counter="20"
                label="タイトル"
                required
              ></v-text-field>
              <v-text-field
                v-model="post.author"
                :rules="authorRules"
                :counter="20"
                label="作者"
                required
              ></v-text-field>
              <v-btn v-if="isCreate" :disabled="!valid" @click="createPost">追加</v-btn>
              <v-btn v-if="!isCreate" :disabled="!valid" @click="updatePost">更新</v-btn>
              <v-btn @click="clear">クリア</v-btn>
            </v-form>
          </v-card>
        </v-dialog>
        <v-row style="width: 550px">
          <v-toolbar color="grey lighten-1">
            <v-toolbar-title>本棚</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn @click="showDialogNew" color="primary" dark class="mb-1">
              新規追加
            </v-btn>
          </v-toolbar>
            <div v-for="post in posts" :key="post.id">
              <v-card class="mx-auto" width="550px" outlined>
                <v-list-item three-line>
                  <v-list-item-content>
                    <v-list-item-title class="headline mb-1">
                      {{post.title}}/{{post.author}}
                    </v-list-item-title>
                    <v-list-item-subtitle>From Apollo-server</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-card-actions>
                  <v-btn color="success" small>
                    <v-icon @click="showDialogUpdate(post.id, post.title, post.author)" small>
                      編集する
                    </v-icon>
                  </v-btn>
                  <v-btn @click="deletePost(post.id, post.title)" small color="error">
                    <v-icon small>
                      削除する
                    </v-icon>
                  </v-btn>
                </v-card-actions>
              </v-card>
            </div>
        </v-row>
      </div>
    </v-main>
  </v-app>
</template>

<script>
import {ALL_POSTS} from "../graphql/query"
import { CREATE_POST, UPDATE_POST, DELETE_POST } from "../graphql/mutation"
export default {
  name: "Post",
  data:() => ({
    post: {
      id:"",
      title: "",
      author: ""
    },
    posts:[],
    valid: true,
    titleRules: [
      v => !!v || "タイトルは必須です",
      v => (v && v.length <= 20) || "タイトルは20文字以下で入力してください"
    ],
    authorRules: [
      v => !!v || "筆者は必須です",
      v => (v && v.length <= 20) || "筆者名は20文字以下で入力してください"
    ],
    progress: false,
    dialog: false,
    isCreate: true
  }),
  apollo: {
    posts: {
      query: ALL_POSTS
    }
  },
  methods: {
    createPost: function() {
      if (this.$refs.form.validate()) {
        this.progress = true
        this.$apollo.mutate({
          mutation:  CREATE_POST,
          variables: {
            title: this.post.title,
            author: this.post.author
          }
        }).then(() => {
          this.$apollo.queries.posts.fetchMore({
            updateQuery: (previousResult, {fetchMoreResult}) => {
              return {
                posts: fetchMoreResult.posts
              }
            }
          })
          this.dialog = false
          this.progress = false
          this.clear()
        }).catch((e) => console.log(e))
      }
    },
    updatePost: function() {
      if (this.$refs.form.validate()) {
        this.progress = true
        this.$apollo.mutate({
          mutation: UPDATE_POST,
          variables: {
            id: this.post.id,
            title: this.post.title,
            author: this.post.author
          }
        }).then(() => {
          this.$apollo.queries.posts.fetchMore( {
            updateQuery: (previeousResult, {fetchMoreResult}) => {
              return  {
                posts: fetchMoreResult.posts
              }
            }
          })
          this.dialog = false
          this.progress = false
          this.clear()
        }).then((e) => console.log(e))
      }
    },
    deletePost: function(id, title) {
      console.log(id, title)
      if (!confirm(title + "を削除してもよろしいですか？"))
      return 
      this.progress = true
      this.$apollo.mutate({
        mutation: DELETE_POST,
        variables: {
          id
        }
      }).then(() => {
        this.$apollo.queries.posts.fetchMore({
          updateQuery:(previousResult, {fetchMoreResult}) => {
            return {posts: fetchMoreResult.posts}
          }
        })
        this.progress = false
      }).catch(e => console.log(e))
    },
    clear: function() {
      this.$refs.form.reset()
    },
    showDialogNew: function() {
      this.isCreate = true
      this.dialog = true
    },
    showDialogUpdate: function(id, title, author) {
      this.isCreate = false
      this.dialog = true
      this.post = {
        id,
        title,
        author
      }
    }
  }
}
</script>
<style scoped>
#block {
  text-align: center; 
  display: inline-block; 
}
</style>