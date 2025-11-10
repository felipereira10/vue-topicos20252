<script setup lang="ts">
import TheWelcome from '../components/TheWelcome.vue'
</script>

<template>
  <main
    class="min-h-screen flex flex-col items-center justify-center bg-gradient-to-br from-blue-50 via-white to-blue-100 p-6"
  >
    <section
      class="bg-white shadow-lg rounded-2xl p-8 w-full max-w-3xl border border-gray-100 transition-transform hover:scale-[1.01]"
    >
      <header class="text-center mb-6">
        <h1 class="text-3xl font-bold text-gray-800 mb-2">Bem-vindo</h1>
        <p class="text-gray-500">Explore as funcionalidades abaixo</p>
      </header>

      <TheWelcome />

      <footer class="text-center mt-6 text-sm text-gray-400">
        © {{ new Date().getFullYear() }} Seu Projeto. Todos os direitos reservados.
      </footer>
    </section>
  </main>
</template>
