<template>
  <div class="fiche-overlay" v-if="visible" @click.self="fermerFiche">
    <div class="fiche-content" :style="ficheStyle">
      <button class="close-button" @click="fermerFiche">×</button>

      <div class="fiche-header">
        <h2>{{ pokemon.name }} #{{ pokemon.id }}</h2>
        <div class="types">
          <span
            v-for="type in pokemon.types"
            :key="type.slot"
            class="type-badge"
            :style="{ backgroundColor: typeColors[type.type.name] }"
          >
            {{ type.type.name }}
          </span>
        </div>
      </div>

      <div class="fiche-body">
        <div class="images">
          <img :src="pokemon.sprites.front_default" :alt="pokemon.name" />
          <img
            :src="pokemon.sprites.back_default"
            :alt="pokemon.name + ' back'"
          />
        </div>

        <div class="stats">
          <h3>Statistiques</h3>
          <div v-for="stat in pokemon.stats" :key="stat.stat.name" class="stat">
            <span class="stat-name">{{ stat.stat.name }}:</span>
            <div class="stat-bar">
              <div
                class="stat-value"
                :style="{ width: (stat.base_stat / 255) * 100 + '%' }"
              >
                {{ stat.base_stat }}
              </div>
            </div>
          </div>
        </div>

        <div class="details">
          <div class="physical">
            <h3>Détails Physiques</h3>
            <p>Taille: {{ pokemon.height / 10 }} m</p>
            <p>Poids: {{ pokemon.weight / 10 }} kg</p>
          </div>

          <div class="abilities">
            <h3>Capacités</h3>
            <ul>
              <li
                v-for="ability in pokemon.abilities"
                :key="ability.ability.name"
              >
                {{ ability.ability.name }}
              </li>
            </ul>
          </div>
        </div>

        <!-- Section pour afficher la chaîne d'évolution -->
        <div class="evolutions" v-if="evolutions.length">
          <h3>Évolutions</h3>
          <div class="evo-chain">
            <div
              v-for="(evo, index) in evolutions"
              :key="index"
              class="evo-item"
            >
              <img
                :src="getSpriteUrl(evo.url)"
                :alt="evo.name"
                class="evo-image"
              />
              <p class="evo-name">{{ evo.name }}</p>
              <span v-if="index < evolutions.length - 1" class="arrow">→</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    pokemon: {
      type: Object,
      required: true,
    },
    visible: {
      type: Boolean,
      required: true,
    },
  },
  data() {
    return {
      typeColors: {
        normal: "#A8A878",
        fire: "#F08030",
        water: "#6890F0",
        electric: "#F8D030",
        grass: "#78C850",
        ice: "#98D8D8",
        fighting: "#C03028",
        poison: "#A040A0",
        ground: "#E0C068",
        flying: "#A890F0",
        psychic: "#F85888",
        bug: "#A8B820",
        rock: "#B8A038",
        ghost: "#705898",
        dragon: "#7038F8",
        dark: "#705848",
        steel: "#B8B8D0",
        fairy: "#EE99AC",
      },
      // Propriété pour stocker la chaîne d'évolution
      evolutions: [],
    };
  },
  computed: {
    ficheStyle() {
      const types = this.pokemon.types.map((type) => type.type.name);
      if (types.length === 1) {
        return { borderColor: this.typeColors[types[0]] };
      }
      return {
        borderImage: `linear-gradient(45deg, ${types
          .map((t) => this.typeColors[t])
          .join(", ")}) 1`,
      };
    },
  },
  watch: {
    // Dès qu'un nouveau Pokémon est passé à la modal, on récupère sa chaîne d'évolution
    pokemon: {
      immediate: true,
      handler(newPokemon) {
        if (newPokemon && newPokemon.species && newPokemon.species.url) {
          this.fetchEvolutionChain();
        } else {
          this.evolutions = [];
        }
      },
    },
  },
  methods: {
    fermerFiche() {
      this.$emit("close");
    },
    async fetchEvolutionChain() {
      try {
        // Récupération des données d'espèces pour obtenir l'URL de la chaîne d'évolution
        const speciesResponse = await fetch(this.pokemon.species.url);
        const speciesData = await speciesResponse.json();
        if (speciesData.evolution_chain && speciesData.evolution_chain.url) {
          const evoResponse = await fetch(speciesData.evolution_chain.url);
          const evoData = await evoResponse.json();
          // On parse la chaîne d'évolution
          this.evolutions = this.parseEvolutionChain(evoData.chain);
        }
      } catch (error) {
        console.error(
          "Erreur lors de la récupération de la chaîne d'évolution:",
          error
        );
      }
    },
    // Fonction pour parser récursivement la chaîne d'évolution
    parseEvolutionChain(chain) {
      let evoArray = [];
      let currentChain = chain;
      while (currentChain) {
        evoArray.push({
          name: currentChain.species.name,
          url: currentChain.species.url,
        });
        if (currentChain.evolves_to && currentChain.evolves_to.length > 0) {
          // On ne prend que la première évolution pour simplifier
          currentChain = currentChain.evolves_to[0];
        } else {
          currentChain = null;
        }
      }
      return evoArray;
    },
    // Méthode pour extraire l'ID du Pokémon à partir de l'URL d'espèce
    getPokemonIdFromSpeciesUrl(url) {
      const parts = url.split("/").filter((part) => part);
      return parts[parts.length - 1];
    },
    // Méthode pour construire l'URL de l'image du Pokémon dans la chaîne d'évolution
    getSpriteUrl(speciesUrl) {
      const id = this.getPokemonIdFromSpeciesUrl(speciesUrl);
      return `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${id}.png`;
    },
  },
};
</script>

<style scoped>
.fiche-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.fiche-content {
  background: white;
  border-radius: 15px;
  padding: 25px;
  width: 90%;
  max-width: 700px;
  max-height: 90vh;
  overflow-y: auto;
  position: relative;
  border: 4px solid transparent;
}

.close-button {
  position: absolute;
  top: 15px;
  right: 15px;
  font-size: 28px;
  background: none;
  border: none;
  cursor: pointer;
  color: #333;
}

.fiche-header {
  text-align: center;
  margin-bottom: 20px;
}

.fiche-header h2 {
  text-transform: capitalize;
  color: #2c3e50;
}

.images {
  display: flex;
  justify-content: center;
  gap: 30px;
  margin-bottom: 25px;
}

.images img {
  width: 150px;
  height: 150px;
}

.stats {
  margin-bottom: 25px;
}

.stat {
  margin-bottom: 12px;
}

.stat-name {
  display: inline-block;
  width: 120px;
  text-transform: capitalize;
  color: #2c3e50;
}

.stat-bar {
  display: inline-block;
  width: calc(100% - 130px);
  background: #eee;
  border-radius: 5px;
  height: 20px;
  vertical-align: middle;
}

.stat-value {
  background: #4caf50;
  height: 100%;
  border-radius: 5px;
  color: white;
  text-align: right;
  padding-right: 5px;
  font-size: 12px;
  line-height: 20px;
}

.details {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

.details h3 {
  color: #2c3e50;
  margin-bottom: 10px;
}

.abilities ul {
  list-style-type: none;
  padding-left: 0;
}

.abilities li {
  background: #f5f5f5;
  padding: 8px 12px;
  margin-bottom: 8px;
  border-radius: 8px;
  text-transform: capitalize;
}

/* Styles pour la chaîne d'évolution */
.evolutions {
  margin-top: 25px;
}
.evo-chain {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  flex-wrap: wrap;
}
.evo-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.evo-image {
  width: 80px;
  height: 80px;
}
.evo-name {
  text-transform: capitalize;
  font-size: 0.9rem;
  margin-top: 5px;
  text-align: center;
}
.arrow {
  font-size: 1.5rem;
  color: #2c3e50;
}
@media (max-width: 768px) {
  .details {
    grid-template-columns: 1fr;
  }
  .images {
    flex-direction: column;
    align-items: center;
  }
}
</style>
