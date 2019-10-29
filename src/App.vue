<template>
  <div id="app">

    <router-view/>
  </div>
</template>

<style lang="scss">
 *{
   padding: 0;
   margin: 0;
 }
 html,body,#app{
   width: 100%;
   height: 100%;
 }
 .el-breadcrumb {
    border-bottom: 1px solid #ccc;
    height: 40px;
    line-height: 40px;
    margin-bottom: 20px;
  }
</style>
