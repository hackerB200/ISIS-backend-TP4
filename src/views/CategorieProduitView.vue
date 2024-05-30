<template>
    <main>
        <div>
            <h1>Les produits par catégorie</h1>
            <select @change="chargeproduits" v-model="categorieSelected">
                <option value=""></option>
                <option v-for="categorie in categories" :key="categorie.code" :value="categorie.code">{{
                    categorie.libelle
                }}</option>
            </select>
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
            </table>
        </div>
    </main>
</template>

<script setup>
import { reactive, onMounted, ref } from "vue";
import { doAjaxRequest } from "@/api";

let produits = ref([]);
let categories = ref([]);
let categorieSelected = ref("");

function showError(error) {
    console.log("Erreur : status %d", error.status)
    console.log(error.body);
    alert(error.message);
}

function chargeproduits() {
    if (categorieSelected.value == "") {
        return;
    }
    doAjaxRequest("/api/categories/" + categorieSelected.value + "/produits")
        .then((json) => {
            produits.value = json._embedded.produits;
        })
        .catch(showError);
}

function chargeCategories() {
    doAjaxRequest("/api/categories?sort=code,desc")
        .then((json) => {
            categories.value = json._embedded.categories;
            console.log(categories.value);
        })
        .catch(showError);
}

onMounted(chargeCategories);

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
