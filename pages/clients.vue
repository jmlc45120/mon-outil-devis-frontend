<template>
    <div class="p-6">
        <h1 class="text-2xl font-bold mb-4">Liste des clients</h1>

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
// Déclaration du type attendu depuis l'API
interface Client {
    id: number
    name: string
    email: string
    company_name: string | null
    phone: string | null
}

// Utilisation de useFetch avec typage
const { data: clients, pending, error } = await useFetch<Client[]>('http://localhost:3333/clients')
</script>