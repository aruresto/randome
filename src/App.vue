<template>
  <div id="app">
    <el-container>
      <!-- <el-header>
        <Header/> 
      </el-header> -->
      <el-main>
        <input-component />
      </el-main>
      <!-- <el-footer>
        <Footer />
      </el-footer> -->
    </el-container>
  </div>
</template>

<script>
import InputComponent from "./components/InputComponent";
// import Header from './components/headers';
// import Footer from './components/footer'

export default {
  name: "app",
  components: {
    InputComponent,
    // Header,
    // Footer
  }
};
</script>

<style lang="scss" >
#app {
  font-family: 'Sarabun', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px; */
}

body {
  margin: 0;
}

button {
 font-family: 'Sarabun', sans-serif;
}

.el-row {
  margin-bottom: 20px;
  &:last-child {
    margin-bottom: 0;
  }
}
</style>
