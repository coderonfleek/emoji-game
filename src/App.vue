<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link>
    </div>
    <router-view/>
  </div>
</template>

<style scoped>
#video1 {
  width: 500px;
  height: 400px;
  background-color: grey;
}

.currentEmoji {
  font-size: 120px;
}
</style>
