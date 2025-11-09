<template>
  <div>
    {{ nome }} - {{ senha }}
    <p>
      <label for="nome">Nome: </label>
      <input id="nome" type="text" v-model="nome" />
      <input id="nome2" type="text" :value="nome" />
    </p>
    <p>
      <label for="senha">Senha: </label>
      <input id="senha" type="text" v-model="senha" />
    </p>
    <button @click="inserirUsuario">Inserir</button>
    <button @click="atualizar">Atualizar</button>
    <p v-if="erro">{{ erro }}</p>
    <p v-else>Tudo ok!</p>
    <table>
      <thead>
        <tr>
          <th>Id</th>
          <th>Nome</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="usuario in usuarios" :key="usuario.id">
          <th>{{ usuario.id }}</th>
          <th>{{ usuario.nome }}</th>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'
import { onMounted } from 'vue' //uso para buscar os dados quado abrir a página

interface usuario {
  id?: number
  nome: string
  senha: string
}

const nome = ref<string>('Teste')
const senha = ref<string>('senha')
const erro = ref<string>()
const usuarios = ref<usuario[]>([
  { id: 1, nome: 'administrador', senha: 'admin' },
  { id: 2, nome: 'usuario', senha: 'teste' },
])

async function inserirUsuario() {
  try {
    await axios.post('usuario', {
      nome: nome.value,
      senha: senha.value,
    })
    nome.value = ''
    senha.value = ''
    atualizar()
    erro.value = undefined
  } catch (error) {
    erro.value = (error as Error).message
  }
}

async function atualizar() {
  usuarios.value = (await axios.get('usuario')).data
}

onMounted(() => {
  atualizar()
})
</script>
