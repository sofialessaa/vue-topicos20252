<template>
  <div>
    <form @submit.prevent="cadastrar"> <!-- usamos o .prevent para o submit nao dar o reload -->
      <p>
        <label id="textoLabel" for="texto">Texto: </label>
        <input type="text" id="texto" v-model="novaAnotacao.texto" required />
      </p>
      <p>
        <label id="dataHoraLabel" for="dataHora">Data/Hora: </label>
        <input type="datetime-local" id="dataHora" v-model="novaAnotacao.dataHora" required />
      </p>
      <p>
        <label id="usuarioLabel" for="usuario">Usuário: </label>
        <select id="usuario" v-model="novaAnotacao.usuario.id" required>
          <option v-for="usuario in usuarios" :key="usuario.id" :value="usuario.id">
            {{ usuario.nome }}
          </option>
        </select>
        <input type="number" id="usuarioInput" v-model="novaAnotacao.usuario.id" required />
      </p>
      <button type="submit">Cadastrar</button>
    </form>
    <form @submit.prevent="pesquisar">
      <p>
        <label id="textPesquisaLabel" for="textoPesquisa">Texto: </label>
        <input type="text" id="textoPesquisa" v-model="textoPesquisa" required />
      </p>
      <p>
        <label id="usuarioPesquisaLabel" for="nomeUsuarioPesquisa">Usuário: </label>
        <select id="nomeUsuarioPesquisa" v-model="nomeUsuarioPesquisa" required>
          <option v-for="usuario in usuarios" :key="usuario.id" :value="usuario.nome">
            {{ usuario.nome }}
          </option>
        </select>
        <input type="text" id="nomeUsuarioPesquisaInput" v-model="nomeUsuarioPesquisa" required />
      </p>
      <button type="submit">Pesquisar</button>
    </form>
    <table v-if="anotacoes.length > 0">
      <thead>
        <tr>
          <th>Id</th>
          <th>Texto</th>
          <th>Antiguidade</th>
          <th>Nome do Usuário</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="anotacao in anotacoes" :key="anotacao.id">
          <th>{{ anotacao.id }}</th>
          <th>{{ anotacao.texto }}</th>
          <th>{{ antiguidade(anotacao.dataHora) }}</th>
          <th>{{ anotacao.usuario.nome }}</th>
        </tr>
      </tbody>
    </table>
    <p v-else>Nenhum registro foi encontrado para os critérios fornecidos</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'
import { onMounted } from 'vue'

interface usuario {
  id: number
  nome?: string
  senha?: string
}

interface anotacao {
  id?: number
  texto: string
  dataHora: string
  usuario: usuario
}

const novaAnotacao = ref<anotacao>({ texto: '', dataHora: '', usuario: { id: 1 } })
const anotacoes = ref<anotacao[]>([{ id: 1, texto: 'bla', dataHora: '', usuario: { id: 1 } }])
const usuarios = ref<usuario[]>([{ id: 1, nome: 'admin' }])
const erro = ref<string>('')
const textoPesquisa = ref<string>('')
const nomeUsuarioPesquisa = ref<string>('')

async function cadastrar() {
  try {
    erro.value = ''
    await axios.post('anotacao', novaAnotacao.value)
    atualizar()
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
  // Calcula a diferença em milissegundos
  const diferencaEmMilissegundos = Date.now() - Date.parse(data)

  // Define o número de milissegundos em um dia
  const milissegundosPorDia = 1000 * 60 * 60 * 24

  // Calcula a diferença em dias (usando Math.abs para garantir que seja positivo)
  return Math.round(Math.abs(diferencaEmMilissegundos) / milissegundosPorDia)
}

onMounted(() => {
  atualizar()
  buscarUsuarios()
})
</script>
