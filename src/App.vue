<template>
  <div id="app">
    <LoadingView v-if="$store.state.isLoading"></LoadingView>
    <TopMenu style="margin-bottom: 1em"></TopMenu>
<!--    <div id="nav">-->
<!--      <router-link to="/">Home</router-link> |-->
<!--      <router-link to="/about">About</router-link>-->
<!--    </div>-->
<!--    <RouteLinker></RouteLinker>-->
      <router-view></router-view>
  </div>
</template>
<script>
// import RouteLinker from '@/components/RouteLinker.vue';
import TopMenu from '@/components/TopMenu.vue';

export default {
  name: 'App',
  components: {
    TopMenu,
    // RouteLinker,
  },
};

</script>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

/*#nav {*/
/*  padding: 30px;*/
/*}*/

/*#nav a {*/
/*  font-weight: bold;*/
/*  color: #2c3e50;*/
/*}*/

/*#nav a.router-link-exact-active {*/
/*  color: #42b983;*/
/*}*/
</style>
