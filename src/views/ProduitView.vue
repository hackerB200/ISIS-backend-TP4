<template>
    <main>
        <div>
            <h1>Les produits</h1>
            <h3>Page {{ actualPage + 1 }} / {{ totalPages }}</h3>
        </div>
        <div>
            <table>
                <caption>Liste des produits</caption>
                <tr>
                    <th>Nom</th>
                    <th>Prix</th>
                    <th>Stock</th>
                    <th>Commandes</th>
                </tr>
                <tr v-if="produits.length === 0">
                    <td colspan="4">Veuillez patienter, chargement des produits...</td>
                </tr>
                <tr v-for="produit in produits" :key="produit.reference">
                    <td>{{ produit.nom }}</td>
                    <td>{{ produit.prixUnitaire }}</td>
                    <td>{{ produit.unitesEnStock }}</td>
                    <td>{{ produit.unitesCommandees }}</td>
                </tr>
                <tr>
                    <td><button @click="goStart" :disabled="actualPage == 0">Debut</button></td>
                    <td><button @click="goBackward" :disabled="actualPage <= 0">Precedent</button></td>
                    <td><button @click="goForward" :disabled="actualPage >= totalPages - 1">Suivant</button>
                    </td>
                    <td><button @click="goEnd" :disabled="totalPages <= 1 || actualPage == totalPages - 1">Fin</button>
                    </td>
                </tr>
            </table>
        </div>
    </main>
</template>

<script setup>
import { reactive, onMounted, ref } from "vue";
import { doAjaxRequest } from "@/api";

let produits = ref([])
let actualPage = ref(0);
let totalPages = ref(0);

// BOUTONS

function goStart() {
    chargeProduits(0);
}

function goBackward() {
    if (actualPage.value <= 0) {
        return showError({ status: 400, message: "Page invalide" });
    }
    chargeProduits(actualPage.value - 1);
}

function goForward() {
    if (actualPage.value >= totalPages.value - 1) {
        return showError({ status: 400, message: "Page invalide" });
    }
    chargeProduits(actualPage.value + 1);
}

function goEnd() {
    if (totalPages.value <= 1) {
        showError({ status: 400, message: "Page invalide" });
    }
    chargeProduits(totalPages.value - 1);
}


// AFFICHAGE ERREURS
function showError(error) {
    console.log("Erreur : status %d", error.status)
    console.log(error.body);
    alert(error.message);
}

// CHARGEMENT DES PRODUITS
function chargeProduits(page = 0, size = 5) {
    if (page < 0 || size < 1) {
        return showError({ status: 400, message: "Page invalide" });
    }

    doAjaxRequest("/api/produits?sort=nom,asc&page=" + page + "&size=" + size)
        .then((json) => {
            produits.value = json._embedded.produits;
            actualPage.value = json.page.number
            totalPages.value = json.page.totalPages
        })
        .catch(showError);
}

onMounted(chargeProduits);

</script>


<style scoped>
td,
th {
    border: 1px solid #ddd;
    padding: 8px;
}

th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: left;
    background-color: #232623;
    color: rgb(255, 255, 255);
}
</style>
