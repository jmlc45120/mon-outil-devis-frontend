<template>
    <div class="p-6 max-w-2xl mx-auto">
        <h1 class="text-2xl font-bold mb-4">Créer un devis</h1>
        <p v-if="formError" class="text-red-600 font-semibold mb-2">
            {{ formError }}
        </p>
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
        <div class="mb-4">
            <label class="font-medium mr-2">Trier par :</label>
            <select v-model="sortBy" class="border px-2 py-1 rounded">
                <option value="title">Titre du devis (A → Z)</option>
                <option value="client">Nom du client (A → Z)</option>
                <option value="date-desc">Date (récent → ancien)</option>
                <option value="date-asc">Date (ancien → récent)</option>
            </select>
        </div>
        <input
            v-model="search"
            type="text"
            placeholder="Rechercher un devis ou un client..."
            class="input mb-4"
        />
        <h2 class="text-xl font-semibold mt-8 mb-4">Devis existants</h2>
        <ul class="space-y-2">
            <li
                v-for="quote in filteredQuotes"
                :key="quote.id"
                class="border p-3 rounded shadow-sm flex justify-between items-center"
            >
                <div>
                    <p><strong>{{ quote.title }}</strong> – {{ quote.total }} €</p>
                    <p class="text-sm text-gray-500">
                        Client : {{ quote.client?.name || 'Client inconnu' }}<br />
                        Créé le : {{ formatDate(quote.createdAt) }}
                    </p>
                </div>
                <button @click="deleteQuote(quote.id)" class="text-red-600 hover:underline text-sm">🗑️ Supprimer</button>
            </li>
        </ul>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
const sortBy = ref<'title' | 'client' | 'date-desc' | 'date-asc'>('title')
const formError = ref('')
const search = ref('')
const filteredQuotes = computed(() => {
    const term = search.value.toLowerCase()
    return quotes.value.filter((quote) => {
        const title = quote.title.toLowerCase()
        const clientName = quote.client?.name?.toLowerCase() || ''
        return title.includes(term) || clientName.includes(term)
    })
})

interface Client {
    id: number
    name: string
    email: string
}
function formatDate(dateStr: string) {
    return new Date(dateStr).toLocaleDateString('fr-FR', {
        day: '2-digit',
        month: 'short',
        year: 'numeric',
    })
}

interface Quote {
id: number
client_id: number
title: string
total: number
client?: Client
createdAt: string
}

const quotes = ref<Quote[]>([])
const clients = ref<Client[]>([])

const form = ref({
clientId: '',
title: '',
total: ''
})

onMounted(loadQuotes)

async function submitQuote() {
  // 🔴 Réinitialise les messages d'erreur
    formError.value = ''

    // ✅ Vérification des champs
    if (!form.value.title.trim() || !form.value.clientId || !form.value.total) {
        formError.value = 'Merci de remplir tous les champs du formulaire.'
        return
    }

    try {
        const payload = {
        clientId: Number(form.value.clientId),
        title: form.value.title,
        total: Number(form.value.total)
        }

        const newQuote = await $fetch<Quote>('http://localhost:3333/quotes', {
        method: 'POST',
        body: payload
        })

        quotes.value.push(newQuote)
        form.value = { clientId: '', title: '', total: '' }
    } catch (err) {
        formError.value = "Une erreur est survenue lors de l'ajout du devis."
        console.error("Erreur lors de l'ajout du devis", err)
    }
}
async function deleteQuote(id: number) {
    if (!confirm("Es-tu sûr de vouloir supprimer ce devis ?")) return

    try {
        await $fetch(`http://localhost:3333/quotes/${id}`, { method: 'DELETE' })
        quotes.value = quotes.value.filter((q) => q.id !== id)
    } catch (err) {
        console.error("Erreur lors de la suppression", err)
    }
}
async function loadQuotes() {
    try {
        clients.value = await $fetch<Client[]>('http://localhost:3333/clients')
        const fetchedQuotes = await $fetch<Quote[]>('http://localhost:3333/quotes')

        // 👉 On trie par titre (alphabétique), puis par date (plus récent en haut)
        quotes.value = fetchedQuotes.sort((a, b) => {
            if (a.title < b.title) return -1
            if (a.title > b.title) return 1
            return new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
        })
    } catch (err) {
        console.error("Erreur lors du chargement des données", err)
    }
}
const sortedQuotes = computed(() => {
    return [...quotes.value].sort((a, b) => {
        if (sortBy.value === 'title') {
            return a.title.localeCompare(b.title)
        } else if (sortBy.value === 'client') {
        const nameA = a.client?.name || ''
        const nameB = b.client?.name || ''
        return nameA.localeCompare(nameB)
        } else if (sortBy.value === 'date-desc') {
        return new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
        } else if (sortBy.value === 'date-asc') {
        return new Date(a.createdAt).getTime() - new Date(b.createdAt).getTime()
        }
        return 0
    })
})
</script>

<style scoped>
.input {
@apply border border-gray-300 px-3 py-2 rounded w-full;
}
</style>