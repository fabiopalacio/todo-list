<script setup>
//Aplicação feita a partir do video: 
//https://www.youtube.com/watch?v=qhjxAP1hFuI

import {ref, onMounted, computed, watch} from 'vue'

//Criação das variaveis utilizadas pela aplicação
  const todos = ref([])
  const name = ref('')

  const input_content = ref('')
  const input_category = ref(null)

  const isModalVisible = ref(false)

//Ordena as tarefas existentes na variável todos em ordem ascendente de data de criação
//e as salva na variável todos_asc
  const todos_asc = computed(() => todos.value.sort((a, b)=>{
    return b.createdAt - a.createdAt
  }))

const todos_length = computed(() => todos.value.length !== 0)


//Adiciona uma tarefa a lista de tarefas. A data é necessária para ordenar a lista
//com as tarefas adicionadas mais recentemente no topo
  const addTodo = () => {
    if(input_content.value.trim() === '' || input_category.value === null){
      return
    }
    todos.value.push({
      content: input_content.value,
      category: input_category.value,
      done: false,
      createdAt: new Date().getTime()
    })

    input_content.value = ''
    input_category.value = null
  }
//Função para remover a tarefa selecionada. Esta função recebe uma tarefa e retorna
//todas as tarefas que sejam diferentes da tarefa selecionada
  const removeTodo =(todo) => {
    todos.value = todos.value.filter((t) => t!==todo)
  }

  //Inverte o valor da modal de alerta.
  const toggleModal = () => {
    isModalVisible.value = !isModalVisible.value
  }

  //Função para apagar todas as informações salvas
  const removeAll = () => {
      todos.value = []
      name.value = ''
      toggleModal()
    
  }


//watch() para adicionar as novas informações ao localstorage. No caso
//dos "todos", é necessário que se use o deep: true para verificar se
//algum valor dentro do array se modificou. Caso contrário, o watch()
//não perceberia as modificações feitas via push()

  watch(name, (newVal) => {
    localStorage.setItem('name', newVal)
  })

  watch(todos, (newVal) => {
    localStorage.setItem('todos', JSON.stringify(newVal))
  }, {deep: true})

//Recupera os valores salvos no localstorage
  onMounted(()=>{
    name.value = localStorage.getItem('name') || ''
    todos.value = JSON.parse(localStorage.getItem('todos')) || []
  })
</script>

<template>
  
  <main class="app">

    <section class="greeting">
      <h2 class="title">
        E aí, <input type="text" placeholder="Seu nome aqui" v-model="name"/>
      </h2>
    </section>

    <section class="create-todo">
      <h3>ADICIONAR TAREFA</h3>
      <form @submit.prevent="addTodo">
        <h4>O que está em sua lista de tarefas?</h4>
        <input 
          type="text"
          placeholder="Insira a tarefa"
          v-model="input_content"/>
        <h4>
          Escolha uma categoria
        </h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              value="profissional"
              v-model="input_category"/>
              <span class="bubble profissional"></span>
              <span class="labelText">Profissional</span>
          </label>

          <label>
            <input 
              type="radio"
              name="category"
              value="personal"
              v-model="input_category"/>
              <span class="bubble personal"></span>
              <div class="labelText">Pessoal</div>
          </label>
        </div>
        <input type="submit" value="Adicionar"/>
      </form>

    </section>

    
    <section class="todo-list" v-if="todos_length">
      <h3>LISTA DE TAREFAS</h3>
      <div class="list">
        <div v-for="todo in todos_asc" :class="`todo-item ${todo.done && 'done'} ${todo.category == 'profissional' ? 'shadowprofissional' : 'shadowpersonal'}`">
          <div class="dflex">
          <label>
            <input type="checkbox" v-model="todo.done">
            <span :class="`bubble ${todo.category == 'profissional' ? 'profissional' : 'personal'}`"></span>
          </label>
        
          <input class="todo-content" type="text" placeholder="Sua tarefa" v-model="todo.content"/>

        </div>
          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Remover</button>
          </div>
        </div>
      </div>
      <button type="button" class="deleteAll" @click=toggleModal()>Apagar tudo</button>
    </section>


    <section class="modalAlert" v-if="isModalVisible">
      <div class="backgroundModal" @click="toggleModal()"></div>
      <div class="modalWindow">
        <p>Deseja remover o nome e as tarefas salvas?</p>
        <div>
          <button class="buttonDelete" @click="removeAll()">Sim</button>
          <button class="buttonCancelDelete" @click="toggleModal()">Não</button>
        </div>
      </div>
    </section>
  </main>

</template>
