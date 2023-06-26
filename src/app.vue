<template>
    <div class="flex flex-col h-screen p-2">
      <VueLegacy/>
    </div>
  </template>
  
  <script>
  import VueLegacy from './vue-legacy';
  export default {
    components: {
      VueLegacy,
    },
  };
  </script>
  
  <style>
  @import url('https://cdn.jsdelivr.net/npm/tailwindcss@2.2.0/dist/tailwind.min.css');
  
  h1,
  p {
    font-family: Lato;
  }
  </style>
  