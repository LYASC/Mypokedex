<template>
  <AccueilPokedex />
</template>

<script>
import AccueilPokedex from "./views/AccueilPokedex.vue"; // Mise à jour de l'import

export default {
  components: {
    AccueilPokedex, // Mise à jour du nom du composant
  },
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  background-color: #f0f0f0;
}
</style>
