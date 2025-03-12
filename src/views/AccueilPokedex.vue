<template>
  <div>
    <h1>Mon Pokédex</h1>

    <!-- Barre de recherche -->
    <div class="search-bar">
      <label for="site-search">Rechercher par nom :</label>
      <input type="search" v-model="rechercheNom" placeholder="ex : Pikachu" />
    </div>

    <!-- Chargement en cours -->
    <div v-if="loading">Chargement en cours...</div>

    <!-- Liste des Pokémon -->
    <div v-else class="pokemon-list">
      <CartePokemon
        v-for="pokemon in pokemonsFiltres"
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
      pokemons: [], // Liste des Pokémon
      loading: true, // État de chargement
      rechercheNom: "", // Valeur de la recherche
    };
  },
  computed: {
    pokemonsFiltres() {
      if (!this.rechercheNom) {
        return this.pokemons; // Si la recherche est vide, retourne tous les Pokémon
      }
      // Filtre les Pokémon dont le nom correspond à la recherche (insensible à la casse)
      return this.pokemons.filter((pokemon) =>
        pokemon.name.toLowerCase().includes(this.rechercheNom.toLowerCase())
      );
    },
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
.search-bar {
  margin-bottom: 20px;
}

input[type="search"] {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  margin-left: 10px;
}

.pokemon-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}
</style>
