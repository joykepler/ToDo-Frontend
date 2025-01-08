<template>
  <q-page class="q-pa-md column items-center justify-center">
    <div class="column items-center">
      <h1 class="text-h4 text-weight-bold text-primary q-mb-lg text-deep-purple-9">ToDo List!</h1>
    </div>
    <div class="row items-center justify-center q-gutter-md" style="max-width: 500px; width: 100%;">
      <q-input
        outlined
        v-model="newTask.title"
        placeholder="Insira sua tarefa"
        style="width: 80%; height: 100%;"
      />
      <q-btn
        unelevated
        size="md"
        @click="addTask"
        color="deep-purple-7"
        text-color="white"
        class="items-center justify-center"
        style="height: 56px"
      >
        <q-icon name="fa-solid fa-plus"/>
      </q-btn>
    </div>

    <div class="q-mt-md" style="width: 100%; max-width: 500px;">
      <q-list bordered separator>
        <q-item v-for="task in tasks" :key="task.id" clickable v-ripple @click="openTaskModal(task)">
          <q-item-section>
            <q-item-label>{{ task.title }}</q-item-label>
            <q-item-label caption>{{ task.status }}</q-item-label>
          </q-item-section>
          <q-item-section side>
            <q-btn flat round color="red" icon="delete" @click.stop="deleteTask(task.id)" />
          </q-item-section>
        </q-item>
      </q-list>
    </div>

    <!-- Modal para edição/visualização de tarefa -->
    <q-dialog v-model="taskModalOpen" persistent>
      <q-card style="min-width: 350px">
        <q-card-section>
          <div class="text-h6">{{ editingTask.id ? 'Editar Tarefa' : 'Adicionar Tarefa' }}</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-input v-model="editingTask.title" label="Título" :rules="[val => !!val || 'Campo obrigatório']" />
          <q-input v-model="editingTask.description" label="Descrição" type="textarea" />
          <q-select v-model="editingTask.status" :options="statusOptions" label="Status" />
        </q-card-section>

        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Cancelar" v-close-popup />
          <q-btn flat label="Salvar" @click="saveTask" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';
import { Notify } from 'quasar';

interface Task {
  id: number;
  title: string;
  description: string;
  status: string;
}

const tasks = ref<Task[]>([]);
const newTask = ref({
  title: '',
  description: '',
  status: 'pendente'
});
const editingTask = ref<Task>({ id: 0, title: '', description: '', status: 'pendente' });
const taskModalOpen = ref(false);

const statusOptions = [
  'pendente',
  'em andamento',
  'concluída'
];

const fetchTasks = async () => {
  try {
    const response = await axios.get('http://localhost:8000/api/tasks');
    tasks.value = response.data;
  } catch (error) {
    console.error('Erro ao buscar tarefas:', error);
    Notify.create({
      type: 'negative',
      message: 'Erro ao carregar tarefas'
    });
  }
};

const addTask = async () => {
  if (newTask.value.title.trim() !== '') {
    try {
      await axios.post('http://localhost:8000/api/tasks', newTask.value);
      newTask.value.title = '';
      await fetchTasks();
      Notify.create({
        type: 'positive',
        message: 'Tarefa adicionada com sucesso!'
      });
    } catch (error) {
      console.error('Erro ao adicionar tarefa:', error);
      Notify.create({
        type: 'negative',
        message: 'Erro ao adicionar tarefa'
      });
    }
  } else {
    Notify.create({
      type: 'warning',
      message: 'Por favor, insira um título para a tarefa'
    });
  }
};

const openTaskModal = (task?: Task) => {
  if (task) {
    editingTask.value = { ...task };
  } else {
    editingTask.value = { id: 0, title: '', description: '', status: 'pendente' };
  }
  taskModalOpen.value = true;
};

const saveTask = async () => {
  try {
    if (editingTask.value.id) {
      await axios.put(`http://localhost:8000/api/tasks/${editingTask.value.id}`, editingTask.value);
      Notify.create({
        type: 'positive',
        message: 'Tarefa atualizada com sucesso!'
      });
    } else {
      await axios.post('http://localhost:8000/api/tasks', editingTask.value);
      Notify.create({
        type: 'positive',
        message: 'Tarefa adicionada com sucesso!'
      });
    }
    await fetchTasks();
  } catch (error) {
    console.error('Erro ao salvar tarefa:', error);
    Notify.create({
      type: 'negative',
      message: 'Erro ao salvar tarefa'
    });
  }
};

const deleteTask = async (id: number) => {
  try {
    await axios.delete(`http://localhost:8000/api/tasks/${id}`);
    await fetchTasks();
    Notify.create({
      type: 'positive',
      message: 'Tarefa excluída com sucesso!'
    });
  } catch (error) {
    console.error('Erro ao excluir tarefa:', error);
    Notify.create({
      type: 'negative',
      message: 'Erro ao excluir tarefa'
    });
  }
};

onMounted(fetchTasks);
</script>
