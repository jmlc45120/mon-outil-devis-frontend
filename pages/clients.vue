<template>
    <div class="p-6 max-w-2xl mx-auto">
        <h1 class="text-2xl font-bold mb-4">Liste des clients</h1>
        <!-- Messages d'erreur ou succès -->
        <div v-if="errorMessage" class="text-red-600 font-semibold mb-2">
            {{ errorMessage }}
        </div>
        <div v-if="successMessage" class="text-green-600 font-semibold mb-2">
            {{ successMessage }}
        </div>
        <!-- Formulaire d'ajout -->
        <form @submit.prevent="submitClient" class="mb-6 space-y-4 border p-4 rounded">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <input v-model="form.name" type="text" placeholder="Nom complet" required class="input" />
                <input v-model="form.email" type="email" placeholder="Email" required class="input" />
                <input v-model="form.company_name" type="text" placeholder="Entreprise (optionnel)" class="input" />
                <input v-model="form.phone" type="text" placeholder="Téléphone (optionnel)" class="input" />
            </div>
            <button :disabled="loading" type="submit" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 disabled:opacity-50">
                {{ loading ? "Ajout en cours..." : "Ajouter un client" }}
            </button>
        </form>
    
        <!-- Liste des clients -->
        <ul class="space-y-2">
            <li
                v-for="client in clients"
                :key="client.id"
                class="border p-3 rounded shadow-sm"
                >
                <p><strong>{{ client.name }}</strong> ({{ client.email }})</p>
                <p class="text-sm text-gray-600">{{ client.company_name }} – {{ client.phone }}</p>
            </li>
        </ul>
    </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

interface Client {
    id: number
    name: string
    email: string
    company_name: string | null
    phone: string | null
}

const clients = ref<Client[]>([])

const form = ref({
    name: '',
    email: '',
    company_name: '',
    phone: '',
})
const successMessage = ref('')
const errorMessage = ref('')
const loading = ref(false)

// Récupération des clients à l'ouverture
onMounted(async () => {
    const { data } = await useFetch<Client[]>('http://localhost:3333/clients')
    if (data.value) clients.value = data.value
})

// Fonction pour ajouter un client
async function submitClient() {
    successMessage.value = ''
    errorMessage.value = ''

    // Validation frontend
    if (!form.value.name.trim()) {
        errorMessage.value = 'Le nom est obligatoire.'
        return
    }

    if (!form.value.email.includes('@')) {
        errorMessage.value = "L'adresse email est invalide."
        return
    }

    if (form.value.phone && !/^\d+$/.test(form.value.phone)) {
        errorMessage.value = 'Le téléphone ne doit contenir que des chiffres.'
        return
    }

    loading.value = true

    try {
        const { data, error } = await useFetch<Client>('http://localhost:3333/clients', {
        method: 'POST',
        body: form.value,
        })

        if (error.value) throw error.value

        if (data.value) {
        clients.value.push(data.value)
        successMessage.value = '🎉 Client ajouté avec succès !'
        form.value = { name: '', email: '', company_name: '', phone: '' }
        }
    } catch (err) {
        errorMessage.value = "Erreur lors de l'ajout du client."
        console.error(err)
    } finally {
        loading.value = false
    }
}
</script>

<style scoped>
.input {
    @apply border border-gray-300 px-3 py-2 rounded w-full;
}
</style>