<template>
  <div class="pokemon-card">
    <img :src="pokemon.sprites.front_default" :alt="pokemon.name" />
    <h2>{{ pokemon.name }}</h2>
    <p>Type(s): {{ pokemon.types.map((type) => type.type.name).join(", ") }}</p>
    <p>
      Abilities:
      {{ pokemon.abilities.map((ability) => ability.ability.name).join(", ") }}
    </p>
  </div>
</template>

<script>
export default {
  props: {
    pokemon: {
      type: Object,
      required: true,
    },
  },
};
</script>

<style scoped>
.pokemon-card {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 16px;
  text-align: center;
  margin: 10px;
}
img {
  width: 100px;
  height: 100px;
}
</style>
