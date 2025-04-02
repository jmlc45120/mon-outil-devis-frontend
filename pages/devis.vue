<template>
    <div class="p-6 max-w-2xl mx-auto">
        <h1 class="text-2xl font-bold mb-4">Créer un devis</h1>
    
        <!-- Formulaire -->
        <form @submit.prevent="submitQuote" class="space-y-4 border p-4 rounded">
            <div>
            <label class="block mb-1 font-medium">Client</label>
            <select v-model="form.clientId" class="input" required>
                <option value="">-- Choisir un client --</option>
                <option v-for="client in clients" :key="client.id" :value="client.id">
                {{ client.name }} ({{ client.email }})
                </option>
            </select>
            </div>
            <div>
            <label class="block mb-1 font-medium">Titre</label>
            <input v-model="form.title" type="text" class="input" required />
            </div>
            <div>
            <label class="block mb-1 font-medium">Montant (€)</label>
            <input v-model="form.total" type="number" class="input" required />
            </div>
    
            <button type="submit" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700">
            Créer le devis
            </button>
        </form>
    
        <h2 class="text-xl font-semibold mt-8 mb-4">Devis existants</h2>
        <ul class="space-y-2">
            <li v-for="quote in quotes" :key="quote.id" class="border p-3 rounded shadow-sm">
            <p>
                <strong>{{ quote.title }}</strong> – {{ quote.total }} €
                <br />
                <span class="text-sm text-gray-500">
                Client : {{ quote.client?.name || 'Client inconnu' }}
                </span>
            </p>
            </li>
        </ul>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Client {
    id: number
    name: string
    email: string
}

interface Quote {
id: number
client_id: number
title: string
total: number
client?: Client // <-- Important : pour quote.client.name
}

const quotes = ref<Quote[]>([])
const clients = ref<Client[]>([])

const form = ref({
clientId: '',
title: '',
total: ''
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
        const payload = {
            clientId: Number(form.value.clientId),
            title: form.value.title,
            total: Number(form.value.total)
        }

    console.log("Formulaire à envoyer", payload)

    const newQuote = await $fetch<Quote>('http://localhost:3333/quotes', {
    method: 'POST',
    body: payload
    })

    quotes.value.push(newQuote)
    form.value = { clientId: '', title: '', total: '' }
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