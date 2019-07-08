<template>
  <div id="app">
    <el-container>
      <el-header>
        <!-- Header Component -->
      </el-header>
      <el-main>
        <input-component />
      </el-main>
      <el-footer>
        <!-- Footer Component -->
      </el-footer>
    </el-container>
  </div>
</template>

<script>
import InputComponent from "./components/InputComponent";

export default {
  name: "app",
  components: {
    InputComponent
  }
};
</script>

<style lang="scss" >
#app {
  font-family: "Itim", cursive;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

button {
  font-family: "Itim", cursive;
}

.el-row {
  margin-bottom: 20px;
  &:last-child {
    margin-bottom: 0;
  }
}
</style>
