<template>
    <div class="p-6 max-w-2xl mx-auto">
        <h1 class="text-2xl font-bold mb-4">Créer un devis</h1>
    
        <!-- Formulaire -->
        <form @submit.prevent="submitQuote" class="space-y-4 border p-4 rounded">
            <div>
            <label class="block mb-1 font-medium">Client</label>
            <select v-model="form.client_id" class="input" required>
                <option value="">-- Choisir un client --</option>
                <option v-for="client in clients" :key="client.id" :value="client.id">
                {{ client.name }} ({{ client.email }})
                </option>
            </select>
            </div>
            <div>
            <label class="block mb-1 font-medium">Description</label>
            <input v-model="form.description" type="text" class="input" required />
            </div>
            <div>
            <label class="block mb-1 font-medium">Montant (€)</label>
            <input v-model="form.amount" type="number" class="input" required />
            </div>
    
            <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">Créer le devis</button>
        </form>
    
        <h2 class="text-xl font-semibold mt-8 mb-4">Devis existants</h2>
        <ul class="space-y-2">
            <li v-for="quote in quotes" :key="quote.id" class="border p-3 rounded shadow-sm">
            <p>
                <strong>{{ quote.description }}</strong> – {{ quote.amount }} €  
                <br />
                <span class="text-sm text-gray-500">Client ID : {{ quote.client_id }}</span>
            </p>
            </li>
        </ul>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Quote {
id: number
client_id: number
description: string
amount: number
}

interface Client {
id: number
name: string
email: string
}

const quotes = ref<Quote[]>([])
const clients = ref<Client[]>([])

const form = ref({
client_id: '',
description: '',
amount: ''
})

onMounted(async () => {
    try {
        clients.value = await $fetch<Client[]>('http://localhost:3333/clients')
        quotes.value = await $fetch<Quote[]>('http://localhost:3333/quotes')
    } catch (err) {
        console.error("Erreur lors du chargement des données", err)
    }
})

async function submitQuote() {
    try {
        const newQuote = await $fetch<Quote>('http://localhost:3333/quotes', {
        method: 'POST',
        body: form.value
        })
        quotes.value.push(newQuote)
        form.value = { client_id: '', description: '', amount: '' }
    } catch (err) {
    console.error("Erreur lors de l'ajout du devis", err)
    }
}
</script>

<style scoped>
    .input {
    @apply border border-gray-300 px-3 py-2 rounded w-full;
    }
</style>