# ToDo List Application (Frontend)

## 📝 Descrição
Frontend da aplicação ToDo List desenvolvida com Vue.js 3 e Quasar Framework. Este é o cliente que consome a API do [backend](https://github.com/joykepler/ToDo-Backend).

## 🛠️ Tecnologias
- Vue.js 3 (Composition API)
- Quasar Framework v2.x
- TypeScript
- Axios
- jsPDF

## ⚙️ Requisitos
- Node.js (v14 ou superior)
- Backend configurado e rodando

## 🚀 Instalação e Configuração

1. Clone o repositório
```bash
git clone https://github.com/joykepler/ToDo-Frontend.git
cd ToDo-Frontend
```
2. Instale as dependências
```bash
npm install
```

3. Configure o arquivo .env

VITE_API_URL=http://localhost:8000/api

4. Inicie o servidor de desenvolvimento
```bash
npm run dev
```

## 🔗 Repositório Backend
https://github.com/joykepler/ToDo-Backend

## 💡 Funcionalidades

## Funcionalidades Principais

- Criar, editar e excluir tarefas
- Marcar tarefas como concluídas
- Adicionar descrições às tarefas
- Categorizar tarefas
- Definir prioridades
- Visualizar status das tarefas

## Funcionalidades Extras
## 📄 Exportação para PDF
A aplicação permite exportar todas as tarefas para um arquivo PDF organizado.

Como testar:

Adicione algumas tarefas à lista
Clique no botão "Exportar PDF" localizado no topo da lista
Um arquivo PDF será gerado e baixado automaticamente
O PDF incluirá:

- Título da tarefa
- Descrição
- Status
- Prioridade
- Categoria
- Estado de conclusão

## 🎯 Como Usar
Acesse a aplicação através do navegador: http://localhost:5173

-Para adicionar uma nova tarefa:
Digite o título no campo de entrada

Pressione Enter ou clique no botão de adicionar

-Para editar uma tarefa:
Clique no ícone de edição

Faça as alterações necessárias

Salve as mudanças

-Para marcar como concluída:
Clique no checkbox ao lado da tarefa
