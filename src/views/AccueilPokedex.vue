<template>
  <div>
    <h1>Mon Pokédex</h1>

    <!-- Barre de recherche -->
    <div class="search-bar">
      <label for="site-search">Rechercher par nom :</label>
      <input type="search" v-model="rechercheNom" placeholder="ex : Pikachu" />
    </div>

    <!-- Option de tri -->
    <div class="tri-options">
      <select v-model="triType">
        <option value="">Tous les types</option>
        <option v-for="type in typesUniques" :value="type" :key="type">
          {{ type }}
        </option>
      </select>

      <select v-model="triOrdre">
        <option value="asc">A-Z</option>
        <option value="desc">Z-A</option>
      </select>
    </div>

    <!-- Chargement en cours -->
    <div v-if="loading">Chargement en cours...</div>

    <!-- Liste des Pokémon -->
    <div v-else class="pokemon-list">
      <CartePokemon
        v-for="pokemon in pokemonsFiltres"
        :key="pokemon.name"
        :pokemon="pokemon"
        @pokemon-click="ouvrirFichePokemon"
      />
    </div>

    <!-- Bouton "Charger plus" -->
    <button @click="chargerPlus" class="load-more-button">
      Charger 5 Pokémon aléatoires
    </button>

    <!-- Modal pour la fiche Pokémon -->
    <PokemonFiche
      v-if="pokemonSelectionne"
      :pokemon="pokemonSelectionne"
      :visible="modalVisible"
      @close="fermerModal"
    />
  </div>
</template>

<script>
import CartePokemon from "@/components/CartePokemon.vue";
import PokemonFiche from "@/components/PokemonFiche.vue";

export default {
  components: {
    CartePokemon,
    PokemonFiche,
  },
  data() {
    return {
      pokemons: [],
      loading: true,
      rechercheNom: "",
      triType: "",
      triOrdre: "asc",
      typesUniques: [],
      //gerer la modal
      pokemonSelectionne: null,
      modalVisible: false,
    };
  },
  computed: {
    pokemonsFiltres() {
      let resultat = [...this.pokemons];

      if (this.rechercheNom) {
        resultat = resultat.filter((pokemon) =>
          pokemon.name.toLowerCase().includes(this.rechercheNom.toLowerCase())
        );
      }

      if (this.triType) {
        resultat = resultat.filter((pokemon) =>
          pokemon.types.some((type) => type.type.name === this.triType)
        );
      }

      if (this.triOrdre === "asc") {
        resultat.sort((a, b) => a.name.localeCompare(b.name));
      } else if (this.triOrdre === "desc") {
        resultat.sort((a, b) => b.name.localeCompare(a.name));
      }

      return resultat;
    },
  },
  async created() {
    try {
      const response = await fetch(
        "https://pokeapi.co/api/v2/pokemon?limit=50"
      );
      const data = await response.json();
      const pokemonUrls = data.results.map((pokemon) => pokemon.url);

      const pokemonDetails = await Promise.all(
        pokemonUrls.map((url) => fetch(url).then((res) => res.json()))
      );

      this.pokemons = pokemonDetails;
      this.typesUniques = [
        ...new Set(
          pokemonDetails.flatMap((p) => p.types.map((t) => t.type.name))
        ),
      ].sort();
    } catch (error) {
      console.error("Erreur lors de la récupération des Pokémon:", error);
    } finally {
      this.loading = false;
    }
  },
  methods: {
    // Méthode pour ouvrir la modal avec la fiche Pokémon
    ouvrirFichePokemon(pokemon) {
      this.pokemonSelectionne = pokemon;
      this.modalVisible = true;
    },
    // Méthode pour fermer la modal
    fermerModal() {
      this.modalVisible = false;
      this.pokemonSelectionne = null;
    },
    async chargerPlus() {
      try {
        const idsAleatoires = Array.from(
          { length: 5 },
          () => Math.floor(Math.random() * 1000) + 1
        );

        const nouveauxPokemons = await Promise.all(
          idsAleatoires.map((id) =>
            fetch(`https://pokeapi.co/api/v2/pokemon/${id}`).then((res) => {
              if (!res.ok)
                throw new Error(`Pokémon avec l'ID ${id} non trouvé`);
              return res.json();
            })
          )
        );

        const pokemonsValides = nouveauxPokemons.filter((p) => p !== null);
        this.pokemons = [...this.pokemons, ...pokemonsValides];
        this.typesUniques = [
          ...new Set(
            this.pokemons.flatMap((p) => p.types.map((t) => t.type.name))
          ),
        ].sort();
      } catch (error) {
        console.error(
          "Erreur lors du chargement des Pokémon supplémentaires:",
          error
        );
      }
    },
  },
};
</script>

<style scoped>
.search-bar {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 20px;
}

.search-bar label {
  font-size: 1.2rem;
  margin-right: 10px;
  color: #2c3e50;
}

input[type="search"] {
  padding: 10px;
  border: 2px solid #2c3e50;
  border-radius: 25px;
  font-size: 1rem;
  width: 300px;
  outline: none;
  transition: border-color 0.3s ease;
}

input[type="search"]:focus {
  border-color: #e74c3c;
}

.tri-options {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-bottom: 20px;
}

.tri-options select {
  padding: 8px 15px;
  border-radius: 8px;
  border: 2px solid #2c3e50;
  background-color: white;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
}

.tri-options select:hover {
  border-color: #e74c3c;
}

.pokemon-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 25px;
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.load-more-button {
  display: block;
  margin: 30px auto;
  padding: 12px 24px;
  background-color: #e74c3c;
  color: white;
  border: none;
  border-radius: 25px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
}

.load-more-button:hover {
  background-color: #c0392b;
  transform: scale(1.05);
}
</style>
