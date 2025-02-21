<template>
  <div class="card">
    <h2>Enregistrer une participation</h2>

    <div class="form-group">
      <label for="personne">Personne</label>
      <select id="personne" v-model="data.formulaire.personne">
        <option v-for="p in data.personnes" :key="p.matricule" :value="p">
          {{ p.nom }} ({{ p.matricule }})
        </option>
      </select>
    </div>

    <div class="form-group">
      <label for="projet">Projet</label>
      <select id="projet" v-model="data.formulaire.projet">
        <option v-for="p in data.projets" :key="p.id" :value="p">
          {{ p.nom }}
        </option>
      </select>
    </div>

    <div class="form-group">
      <label for="role">Rôle</label>
      <input id="role" v-model="data.formulaire.role" type="text" placeholder="Développeur">
    </div>

    <div class="form-group">
      <label>Pourcentage</label>
      <input
        type="range"
        v-model="pourcentageAffiche"
        min="0"
        max="100"
        step="5"
        @input="mettreAJourPourcentage"
      >
      <p>{{ pourcentageAffiche }}%</p>
    </div>

    <button class="btn" @click="enregistrerParticipation">Enregistrer</button>
  </div>
</template>

<script setup>
import {onMounted, reactive, ref} from "vue";
import doAjaxRequest from "@/util/util.js";

// Données réactives
const participationVide = {
  personne: "",
  projet: "",
  role: "",
  pourcentage: 0.0,
};

const pourcentageAffiche = ref(0);

const mettreAJourPourcentage = () => {
  data.formulaire.pourcentage = pourcentageAffiche.value / 100;
};

let data = reactive({
  formulaire: { ...participationVide },
  personnes: [],
  projets: [],
});

async function enregistrerParticipation() {
  if (!data.formulaire.personne || !data.formulaire.projet) {
    alert("Veuillez sélectionner une personne et un projet !");
    return;
  }

  const personneHref = data.formulaire.personne._links.self.href;
  const projetHref = data.formulaire.projet._links.self.href;

  // Vérification avant d'envoyer la requête
  if (await participationExiste(personneHref, projetHref)) {
    alert("Cette personne participe déjà à ce projet !");
    return;
  }

  const options = {
    method: "POST",
    body: JSON.stringify({
      personne: personneHref,
      projet: projetHref,
      role: data.formulaire.role,
      pourcentage: data.formulaire.pourcentage / 100 // ✅ Format correct
    }),
    headers: {
      "Content-Type": "application/json",
    },
  };

  try {
    const response = await fetch("/api/participations", options);
    if (!response.ok) throw new Error("Échec de l'enregistrement");

    alert("Participation enregistrée avec succès !");
    data.formulaire = { personne: null, projet: null, role: "", pourcentage: 0 };
  } catch (error) {
    alert("Erreur lors de l'enregistrement : " + error.message);
  }
}

// Fonction pour récupérer les personnes et projets
function refresh() {
  doAjaxRequest("/api/personnes")
    .then((result) => {
      data.personnes = result._embedded.personnes;
    })
    .catch((error) => alert(error.message));

  doAjaxRequest("/api/projets")
    .then((result) => {
      data.projets = result._embedded.projets;
    })
    .catch((error) => alert(error.message));
}

// Charger les données au montage du composant
onMounted(refresh);

async function participationExiste(personneHref, projetHref) {
  try {
    const response = await fetch("/api/participations");
    const data = await response.json();

    return data._embedded.participations.some(p =>
      p._links.personne.href === personneHref &&
      p._links.projet.href === projetHref
    );
  } catch (error) {
    console.error("Erreur lors de la vérification :", error);
    return false;
  }
}

</script>

<style scoped>
.card {
  max-width: 400px;
  margin: 2rem auto;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  text-align: left;
}

h2 {
  text-align: center;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  font-weight: bold;
  display: block;
  margin-bottom: 5px;
}

select, input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 6px;
}

input[type="range"] {
  width: 100%;
  margin-top: 5px;
}

p {
  text-align: center;
  font-size: 16px;
  font-weight: bold;
}

.btn {
  width: 100%;
  background: #007BFF;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 16px;
}

.btn:hover {
  background: #0056b3;
}
</style>
