<template>
  <div>
    <h1>Mon Pokédex</h1>
    <div v-if="loading">Chargement en cours...</div>
    <div v-else class="pokemon-list">
      <CartePokemon
        v-for="pokemon in pokemons"
        :key="pokemon.name"
        :pokemon="pokemon"
      />
    </div>
  </div>
</template>

<script>
import CartePokemon from "@/components/CartePokemon.vue";

export default {
  components: {
    CartePokemon,
  },
  data() {
    return {
      pokemons: [],
      loading: true,
    };
  },
  async created() {
    try {
      // Récupérer la liste des Pokémon
      const response = await fetch(
        "https://pokeapi.co/api/v2/pokemon?limit=20"
      );
      const data = await response.json();
      const pokemonUrls = data.results.map((pokemon) => pokemon.url);

      // Récupérer les détails de chaque Pokémon
      const pokemonDetails = await Promise.all(
        pokemonUrls.map((url) => fetch(url).then((res) => res.json()))
      );

      // Mettre à jour la liste des Pokémon
      this.pokemons = pokemonDetails;
    } catch (error) {
      console.error("Erreur lors de la récupération des Pokémon:", error);
    } finally {
      // Désactiver le chargement
      this.loading = false;
    }
  },
};
</script>

<style scoped>
.pokemon-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}
</style>
