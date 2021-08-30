<template>
  <div id="app">
    <Post></Post>
  </div>
</template>

<script>
import Post from './components/Post.vue';

export default {
  name: 'App',

  components: {
    Post,
  },

  data: () => ({
    //
  }),
};
</script>
<style lang="scss" scoped>
#app { 
  font-family: Avenir, Helvetica, Arial, sans-serif; 
  -webkit-font-smoothing: antialiased; 
  -moz-osx-font-smoothing: grayscale; 
  text-align: center; 
  color: #3c3e50
}
</style>