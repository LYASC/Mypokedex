<template>
  <div class="pokemon-card-container" :style="{ background: getTypeColor() }">
    <div class="pokemon-card" @click="ouvrirFichePokemon">
      <div class="card-content">
        <img :src="pokemon.sprites.front_default" :alt="pokemon.name" />
        <h2>{{ pokemon.name }}</h2>
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

      <div class="abilities">
        <p>Abilities:</p>
        <ul>
          <li v-for="ability in pokemon.abilities" :key="ability.ability.name">
            {{ ability.ability.name }}
          </li>
        </ul>
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
    };
  },
  methods: {
    getTypeColor() {
      const types = this.pokemon.types.map(
        (type) => this.typeColors[type.type.name]
      );
      if (types.length === 1) return types[0];
      return `linear-gradient(45deg, ${types.join(", ")})`;
    },
    // Méthode pour émettre l'événement au clic sur la carte
    ouvrirFichePokemon() {
      this.$emit("pokemon-click", this.pokemon);
    },
  },
};
</script>

<style scoped>
.pokemon-card-container {
  border-radius: 15px;
  padding: 3px;
  background: linear-gradient(45deg, red, blue);
  display: inline-block;
  min-height: 100%; /* Suppression de la hauteur fixe */
}

.pokemon-card {
  background: white;
  border-radius: 12px;
  padding: 20px;
  text-align: center;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  display: flex;
  flex-direction: column;
  height: 100%;
}

.card-content {
  flex: 1; /* Prend tout l'espace disponible sauf celui des abilities */
}

.pokemon-card:hover {
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
}

.pokemon-card img {
  width: 120px;
  height: 120px;
  margin: 0 auto 15px;
  display: block;
}

.pokemon-card h2 {
  font-size: 1.5rem;
  color: #2c3e50;
  margin-bottom: 10px;
  text-transform: capitalize;
}

.types {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 15px;
  flex-wrap: wrap;
}

.type-badge {
  padding: 5px 10px;
  border-radius: 15px;
  color: white;
  font-size: 0.9rem;
  text-transform: capitalize;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.abilities {
  margin-top: auto; /* Pousse la section vers le bas */
  padding-top: 15px;
  border-top: 1px solid #eee;
}

.abilities p {
  font-weight: bold;
  margin-bottom: 8px;
  color: #2c3e50;
  text-align: left;
}

.abilities ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
  text-align: left;
}

.abilities li {
  background-color: #f8f8f8;
  padding: 6px 10px;
  border-radius: 10px;
  margin-bottom: 5px;
  font-size: 0.85rem;
  color: #333;
  text-transform: capitalize;
  display: inline-block;
  margin-right: 5px;
}
</style>
