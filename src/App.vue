<script setup>
import HelloWorld from './components/HelloWorld.vue'
import MineSweeper from './components/MineSweeper.vue'
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />
    <div class="wrapper">
      <HelloWorld msg="MineSweeper" />
    </div>
  </header>

  <main>
    <MineSweeper />
  </main>
</template>

<style scoped>
header {
  display: flex;
  place-items: center;
  justify-content: center;
  padding-bottom: calc(var(--section-gap) / 4);
  line-height: 1.5;
}

header .logo {
  display: block;
  margin: 0 auto 2rem;
  margin: 0 2rem 0 0;
}

header .wrapper {
  display: flex;
  place-items: flex-start;
  flex-wrap: wrap;
}

@media (min-width: 1024px) {
  header {
    padding-bottom: 0;
    padding-right: calc(var(--section-gap) / 2);
  }
}
</style>
