<template>
  <div class="card">
    <h2>Enregistrer une participation</h2>

    <div class="form-group">
      <label for="personne">Personne</label>
      <select id="personne" v-model="form.personne">
        <option v-for="p in personnes" :key="p.id" :value="p.id">
          {{ p.nom }}
        </option>
      </select>
    </div>

    <div class="form-group">
      <label for="projet">Projet</label>
      <select id="projet" v-model="form.projet">
        <option v-for="p in projets" :key="p.id" :value="p.id">
          {{ p.nom }} ({{ p.etat }})
        </option>
      </select>
    </div>

    <div class="form-group">
      <label for="role">Rôle</label>
      <input id="role" v-model="form.role" type="text" placeholder="Développeur">
    </div>

    <div class="form-group">
      <label>Pourcentage</label>
      <input type="range" v-model="form.pourcentage" min="0" max="100" step="5">
      <p>{{ form.pourcentage }}%</p>
    </div>

    <button class="btn" @click="enregistrerParticipation">Enregistrer</button>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import doAjaxRequest from "@/util/util.js";

// Données réactives
const form = ref({
  personne: "",
  projet: "",
  role: "",
  pourcentage: 10,
});

const personnes = ref([]);
const projets = ref([]);

// Charger les personnes et projets
async function fetchData() {
  try {
    personnes.value = (await doAjaxRequest("/api/personnes"))._embedded.personnes;
    projets.value = (await doAjaxRequest("/api/projets"))._embedded.projets;
  } catch (error) {
    alert("Erreur lors du chargement des données");
  }
}

// Fonction pour enregistrer la participation
async function enregistrerParticipation() {
  const data = {
    matricule: form.value.personne,
    codeProjet: form.value.projet,
    role: form.value.role,
    pourcentage: form.value.pourcentage / 100,
  };

  try {
    await doAjaxRequest("/api/participations", { method: "POST", body: JSON.stringify(data), headers: { "Content-Type": "application/json" } });
    alert("Participation enregistrée !");
  } catch (error) {
    alert("Erreur: " + error.message);
  }
}

// Charger les données au montage
onMounted(fetchData);
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
