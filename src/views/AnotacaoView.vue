<template>
  <div class="min-h-screen bg-gray-50 p-6 flex flex-col items-center">
    <div class="w-full max-w-4xl bg-white rounded-2xl shadow-lg p-8 border border-gray-100">
      <h1 class="text-2xl font-bold text-gray-800 mb-6 text-center">Gerenciar Anotações</h1>

      <!-- Cadastro -->
      <form @submit.prevent="cadastrar" class="grid md:grid-cols-3 gap-4 mb-8">
        <div>
          <label for="texto" class="block text-sm font-medium text-gray-600 mb-1">Texto</label>
          <input
            type="text"
            id="texto"
            v-model="novaAnotacao.texto"
            required
            class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400"
          />
        </div>

        <div>
          <label for="dataHora" class="block text-sm font-medium text-gray-600 mb-1">Data/Hora</label>
          <input
            type="datetime-local"
            id="dataHora"
            v-model="novaAnotacao.dataHora"
            required
            class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400"
          />
        </div>

        <div>
          <label for="usuario" class="block text-sm font-medium text-gray-600 mb-1">Usuário</label>
          <select
            id="usuario"
            v-model="novaAnotacao.usuario.id"
            required
            class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400"
          >
            <option v-for="usuario in usuarios" :key="usuario.id" :value="usuario.id">
              {{ usuario.nome }}
            </option>
          </select>
        </div>

        <button
          type="submit"
          class="col-span-full bg-green-600 text-white py-2 rounded-lg hover:bg-green-700 transition-all mt-2"
        >
          Cadastrar
        </button>
      </form>

      <!-- Pesquisa -->
      <form @submit.prevent="pesquisar" class="grid md:grid-cols-2 gap-4 mb-6">
        <div>
          <label for="textoPesquisa" class="block text-sm font-medium text-gray-600 mb-1">Texto</label>
          <input
            type="text"
            id="textoPesquisa"
            v-model="textoPesquisa"
            required
            class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400"
          />
        </div>

        <div>
          <label for="nomeUsuarioPesquisa" class="block text-sm font-medium text-gray-600 mb-1">Usuário</label>
          <select
            id="nomeUsuarioPesquisa"
            v-model="nomeUsuarioPesquisa"
            required
            class="w-full border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-400"
          >
            <option v-for="usuario in usuarios" :key="usuario.id" :value="usuario.nome">
              {{ usuario.nome }}
            </option>
          </select>
        </div>

        <button
          type="submit"
          class="col-span-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 transition-all mt-2"
        >
          Pesquisar
        </button>
      </form>

      <!-- Tabela -->
      <div v-if="anotacoes.length > 0" class="overflow-x-auto">
        <table class="w-full border-collapse border border-gray-200 text-sm">
          <thead class="bg-blue-600 text-white">
            <tr>
              <th class="py-2 px-3 text-left">ID</th>
              <th class="py-2 px-3 text-left">Texto</th>
              <th class="py-2 px-3 text-left">Antiguidade (dias)</th>
              <th class="py-2 px-3 text-left">Usuário</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="anotacao in anotacoes"
              :key="anotacao.id"
              class="hover:bg-gray-50 border-t border-gray-200"
            >
              <td class="py-2 px-3">{{ anotacao.id }}</td>
              <td class="py-2 px-3">{{ anotacao.texto }}</td>
              <td class="py-2 px-3">{{ antiguidade(anotacao.dataHora) }}</td>
              <td class="py-2 px-3">{{ anotacao.usuario.nome }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <p v-else class="text-center text-gray-500 mt-6">
        Nenhum registro encontrado para os critérios fornecidos.
      </p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

interface Usuario {
  id: number
  nome?: string
  senha?: string
}

interface Anotacao {
  id?: number
  texto: string
  dataHora: string
  usuario: Usuario
}

const novaAnotacao = ref<Anotacao>({ texto: '', dataHora: '', usuario: { id: 1 } })
const anotacoes = ref<Anotacao[]>([])
const usuarios = ref<Usuario[]>([])
const erro = ref<string>('')
const textoPesquisa = ref<string>('')
const nomeUsuarioPesquisa = ref<string>('')

async function cadastrar() {
  try {
    erro.value = ''
    await axios.post('anotacao', novaAnotacao.value)
    await atualizar()
    novaAnotacao.value.texto = ''
    novaAnotacao.value.dataHora = ''
  } catch (e) {
    erro.value = (e as Error).message
  }
}

async function atualizar() {
  anotacoes.value = (await axios.get('anotacao')).data
}

async function pesquisar() {
  anotacoes.value = (
    await axios.get('anotacao/busca', {
      params: {
        texto: textoPesquisa.value,
        usuario: nomeUsuarioPesquisa.value,
      },
    })
  ).data
}

async function buscarUsuarios() {
  usuarios.value = (await axios.get('usuario')).data
}

function antiguidade(data: string) {
  const diff = Date.now() - Date.parse(data)
  const dia = 1000 * 60 * 60 * 24
  return Math.round(Math.abs(diff) / dia)
}

onMounted(() => {
  atualizar()
  buscarUsuarios()
})
</script>