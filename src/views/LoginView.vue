<template>
  <div class="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-50 via-white to-blue-100 p-4">
    <div class="bg-white shadow-lg rounded-2xl p-8 w-full max-w-sm border border-gray-100">
      <h1 class="text-3xl font-bold text-center text-gray-800 mb-6">Login</h1>

      <div class="space-y-4">
        <div>
          <label for="nome" class="block text-sm font-medium text-gray-600 mb-1">Nome</label>
          <input
            id="nome"
            type="text"
            v-model="nome"
            class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400"
          />
        </div>

        <div>
          <label for="senha" class="block text-sm font-medium text-gray-600 mb-1">Senha</label>
          <input
            id="senha"
            type="password"
            v-model="senha"
            class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400"
          />
        </div>

        <button
          @click="store.login(nome, senha)"
          class="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 transition-all"
        >
          Entrar
        </button>

        <p v-if="store.erro" class="text-red-500 text-center text-sm mt-2">{{ store.erro }}</p>
        <p v-if="store.token" class="text-green-600 text-center text-sm mt-2">{{ store.token }}</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { usuarioStore } from '@/stores/usuario'

const nome = ref('Aluno')
const senha = ref('123456')
const store = usuarioStore()
</script>