<template>
  <q-page class="q-pa-md column items-center justify-center">
    <div class="column items-center">
      <h1 class="text-h4 text-weight-bold text-primary q-mb-lg text-deep-purple-9">ToDo List!</h1>
    </div>

    <div class="row items-center justify-center q-gutter-sm" style="max-width: 620px; width: 100%;">
      <q-input
        outlined
        v-model="newTask.titulo"
        placeholder="Insira sua tarefa"
        style="width: 70%; height: 100%;"
        @keyup.enter="addTask"
        color="deep-purple-9"
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
        <q-item v-for="task in tasks" :key="task.id" clickable v-ripple>
          <q-item-section avatar>
            <q-checkbox
            v-model="task.completed"
            @update:model-value="updateTaskStatus(task)"
            color="deep-purple-7"
            @click.stop
            :disable="updating"
  />
</q-item-section>
          <q-item-section @click="openTaskModal(task)">
            <q-item-label :class="{ 'text-strike': task.completed }">{{ task.titulo }}</q-item-label>
            <q-item-label caption>{{ task.status }}</q-item-label>
            <q-item-label caption v-if="task.description">
              {{ truncateDescription(task.description) }}
            </q-item-label>
          </q-item-section>
          <q-item-section side>
            <div class="row items-center">
              <q-btn flat round color="deep-purple-7" icon="edit" @click.stop="openTaskModal(task)" />
              <q-btn flat round color="red" icon="delete" @click.stop="confirmDeleteTask(task)" />
            </div>
          </q-item-section>
        </q-item>
      </q-list>
    </div>

    <!-- Modal para edição/visualização de tarefa -->
    <q-dialog v-model="taskModalOpen" persistent>
      <q-card style="min-width: 350px">
        <q-card-section class="row items-center">
          <div class="text-h6">{{ editingTask.id ? 'Editar Tarefa' : 'Adicionar Tarefa' }}</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-input
            v-model="editingTask.titulo"
            label="Título"
            :rules="[val => !!val || 'Campo obrigatório']"
            class="q-mb-md"
          />
          <q-input
            v-model="editingTask.description"
            label="Descrição"
            type="textarea"
            class="q-mb-md"
          />
          <q-select
            v-model="editingTask.status"
            :options="statusOptions"
            option-value="value"
            option-label="label"
            emit-value
            label="Status"
            class="q-mb-md"
          />
          <q-select
            v-model="editingTask.prioridade"
            :options="prioridadeOptions"
            option-value="value"
            option-label="label"
            emit-value
            label="Prioridade"
            class="q-mb-md"
          />
          <q-select
            v-if="categorias.length"
            v-model="editingTask.categoria_id"
            :options="categorias"
            option-label="nome"
            option-value="id"
            label="Categoria"
            class="q-mb-md"
          />
        </q-card-section>

        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Cancelar" v-close-popup />
          <q-btn flat label="Salvar" @click="saveTask" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Dialog de confirmação para deletar -->
    <q-dialog v-model="deleteConfirmOpen" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <span class="q-ml-sm">Tem certeza que deseja excluir esta tarefa?</span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancelar" color="primary" v-close-popup />
          <q-btn flat label="Excluir" color="negative" @click="deleteTask" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <div
    class="q-mt-md"
    style="width: 100%; max-width: 500px; display: flex; justify-content: flex-end;">
      <q-btn
        unelevated
        color="deep-purple-7"
        icon="fa-solid fa-file-pdf"
        label="Exportar PDF"
        @click="exportToPDF"
        class="q-my-md"
      />
    </div>
  </q-page>
</template>


<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';
import { Notify } from 'quasar';
import { jsPDF } from 'jspdf';

interface Categoria {
  id: number;
  nome: string;
  cor: string;
}

interface Task {
  id: number;
  titulo: string;
  description: string;
  status: string;
  completed: boolean;
  prioridade?: string;
  categoria_id?: number | null;
}

interface TaskOption {
  label: string;
  value: string;
}

const tasks = ref<Task[]>([]);
const categorias = ref<Categoria[]>([]);

const newTask = ref({
  titulo: '',
  description: '',
  status: 'pendente',
  completed: false,
  prioridade: 'media',
  categoria_id: null as number | null
});

const editingTask = ref<Task>({
  id: 0,
  titulo: '',
  description: '',
  status: 'pendente',
  completed: false,
  prioridade: 'media',
  categoria_id: null
});

const taskModalOpen = ref(false);
const deleteConfirmOpen = ref(false);
const taskToDelete = ref<number | null>(null);
const updating = ref(false);

const statusOptions: TaskOption[] = [
  { label: 'PENDENTE', value: 'pendente' },
  { label: 'EM ANDAMENTO', value: 'em_andamento' },
  { label: 'CONCLUÍDA', value: 'concluida' }
];

const prioridadeOptions: TaskOption[] = [
  { label: 'BAIXA', value: 'baixa' },
  { label: 'MÉDIA', value: 'media' },
  { label: 'ALTA', value: 'alta' }
];

// const formatStatus = (status: string): string => {
//   switch (status) {
//     case 'em_andamento':
//       return 'Em andamento';
//     case 'concluida':
//       return 'Concluída';
//     default:
//       return status.charAt(0).toUpperCase() + status.slice(1);
//   }
// };

const fetchTasks = async () => {
  try {
    const response = await axios.get('http://localhost:8000/api/tarefas');
    tasks.value = response.data.map((task: Task) => ({
      ...task,
      completed: task.status === 'concluida'
    }));
  } catch (error) {
    console.error('Erro ao buscar tarefas:', error);
    Notify.create({
      type: 'negative',
      message: 'Erro ao carregar tarefas'
    });
  }
};

const fetchCategorias = async () => {
  try {
    const response = await axios.get('http://localhost:8000/api/categorias');
    categorias.value = response.data;
  } catch (error) {
    console.error('Erro ao buscar categorias:', error);
  }
};

const truncateDescription = (description: string) => {
  return description.length > 50 ? description.substring(0, 50) + '...' : description;
};

const addTask = async () => {
  if (newTask.value.titulo.trim() !== '') {
    try {
      await axios.post('http://localhost:8000/api/tarefas', newTask.value);
      newTask.value.titulo = '';
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
    editingTask.value = {
      ...task,
      status: task.status,
      prioridade: task.prioridade || 'media'
    };
  } else {
    editingTask.value = {
      id: 0,
      titulo: '',
      description: '',
      status: 'pendente',
      completed: false,
      prioridade: 'media',
      categoria_id: null
    };
  }
  taskModalOpen.value = true;
};

const saveTask = async () => {
  try {
    if (!editingTask.value.titulo.trim()) {
      Notify.create({
        type: 'warning',
        message: 'O título da tarefa não pode estar vazio'
      });
      return;
    }

    const taskData = {
      ...editingTask.value,
      status: editingTask.value.status,
      prioridade: editingTask.value.prioridade,
      completed: editingTask.value.status === 'concluida'
    };

    if (editingTask.value.id) {
      await axios.put(`http://localhost:8000/api/tarefas/${editingTask.value.id}`, taskData);
      Notify.create({
        type: 'positive',
        message: 'Tarefa atualizada com sucesso!'
      });
    } else {
      await axios.post('http://localhost:8000/api/tarefas', taskData);
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

const updateTaskStatus = async (task: Task) => {
  updating.value = true;
  try {
    const updatedTask = {
      ...task,
      status: task.completed ? 'concluida' : 'pendente'
    };

    await axios.put(`http://localhost:8000/api/tarefas/${task.id}`, updatedTask);

    const taskIndex = tasks.value.findIndex(t => t.id === task.id);
    if (taskIndex !== -1) {
      tasks.value[taskIndex] = {
        ...updatedTask,
        completed: task.completed
      };
    }

    Notify.create({
      type: 'positive',
      message: `Tarefa marcada como ${task.completed ? 'concluida' : 'pendente'}!`
    });
  } catch (error) {
    task.completed = !task.completed;
    console.error('Erro ao atualizar status da tarefa:', error);
    Notify.create({
      type: 'negative',
      message: 'Erro ao atualizar status da tarefa'
    });
  } finally {
    updating.value = false;
  }
};

const confirmDeleteTask = (task: Task) => {
  taskToDelete.value = task.id;
  deleteConfirmOpen.value = true;
};

const deleteTask = async () => {
  if (taskToDelete.value) {
    try {
      await axios.delete(`http://localhost:8000/api/tarefas/${taskToDelete.value}`);
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
    taskToDelete.value = null;
  }
};

onMounted(() => {
  fetchTasks();
  fetchCategorias();
});

//export pdf

const exportToPDF = () => {
  try {
    const doc = new jsPDF({
      orientation: 'portrait',
      unit: 'pt',
      format: 'a4',
      putOnlyUsedFonts: true,
      floatPrecision: 16
    });

    doc.setFont('helvetica', 'normal');

    // Função para limpar texto
    const cleanText = (text: string) => {
      return text.normalize('NFD').replace(/[\u0300-\u036f]/g, '');
    };

    doc.setFontSize(20);
    doc.setTextColor(89, 54, 148);
    doc.text('Lista de Tarefas', 40, 40);

    doc.setFontSize(12);
    doc.setTextColor(0, 0, 0);

    let yPosition = 80;

    // Iterar sobre as tarefas
    tasks.value.forEach((task) => {
      const taskStatus = task.completed ? 'X' : 'O';
      const taskTitle = cleanText(`${taskStatus} ${task.titulo}`);
      doc.setFont('helvetica', 'normal');
      doc.text(taskTitle, 40, yPosition);

      // Descrição
      if (task.description) {
        yPosition += 20;
        doc.setFont('helvetica', 'italic');
        doc.setFontSize(10);
        const description = cleanText(`Descricao: ${truncateDescription(task.description)}`);
        doc.text(description, 50, yPosition);
        doc.setFontSize(12);
      }

      // Status e Prioridade
      yPosition += 20;
      doc.setFont('helvetica', 'normal');
      const statusText = cleanText(`Status: ${task.status} | Prioridade: ${task.prioridade || 'Nao definida'}`);
      doc.text(statusText, 50, yPosition);

      // Categoria
      const categoria = categorias.value.find(cat => cat.id === task.categoria_id);
      if (categoria) {
        yPosition += 20;
        const categoriaText = cleanText(`Categoria: ${categoria.nome}`);
        doc.text(categoriaText, 50, yPosition);
      }

      // Espaçamento entre tarefas
      yPosition += 30;

      if (yPosition > 750) {
        doc.addPage();
        yPosition = 40;
      }
    });

    const date = new Date();
    const fileName = `todo-list-${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`;

    doc.save(`${fileName}.pdf`);

    Notify.create({
      type: 'positive',
      message: 'PDF exportado com sucesso!'
    });

  } catch (error) {
    console.error('Erro ao exportar PDF:', error);
    Notify.create({
      type: 'negative',
      message: 'Erro ao exportar PDF'
    });
  }
};

</script>

<style>
.text-strike {
  text-decoration: line-through;
}
</style>
