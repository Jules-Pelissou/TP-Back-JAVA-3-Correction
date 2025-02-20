<script setup>
import {onMounted, reactive} from "vue";
// Importer la fonction doAjaxRequest qui gère les erreurs d'API
import doAjaxRequest from "@/util/util.js"

// Pour réinitialiser le formulaire
const projetVide = {
  personne: "",
  projet : "",
  role : "",
  pourcentage : 10
};

// Les données du composant
let data = reactive({


  // Les données saisies dans le formulaire
  formulaire: {...projetVide},



  // Ce qui va être utile a display

  // Les projets récupérés depuis l'API
  projets: [],

  // Les personnes récupérées via l'API
  listePersonnes : [],

  // Les rôles via API
  listeRoles : [],
});

function ajouteProjet() {

  // Ajouter un projet avec les données du formulaire
  const options = { // Options de la requête fetch
    method: "POST", // Verbe HTTP POST pour ajouter un enregistrement
    // On transmet les données du formulaire dans le corps de la requête
    body: JSON.stringify(data.formulaire),
    headers: {
      "Content-Type": "application/json",
    },
  };
  // On appelle l'API REST générée par les repositories Spring Data REST
  doAjaxRequest("/api/projets", options)
    .then((result) => {
      console.log("Projet ajouté :", result);
      // Réinitialiser le formulaire
      data.formulaire = {...projetVide};
      refresh(); // Rafraîchir la liste des pays
    })
    .catch(error => alert(error.message));
}

function ajouterParticipation(){

  console.log(data.formulaire);

  // Options du POST

  const options = {
    method: "POST",
      body: JSON.stringify(data.formulaire),
      headers: {
      "Content-Type": "application/json",
    },
  };

  doAjaxRequest("/api/participations", options)
    .then((result) => {
      console.log("Participation ajoutée :", result);
      data.formulaire = {...projetVide};
    })
    .catch(error => alert(error.message));

}

function getPersonnes(){
  doAjaxRequest("/api/personnes") // Méthode GET par défaut
    .then((result) => {
      let resultat = result;
      console.log(resultat._embedded.personnes);
      data.listePersonnes = resultat._embedded.personnes;
    })
    .catch(error => alert(error.message));
}

function getProjets(){
  doAjaxRequest("/api/projets") // Méthode GET par défaut
    .then((result) => {
      data.projets = result._embedded.projets;
      console.log(data.projets);
    })
    .catch(error => alert(error.message));
}

function getRoles(){
  doAjaxRequest("/api/participations") // Méthode GET par défaut
    .then((result) => {

      result._embedded.participations.forEach(p => {
        if (!data.listeRoles.includes(p.role)) {
          data.listeRoles.push(p.role);
        }
      });

    })
    .catch(error => alert(error.message));
}

function getInfos() {
  getProjets();
  getPersonnes();
  getRoles();
}

// Appeler la fonction refresh() pour récupérer la liste des pays au chargement du composant
onMounted(getInfos);
</script>


<template>
  <div class="container">
    <h2>Enregistrer une participation</h2>
    <form @submit.prevent="ajouterParticipation">
      <div id="form">
        <label for="personne">Personne </label>
        <select id="personne" v-model="data.formulaire.personne">
          <option v-for="personne in data.listePersonnes" :key="personne.nom" :value="personne.matricule">{{ personne.nom }} {{personne.prenom}}</option>
        </select>

        <label for="projet">Projet </label>
        <select id="projet" v-model="data.formulaire.projet">
          <option v-for="projet in data.projets" :key="projet.nom" :value="projet.nom">{{ projet.nom }}</option>
        </select>

        <label for="role">Rôle </label>
        <select id="role" v-model="data.formulaire.role">
          <option v-for="role in data.listeRoles" :key="role">{{ role }}</option>
        </select>

        <label for="slider">Pourcentage</label>
        <input type="range" id="slider" min="0" max="100" v-model="data.formulaire.pourcentage">
        <p class="percentage">{{ data.formulaire.pourcentage }}%</p>

        <button type="submit">Ajouter</button>
      </div>
    </form>
  </div>
</template>

<!-- Un CSS pour ce composant -->
<style scoped>

#form {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

#form label {
  font-weight: bold;
  margin-bottom: 5px;
}

#form select,
#form input[type="range"] {
  padding: 8px;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 5px;
  width: 100%;
}

button {
  margin-top: 10px;
  background-color: #007BFF;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}


.container {
  margin: 2rem auto;
  max-width: 600px;
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 15px;
}

.label {
  display: block;
  font-weight: bold;
  margin-bottom: 5px;
}

.input-field {
  width: 100%;
  padding: 10px;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.input-field:focus {
  outline: none;
  border-color: #007BFF;
  box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
}

.button-container {
  text-align: center;
}

.submit-button {
  padding: 10px 20px;
  font-size: 1rem;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.submit-button:hover {
  background-color: #0056b3;
}

.table-container {
  margin-top: 20px;
}

table {
  width: 100%;
  border-collapse: collapse; /* Supprime les bordures en double */
  margin-top: 10px;
}

thead th {
  background-color: #007BFF;
  color: #fff;
  padding: 10px;
  text-align: left; /* Alignement à gauche */
  border: 1px solid #ddd;
}

tbody td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left; /* Alignement à gauche */
}

tbody tr:nth-child(odd) {
  background-color: #f2f2f2; /* Lignes alternées pour une meilleure lisibilité */
}

tbody tr:hover {
  background-color: #eaeaea;
}

th, td {
  font-size: 0.9rem;
  text-align: left;
}

code {
  color: #d63384;
  font-weight: bold;
  background-color: #f8f9fa;
  padding: 2px 4px;
  border-radius: 3px;
}

</style>
