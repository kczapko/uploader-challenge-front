<template lang="pug">
header
  h1 Image Uploader
main
  app-uploader
footer created by 
  b Krzysztof Czapko 
  | - devChallenges.io
</template>

<script>
import AppUploader from '@/components/Uploader.vue';

export default {
  name: 'App',
  components: { AppUploader },
};
</script>

<style lang="scss">
*,
*::before,
*::after {
  box-sizing: inherit;
  margin: 0;
}

html {
  box-sizing: border-box;
  font-size: 62.5%;
}

body {
  color: #000;
  font-family: 'Poppins', sans-serif;
  font-size: 1.4rem;
  background: #fafafb;
}

#app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  justify-content: space-between;
  align-items: center;
}

header h1 {
  margin: 2rem 0;
  font-size: 2rem;
  color: #333;
}

footer {
  margin: 2rem 0;
  color: #a9a9a9;
}
</style>
