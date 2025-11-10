<template>
  <div class="min-h-screen bg-gray-50 py-10 px-6">
    <div class="max-w-3xl mx-auto bg-white shadow-lg rounded-2xl p-8">
      <h1 class="text-2xl font-bold text-gray-800 mb-6 text-center">
        Gerenciamento de Usuários
      </h1>

      <div class="space-y-4 mb-8">
        <div>
          <label for="nome" class="block text-sm font-medium text-gray-700">Nome</label>
          <input
            id="nome"
            type="text"
            v-model="nome"
            placeholder="Digite o nome do usuário"
            class="w-full mt-1 rounded-lg border-gray-300 focus:border-blue-500 focus:ring focus:ring-blue-200"
          />
        </div>

        <div>
          <label for="senha" class="block text-sm font-medium text-gray-700">Senha</label>
          <input
            id="senha"
            type="password"
            v-model="senha"
            placeholder="Digite a senha"
            class="w-full mt-1 rounded-lg border-gray-300 focus:border-blue-500 focus:ring focus:ring-blue-200"
          />
        </div>

        <div class="flex justify-between mt-6">
          <button
            @click="inserirUsuario"
            class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition"
          >
            Inserir
          </button>
          <button
            @click="atualizar"
            class="bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition"
          >
            Atualizar
          </button>
        </div>

        <p v-if="erro" class="text-red-500 text-sm mt-2">{{ erro }}</p>
        <p v-else class="text-green-600 text-sm mt-2">Tudo ok!</p>
      </div>

      <div class="overflow-x-auto">
        <table class="min-w-full text-sm text-left border border-gray-200 rounded-lg overflow-hidden">
          <thead class="bg-gray-100 text-gray-700 uppercase text-xs">
            <tr>
              <th class="px-4 py-2 border-b">Id</th>
              <th class="px-4 py-2 border-b">Nome</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="usuario in usuarios"
              :key="usuario.id"
              class="hover:bg-blue-50 transition"
            >
              <td class="px-4 py-2 border-b">{{ usuario.id }}</td>
              <td class="px-4 py-2 border-b">{{ usuario.nome }}</td>
            </tr>
          </tbody>
        </table>
        <p v-if="usuarios.length === 0" class="text-gray-500 text-center mt-4">
          Nenhum usuário cadastrado
        </p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

interface Usuario {
  id?: number
  nome: string
  senha: string
}

const nome = ref<string>('')
const senha = ref<string>('')
const erro = ref<string>()
const usuarios = ref<Usuario[]>([])

async function inserirUsuario() {
  try {
    await axios.post('usuario', { nome: nome.value, senha: senha.value })
    nome.value = ''
    senha.value = ''
    atualizar()
    erro.value = undefined
  } catch (error) {
    erro.value = (error as Error).message
  }
}

async function atualizar() {
  try {
    usuarios.value = (await axios.get('usuario')).data
  } catch (e) {
    erro.value = 'Erro ao buscar usuários.'
  }
}

onMounted(() => atualizar())
</script>

<style scoped>
input {
  transition: all 0.2s ease;
}
</style>