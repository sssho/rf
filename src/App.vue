<template>
  <FileLink />
</template>

<script>
import FileLink from "./components/FileLink.vue";

export default {
  name: "App",
  components: {
    FileLink
  }
};
</script>

<style>
/* * {
    background-color: white;;
} */

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-color: white;;
  margin-top: 40px;
  padding-top: 15px;
}
</style>
