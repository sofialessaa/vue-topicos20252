<template>
  <div>
    <form @submit.prevent="cadastrar"> <!-- usamos o .prevent para o submit nao dar o reload -->
      <p>
        <label id="feedbackLabel" for="feedback">Feedback: </label>
        <input type="text" id="feedback" v-model="novaRevisao.feedback" required />
      </p>
      <p>
        <label id="dataHoraCriacaoLabel" for="dataHoraCriacao">Data/Hora de criação: </label>
        <input type="datetime-local" id="dataHoraCriacao" v-model="novaRevisao.dataHoraCriacao" required />
      </p>
      <p>
        <label id="dataHoraCorrecaoLabel" for="dataHoraCorrecao">Data/Hora de correção: </label>
        <input type="datetime-local" id="dataHoraCorrecao" v-model="novaRevisao.dataHoraCorrecao" />
      </p>
      <p>
        <label id="secaoLabel" for="secao">Seção: </label>
        <select id="secao" v-model="novaRevisao.secao.id" required>
          <option value="" disabled>Selecione uma seção</option>

          <option v-for="secao in secoes" :key="secao.id" :value="secao.id">
            {{ secao.titulo }} - {{ secao.trabalho?.titulo }}
          </option>
        </select>
      </p>
      <button type="submit">Cadastrar</button>
    </form>
  </div>
  <div>
    <form @submit.prevent="pesquisar">
      <p>
        <label id="secaoPesquisaLabel" for="secaoPesquisaInput">Título da Seção: </label>
        <input type="text" id="secaoPesquisaInput" v-model="secaoPesquisa" required />
      </p>
      <p>
        <label id="dataHoraCriacaoPesquisaLabel" for="dataHoraCriacaoPesquisaInput">Data/Hora de criação: </label>
        <input type="datetime-local" id="dataHoraCriacaoPesquisaInput" v-model="dataHoraCriacaoPesquisa" required />
      </p>
      <button type="submit">Pesquisar</button>
    </form>
  </div>
  <div>
    <table v-if="revisao.length > 0">
      <thead>
        <tr>
          <th>Id</th>
          <th>Feedback</th>
          <th>Título da Seção</th>
          <th>Título do Trabalho</th>
          <th>Situação</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="revisao in revisao" :key="revisao.id">
          <th>{{ revisao.id }}</th>
          <th>{{ revisao.feedback }}</th>
          <th>{{ revisao.secao.titulo }}</th>
          <th>{{ revisao.secao.trabalho?.titulo }}</th>
          <th>{{ calcularSituacao(revisao.dataHoraCriacao, revisao.dataHoraCorrecao) }}</th>
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

interface trabalho {
  id: number
  titulo?: string
  dataHoraEntrega?: string
  descricao?: string
  nota?: number
  //nao preciso do usuario
}

interface secao {
  id: number //id obrigatório e o resto não é
  titulo?: string
  dataHoraCriacao?: string
  conteudo?: string
  trabalho?: trabalho
}

interface revisao {
  id?: number //coloco ? dps do id pq ele pode ser nulo (post)
  feedback: string
  dataHoraCriacao: string
  dataHoraCorrecao: string
  secao: secao
}

const revisao = ref<revisao[]>([])

const formLimpo = () => ({
  feedback: '',
  dataHoraCriacao: '',
  dataHoraCorrecao: '',
  secao: { id: '' }
})
const novaRevisao = ref(formLimpo())
const erro = ref<string>('')
const secoes = ref<secao[]>([])

const secaoPesquisa = ref<string>('')
const dataHoraCriacaoPesquisa = ref<string>('')

function calcularSituacao(dataHoraCriacao: string, dataHoraCorrecao: string): string {
  if (!dataHoraCorrecao) {
    return 'pendente'
  }
  // 2. Converte as strings de data em objetos Date
  const criacao = new Date(dataHoraCriacao)
  const correcao = new Date(dataHoraCorrecao)

  // 3. Calcula a diferença em milissegundos
  const diffMs = correcao.getTime() - criacao.getTime()

  // 4. Converte a diferença de milissegundos para horas e arredonda
  const diffHoras = Math.round(diffMs / (1000 * 60 * 60))

  // 5. Retorna a string formatada
  return `revisada em ${diffHoras} horas`
}

async function atualizar() {
  revisao.value = (await axios.get('revisao')).data
}

async function cadastrar() {
  try {
    erro.value = ''

    const payload = {
      ...novaRevisao.value,
      dataHoraCorrecao: novaRevisao.value.dataHoraCorrecao || null
    }

    await axios.post('revisao', payload)
    await atualizar()

    novaRevisao.value = formLimpo()

  } catch (e) {
    erro.value = (e as Error).message
  }
}

async function buscarSecoes() {
  secoes.value = (await axios.get('secao')).data
}

async function pesquisar() {
  revisao.value = (
    await axios.get('revisao/busca', {
      params: {
        secao: secaoPesquisa.value,
        dataHoraCriacao: dataHoraCriacaoPesquisa.value,
      },
    })
  ).data
}

onMounted(() => {
  atualizar()
  buscarSecoes()
})

</script>
