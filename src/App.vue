<template>
  <div class="flex">
    <h4>Find your word, we'll be glad to help you!</h4>
  </div>
  <form @submit.prevent="getDefinitions">
    <input type="text" id="word" v-model.trim="word" required />
    <button class="btn waves-effect waves-light" type="submit">
      Get definition
    </button>
  </form>
  <div class="flex">
    <li v-for="text in textArray.slice(0, 1)" :key="text">
      <p class="pron">
        Pronunciation: <span class="test-meaning">[ {{ text }} ]</span>
      </p>
    </li>
  </div>
  <div class="flex">
    <li v-for="origin of originArray.slice(0, 1)" :key="origin">
      <audio controls>
        <source :src="origin" type="audio/mpeg" />
      </audio>
    </li>
  </div>
  <h5 v-if="word && word !== ''">
    Definition for: <span class="wrong">{{ word }}</span>
  </h5>
  <div class="spinner flex">
    <spin-loader v-if="isLoading"></spin-loader>
  </div>

  <div class="flex">
    <p class="error" v-if="!validQuery">
      Error: The word - <span class="wrong">{{ word }}</span> that you are
      searching for does not exist. Please try a different word.
    </p>
  </div>
  <div
    id="cardContainer"
    v-for="definition in definitionsArray"
    :key="definition.definition"
  >
    <div class="card">
      <div class="col s12 m5">
        <div class="card-panel teal">
          <p>{{ definition }}</p>
        </div>
      </div>
    </div>
  </div>

  <vue-copyright></vue-copyright>
</template>

<script>
import { ref } from "vue";
import axios from "axios";
import SpinLoader from "./components/SpinLoader.vue";
import VueCopyright from "./components/VueCopyright.vue";
export default {
  name: "App",
  components: { SpinLoader, VueCopyright },

  setup() {
    const definitionsArray = ref([]);
    const originArray = ref([]);
    const textArray = ref([]);
    const word = ref("");
    let validQuery = ref(true);
    let isLoading = ref(false);

    async function getDefinitions() {
      validQuery.value = true;
      isLoading.value = true;

      originArray.value.splice(0, originArray.value.length);
      textArray.value.splice(0, textArray.value.length);
      definitionsArray.value.splice(0, definitionsArray.value.length);
      try {
        const url = `https://api.dictionaryapi.dev/api/v2/entries/en/${word.value}`;

        const { data } = await axios.get(url);

        data.map(({ meanings }) => {
          meanings.map(({ definitions }) => {
            definitions.forEach(({ definition }) => {
              if (typeof definition === "string") {
                definitionsArray.value.push(definition);
              }
            });
          });
        });

        data.map(({ phonetics }) => {
          phonetics.forEach(({ text }) => {
            if (typeof text === "string") {
              textArray.value.push(text);
            }
          });
        });

        data.map(({ phonetics }) => {
          phonetics.forEach(({ audio }) => {
            if (typeof audio === "string") {
              originArray.value.push(audio);
            }
          });
        });

        isLoading.value = false;
      } catch (error) {
        validQuery.value = false;
        isLoading.value = false;
      }
    }

    return {
      word,
      getDefinitions,
      definitionsArray,
      validQuery,
      isLoading,
      originArray,
      textArray,
    };
  },
};
</script>

<style>
body {
  background: #3f3d3b;
  color: #fff;
  width: 80%;
  margin: 0 auto;
}

form {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 50px;
}

.flex {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 50px;
}

input#word {
  color: #fff;
  width: 50% !important;
  margin-right: 30px;
  padding-left: 20px;
}
#cardContainer {
  color: #fff;
  text-transform: capitalize;
}

.card-panel {
  border-radius: 10px;
}

.card {
  border-radius: 10px;
}

.error {
  color: red;
  font-size: 18px;
}

li {
  list-style: none;
}

.wrong {
  color: #fff;
  text-transform: uppercase;
  font-weight: bold;
  margin-right: 5px;
  margin-left: 5px;
}

.test-meaning {
  text-decoration: underline;
  font-size: 1.3rem;
  letter-spacing: 0.1rem;
}

.pron{
  font-size: 1.2rem;
  letter-spacing: 0.03rem;
  margin-bottom: 0;
}
</style>
