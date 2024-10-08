<template>
  <div class="body">
    <div class="container">
      <v-card class="mx-auto pa-12 profil" rounded="lg">
        <v-btn
          class="ma-5 back-btn"
          variant="text"
          @click="retourPagePrecedente"
          icon="mdi-arrow-left"
        ></v-btn>
        <div class="head">
          <h1 class="title text-capitalize">
            {{ currentUser.firstName }} {{ currentUser.name }}
            {{ currentUser.email }}
          </h1>
        </div>
        <div class="text-center">
          <div class="mb-5">
            <h2>Vos principaux centres d'intérêts</h2>
            <p>(3 max)</p>

            <v-row>
              <v-col cols="12">
                <v-chip
                  closable
                  class="ma-1"
                  v-for="userHobby in userHobbies"
                  :key="userHobby.id"
                  @click:close="removeHobbyFromUser(userHobby.id)"
                >
                  {{ userHobby.label }}
                </v-chip>
              </v-col>
            </v-row>
            <v-btn
              prepend-icon="mdi-plus"
              class="mt-5 add-hobby-btn"
              color="blue"
              @click="toggleHobbyModal"
            >
              Ajouter des hobbies
            </v-btn>
          </div>
        </div>
      </v-card>
      <v-card
        :class="['categories', { 'categories-expanded': isCategoriesExpanded }]"
        class="mx-auto"
      >
        <div
          style="display: flex; flex-direction: column; height: 100%"
          class="pa-3 mx-auto"
        >
          <div class="scrollable-categories">
            <div
              class="category ma-2 pa-1 rounded-lg"
              v-for="category in categories"
              :key="category.id"
              @click="toggleHobbies(category.id)"
            >
              <div style="display: flex">
                <v-icon class="mr-2" :icon="'mdi-' + category.icon"></v-icon>
                <h3>
                  {{ category.label }}
                </h3>
              </div>
              <div :class="['hobbies', { show: category.showHobbies }]">
                <v-chip
                  class="mr-2 mb-2"
                  v-for="hobby in category.hobbies"
                  :key="hobby.id"
                  @click="addHobbyToUser(hobby.id)"
                >
                  {{ hobby.label }}
                </v-chip>
              </div>
            </div>
          </div>
        </div>
      </v-card>
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import apiService from "@/services/apiService";

export default {
  data() {
    return {
      showHobbyModal: false, // Contrôle l'affichage du modal d'ajout de hobbies
      newHobby: "", // Stocke la valeur du nouveau hobby à ajouter
      valid: true, // Indique si le formulaire est valide
      categories: [], // Liste des catégories récupérées depuis l'API
      hobbies: null, // Liste des hobbies récupérés depuis l'API
      isCategoriesExpanded: false, // Contrôle si les catégories sont étendues ou non
      selectedHobby: null, // Stocke l'hobby sélectionné (peut-être utilisé dans le modal)
      availableHobbies: [], // Liste des hobbies disponibles pour ajout
    };
  },
  async mounted() {
    try {
      // Exemple d'appel API pour récupérer des données initiales
      const categories = await apiService.get("/categories");
      console.log("Données initiales récupérées:", categories);
      this.categories = categories;

      const hobbies = await apiService.get("/hobbies");
      console.log("Données initiales récupérées:", hobbies);
      this.hobbies = hobbies;
      this.availableHobbies = hobbies; // Remplir la liste des hobbies disponibles
      console.log(this.availableHobbies);
    } catch (error) {
      console.error(
        "Erreur lors de la récupération des données initiales:",
        error
      );
    }
  },
  methods: {
    // Méthode pour basculer l'affichage des hobbies d'une catégorie
    toggleHobbies(categoryId) {
      this.categories = this.categories.map((category) => {
        if (category.id === categoryId) {
          category.showHobbies = !category.showHobbies; // Change l'état d'affichage des hobbies
        }
        return category;
      });
    },
    
    // Méthode pour revenir à la page précédente dans l'historique du navigateur
    retourPagePrecedente() {
      this.$router.go(-1);
    },
    
    // Méthode pour ouvrir ou fermer le modal d'ajout de hobbies
    toggleHobbyModal() {
      this.showHobbyModal = true; // Ouvre le modal
      this.isCategoriesExpanded = !this.isCategoriesExpanded; // Bascule l'état des catégories
    },
    
    // Méthode asynchrone pour ajouter un hobby à l'utilisateur
    async addHobbyToUser(hobbyId) {
      if (
        !hobbyId || // Vérifie si l'ID du hobby est valide
        this.userHobbies.length >= 3 || // Vérifie si le nombre de hobbies est déjà au maximum
        this.userHobbies.some((hobby) => hobby.id === hobbyId) // Vérifie si le hobby est déjà ajouté
      ) {
        return; // Ne fait rien si une des conditions est remplie
      }
      const userId = this.currentUser.id; // Récupère l'ID de l'utilisateur actuel
      apiService
        .post(`/users/hobbies/${userId}/${hobbyId}`) // Envoie une requête POST pour ajouter le hobby
        .then(async (res) => {
          // Recharger la liste des hobbies de l'utilisateur après l'ajout
          if (res?.success) {
            const userHobbies = await apiService.get(
              `/users/${userId}/hobbies`
            );
            this.$store.commit("setUserHobbies", userHobbies);
            console.log("Liste des hobbies rechargée:", userHobbies);
          }
        });
    },
    
    // Méthode asynchrone pour supprimer un hobby de l'utilisateur
    async removeHobbyFromUser(hobbyId) {
      try {
        const userId = this.currentUser.id; // Récupère l'ID de l'utilisateur actuel
        await apiService.delete(`/users/hobbies/${userId}/${hobbyId}`); // Envoie une requête DELETE pour supprimer le hobby
        console.log("Hobby supprimé:", hobbyId);

        // Recharger la liste des hobbies de l'utilisateur après la suppression
        const userHobbies = await apiService.get(`/users/${userId}/hobbies`);
        this.$store.commit("setUserHobbies", userHobbies);
        console.log("Liste des hobbies rechargée:", userHobbies);
      } catch (error) {
        console.error("Erreur lors de la suppression du hobby:", error);
      }
    },
    
    // Règle de validation pour le nombre maximum de hobbies
    maxHobbiesRule(v) {
      if (!v) {
        return true;
      }
      return v.length <= 3 || "Vous ne pouvez pas ajouter plus de 3 hobbies.";
    },
    
    // Règle de validation pour les hobbies uniques
    uniqueHobbyRule(v) {
      if (!v) {
        return true;
      }
      const selectedHobbyIds = v.map((hobby) => hobby.id);
      const userHobbyIds = this.userHobbies.map((hobby) => hobby.id);
      const hasDuplicate = selectedHobbyIds.some((id) =>
        userHobbyIds.includes(id)
      );
      return !hasDuplicate || "Ce hobby est déjà ajouté.";
    },
  },
  computed: {
    // Mappe les getters du store Vuex pour obtenir les données nécessaires
    ...mapGetters({
      currentUser: "getUser", // Récupère l'utilisateur actuel
      userHobbies: "getUserHobbies", // Récupère les hobbies de l'utilisateur
    }),
  },
};
</script>

<style scoped>
html {
  overflow-y: hidden;
}

.back-btn {
  top: 0;
  position: absolute;
}

.body {
  background: linear-gradient(115deg, #4faaf5 0%, #1f30c9 100%) !important;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}

.container {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
  align-content: center;
  align-items: center;
  height: 50vh;
  /* flex-direction: column; */
  border-radius: 8px !important;
  overflow: hidden;
  background-color: #f9f9f9;
}

.category > div {
  padding: 0 0.5rem;
  gap: 0.25rem;
  align-items: center;
}

.flex {
  display: flex;
  justify-content: center;
}

.profil {
  height: 100%;
  z-index: 10;
  padding: 0 !important;
  transition: max-width 0.5s ease;
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 35vw;
  box-shadow: 0px 5px 5px -3px var(--v-shadow-key-umbra-opacity, rgba(0, 0, 0, 0.2)),
    0px 8px 10px 1px var(--v-shadow-key-penumbra-opacity, rgba(0, 0, 0, 0.14)),
    0px 3px 14px 2px var(--v-shadow-key-ambient-opacity, rgba(0, 0, 0, 0.12)) !important;
}

.categories {
  height: 100%;
  width: 0vw;
  box-shadow: none !important;
  z-index: 1;
  transition: width 0.5s ease;
}

.categories-expanded {
  background-color: #f9f9f9;
  width: 35vw !important;
}

.scrollable-categories {
  overflow-y: auto;
  overflow-x: hidden;
}

.category {
  cursor: pointer;
  color: white;
  background: linear-gradient(115deg, #4faaf5 0%, #1f30c9 100%) !important;
  align-content: stretch;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  flex-direction: column;
}

.category:hover {
  transition: 0.3s ease;
  opacity: 0.8;
}

.hobbies {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.5s ease, padding 0.5s ease;
}

.hobbies.show {
  border-top: 0.5px white solid;
  max-height: 500px; /* Vous pouvez ajuster cette valeur en fonction de la hauteur maximale que vos hobbies peuvent atteindre */
  padding: 10px;
}

.head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.head .title {
  flex: 1;
  text-align: center;
}
</style>
