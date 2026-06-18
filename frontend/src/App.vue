<template>
  <div id="app">
    <!-- Sidebar -->
    <div class="sidebar">
      <div class="logo">
        <h2 style="font-size: 16px; font-weight: 700; color: #2d3748;">💼My Task Management</h2>
      </div>
      
      <div class="search-box">
        <input 
          v-model="searchQuery" 
          @input="searchTasks"
          type="text" 
          placeholder="Search tasks..."
          class="search-input"
        >
      </div>

      <div class="menu-section">
        <h3>TASKS</h3>
        <ul>
          <li @click="filterTasks('all')" :class="{ active: currentFilter === 'all' && !currentListId }">
            All Tasks
          </li>
          <li @click="filterTasks('upcoming')" :class="{ active: currentFilter === 'upcoming' }">
            Upcoming
          </li>
          <li @click="filterTasks('today')" :class="{ active: currentFilter === 'today' }">
            Today
          </li>
          <li @click="filterTasks('sticky')" :class="{ active: currentFilter === 'sticky' }">
            Sticky Wall
          </li>
        </ul>
      </div>

      <div class="menu-section">
        <h3>LISTS</h3>
        <ul>
          <li v-for="list in lists" :key="list.id" 
              @click="filterByList(list)"
              :class="{ active: currentListId === list.id }">
            {{ list.name }}
          </li>
        </ul>
        <button @click="showAddListModal = true" class="add-list-btn">
          + Add List
        </button>
      </div>
    </div>

    <!-- Main Content -->
    <div class="main-content">
      <div class="header">
        <h1>{{ pageTitle }}</h1>
        <button @click="openAddTaskModal" class="add-task-btn">
          + Add Task
        </button>
      </div>

      <!-- 3 Cards Layout -->
      <div class="task-board">
        <!-- Card 1: Pending Tasks -->
        <div class="task-card-column">
          <div class="card-header pending">
            <h3>Pending</h3>
            <span class="badge">{{ getTasksByStatus('pending').length }}</span>
          </div>
          <div class="card-body">
            <div v-if="getTasksByStatus('pending').length === 0" class="empty-card">
              <p>No pending tasks yet.</p>
            </div>
            <div v-for="task in getTasksByStatus('pending')" :key="task.id" 
                 class="task-item" 
                 :class="{ sticky: task.is_sticky }">
              <div class="task-content">
                <div class="task-title">
                  <h4>{{ task.title }}</h4>
                  <div class="task-actions">
                    <button @click="toggleSticky(task)" class="sticky-btn" :title="task.is_sticky ? 'Remove from sticky' : 'Add to sticky'">
                      {{ task.is_sticky ? 'Unpin' : 'Pin' }}
                    </button>
                    <button @click="deleteTask(task.id)" class="delete-btn">Delete</button>
                  </div>
                </div>
                <p class="task-desc">{{ task.description || 'No description' }}</p>
                <div class="task-info">
                  <span class="priority" :class="task.priority">{{ task.priority }}</span>
                  <span class="due-date">Due: {{ formatDate(task.due_date) }}</span>
                  <span class="task-list-name">{{ task.list_name }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Card 2: In Progress Tasks -->
        <div class="task-card-column">
          <div class="card-header in-progress">
            <h3>In Progress</h3>
            <span class="badge">{{ getTasksByStatus('in_progress').length }}</span>
          </div>
          <div class="card-body">
            <div v-if="getTasksByStatus('in_progress').length === 0" class="empty-card">
              <p>No tasks in progress yet.</p>
            </div>
            <div v-for="task in getTasksByStatus('in_progress')" :key="task.id" 
                 class="task-item" 
                 :class="{ sticky: task.is_sticky }">
              <div class="task-content">
                <div class="task-title">
                  <h4>{{ task.title }}</h4>
                  <div class="task-actions">
                    <button @click="toggleSticky(task)" class="sticky-btn" :title="task.is_sticky ? 'Remove from sticky' : 'Add to sticky'">
                      {{ task.is_sticky ? 'Unpin' : 'Pin' }}
                    </button>
                    <button @click="deleteTask(task.id)" class="delete-btn">Delete</button>
                  </div>
                </div>
                <p class="task-desc">{{ task.description || 'No description' }}</p>
                <div class="task-info">
                  <span class="priority" :class="task.priority">{{ task.priority }}</span>
                  <span class="due-date">Due: {{ formatDate(task.due_date) }}</span>
                  <span class="task-list-name">{{ task.list_name }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Card 3: Completed Tasks -->
        <div class="task-card-column">
          <div class="card-header completed">
            <h3>Completed</h3>
            <span class="badge">{{ getTasksByStatus('completed').length }}</span>
          </div>
          <div class="card-body">
            <div v-if="getTasksByStatus('completed').length === 0" class="empty-card">
              <p>No completed tasks yet.</p>
            </div>
            <div v-for="task in getTasksByStatus('completed')" :key="task.id" 
                 class="task-item completed-task" 
                 :class="{ sticky: task.is_sticky }">
              <div class="task-content">
                <div class="task-title">
                  <h4><s>{{ task.title }}</s></h4>
                  <div class="task-actions">
                    <button @click="toggleSticky(task)" class="sticky-btn" :title="task.is_sticky ? 'Remove from sticky' : 'Add to sticky'">
                      {{ task.is_sticky ? 'Unpin' : 'Pin' }}
                    </button>
                    <button @click="deleteTask(task.id)" class="delete-btn">Delete</button>
                  </div>
                </div>
                <p class="task-desc"><s>{{ task.description || 'No description' }}</s></p>
                <div class="task-info">
                  <span class="priority" :class="task.priority">{{ task.priority }}</span>
                  <span class="due-date">Due: {{ formatDate(task.due_date) }}</span>
                  <span class="task-list-name">{{ task.list_name }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Loading State -->
      <div v-if="loading" class="loading-overlay">
        <div class="spinner"></div>
        <p>Loading tasks...</p>
      </div>
    </div>

    <!-- Add Task Modal -->
    <div v-if="showAddTaskModal" class="modal-overlay" @click.self="closeModal('task')">
      <div class="modal">
        <h2>Add New Task</h2>
        <form @submit.prevent="addTask">
          <div class="form-group">
            <label>Title *</label>
            <input v-model="newTask.title" required>
          </div>
          <div class="form-group">
            <label>Description</label>
            <textarea v-model="newTask.description" rows="3"></textarea>
          </div>
          <div class="form-group">
            <label>List</label>
            <select v-model="newTask.task_list_id">
              <option v-for="list in lists" :key="list.id" :value="list.id">
                {{ list.name }}
              </option>
            </select>
          </div>
          <div class="form-group">
            <label>Priority</label>
            <select v-model="newTask.priority">
              <option value="low">Low</option>
              <option value="medium">Medium</option>
              <option value="high">High</option>
            </select>
          </div>
          <div class="form-group">
            <label>Status</label>
            <select v-model="newTask.status">
              <option value="pending">Pending</option>
              <option value="in_progress">In Progress</option>
              <option value="completed">Completed</option>
            </select>
          </div>
          <div class="form-group">
            <label>Due Date</label>
            <input v-model="newTask.due_date" type="date">
          </div>
          <div class="form-actions">
            <button type="button" @click="closeModal('task')">Cancel</button>
            <button type="submit">Add Task</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Add List Modal -->
    <div v-if="showAddListModal" class="modal-overlay" @click.self="closeModal('list')">
      <div class="modal">
        <h2>Add New List</h2>
        <form @submit.prevent="addList">
          <div class="form-group">
            <label>List Name *</label>
            <input v-model="newListName" required>
          </div>
          <div class="form-actions">
            <button type="button" @click="closeModal('list')">Cancel</button>
            <button type="submit">Add List</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  data() {
    return {
      tasks: [],
      lists: [],
      filteredTasks: [],
      currentFilter: 'all',
      currentListId: null,
      currentListName: '',
      searchQuery: '',
      loading: false,
      showAddTaskModal: false,
      showAddListModal: false,
      newListName: '',
      loadingTimer: null,
      newTask: {
        title: '',
        description: '',
        task_list_id: null,
        priority: 'medium',
        due_date: '',
        status: 'pending'
      },
      apiBase: '/api'
    }
  },
  computed: {
    pageTitle() {
      if (this.currentFilter === 'upcoming') return 'Upcoming Tasks';
      if (this.currentFilter === 'today') return "Today's Tasks";
      if (this.currentFilter === 'sticky') return 'Sticky Notes';
      if (this.currentListName) return `${this.currentListName} List`;
      return 'All Tasks';
    }
  },
  mounted() {
    this.fetchTasks();
    this.fetchLists();
  },
  methods: {
    startLoading() {
      this.loading = true;
      clearTimeout(this.loadingTimer);
      this.loadingTimer = setTimeout(() => {
        this.loading = false;
      }, 8000);
    },
    stopLoading() {
      clearTimeout(this.loadingTimer);
      this.loadingTimer = null;
      this.loading = false;
    },
    normalizeTask(task) {
      return {
        ...task,
        status: task.status || 'pending',
        list_name: task.list_name || (task.taskList && task.taskList.name) || 'Unassigned'
      };
    },
    getTasksByStatus(status) {
      return this.filteredTasks.filter(task => task.status === status);
    },
    async fetchTasks() {
      this.startLoading();
      try {
        const response = await axios.get(`${this.apiBase}/tasks`);
        this.tasks = response.data.map(task => this.normalizeTask(task));
        this.filteredTasks = this.tasks;
      } catch (error) {
        console.error('Error fetching tasks:', error);
        alert('Failed to load tasks. Please check your connection.');
      } finally {
        this.stopLoading();
      }
    },
    async fetchLists() {
      try {
        const response = await axios.get(`${this.apiBase}/task-lists`);
        this.lists = response.data;
        if (!this.newTask.task_list_id && this.lists.length > 0) {
          this.newTask.task_list_id = this.lists[0].id;
        }
      } catch (error) {
        console.error('Error fetching lists:', error);
      }
    },
    filterTasks(filter) {
      this.currentFilter = filter;
      this.currentListId = null;
      this.currentListName = '';
      this.searchQuery = '';

      if (filter === 'upcoming') {
        this.fetchUpcoming();
      } else if (filter === 'today') {
        this.fetchToday();
      } else if (filter === 'sticky') {
        this.fetchSticky();
      } else {
        this.filteredTasks = this.tasks;
      }
    },
    async fetchUpcoming() {
      this.startLoading();
      try {
        const response = await axios.get(`${this.apiBase}/tasks/upcoming`);
        this.filteredTasks = response.data.map(task => this.normalizeTask(task));
      } catch (error) {
        console.error('Error fetching upcoming tasks:', error);
      } finally {
        this.stopLoading();
      }
    },
    async fetchToday() {
      this.startLoading();
      try {
        const response = await axios.get(`${this.apiBase}/tasks/today`);
        this.filteredTasks = response.data.map(task => this.normalizeTask(task));
      } catch (error) {
        console.error("Error fetching today's tasks:", error);
      } finally {
        this.stopLoading();
      }
    },
    async fetchSticky() {
      this.startLoading();
      try {
        const response = await axios.get(`${this.apiBase}/tasks/sticky`);
        this.filteredTasks = response.data.map(task => this.normalizeTask(task));
      } catch (error) {
        console.error('Error fetching sticky tasks:', error);
      } finally {
        this.stopLoading();
      }
    },
    filterByList(list) {
      this.currentListId = list.id;
      this.currentListName = list.name;
      this.currentFilter = 'all';
      this.searchQuery = '';
      this.filteredTasks = this.tasks.filter(task => task.task_list_id === list.id);
    },
    openAddTaskModal() {
      this.resetNewTask();
      this.showAddTaskModal = true;
    },
    async searchTasks() {
      if (!this.searchQuery.trim()) {
        if (this.currentListId) {
          this.filteredTasks = this.tasks.filter(task => task.task_list_id === this.currentListId);
        } else {
          this.filteredTasks = this.tasks;
        }
        return;
      }

      try {
        const response = await axios.get(`${this.apiBase}/tasks?search=${this.searchQuery}`);
        this.filteredTasks = response.data.map(task => this.normalizeTask(task));
      } catch (error) {
        console.error('Error searching tasks:', error);
      }
    },
    async addTask() {
      try {
        const selectedListId = this.newTask.task_list_id;
        const response = await axios.post(`${this.apiBase}/tasks`, {
          title: this.newTask.title,
          description: this.newTask.description,
          task_list_id: selectedListId,
          priority: this.newTask.priority,
          due_date: this.newTask.due_date,
          status: this.newTask.status
        });
        const createdTask = this.normalizeTask({
          ...response.data,
          task_list_id: selectedListId,
          status: this.newTask.status || 'pending',
          taskList: this.lists.find(list => list.id === selectedListId) || response.data.taskList
        });
        this.tasks.unshift(createdTask);
        this.updateFilteredTasks();
        this.closeModal('task');
        this.resetNewTask();
      } catch (error) {
        console.error('Error adding task:', error);
        const message = error?.response?.data?.message
          || Object.values(error?.response?.data?.errors || {}).flat().join(' ')
          || error.message
          || 'Failed to add task. Please try again.';
        alert(message);
      }
    },
    async addList() {
      if (!this.newListName.trim()) return;

      try {
        const response = await axios.post(`${this.apiBase}/task-lists`, {
          name: this.newListName
        });
        this.lists.push(response.data);
        this.closeModal('list');
        this.newListName = '';
      } catch (error) {
        console.error('Error adding list:', error);
        alert('Failed to add list. Please try again.');
      }
    },
    async toggleSticky(task) {
      try {
        const response = await axios.post(`${this.apiBase}/tasks/${task.id}/toggle-sticky`);
        const index = this.tasks.findIndex(t => t.id === task.id);
        this.tasks[index] = this.normalizeTask(response.data);
        this.updateFilteredTasks();
      } catch (error) {
        console.error('Error toggling sticky:', error);
      }
    },
    async deleteTask(id) {
      if (!confirm('Are you sure you want to delete this task?')) return;

      try {
        await axios.delete(`${this.apiBase}/tasks/${id}`);
        this.tasks = this.tasks.filter(t => t.id !== id);
        this.updateFilteredTasks();
      } catch (error) {
        console.error('Error deleting task:', error);
        alert('Failed to delete task. Please try again.');
      }
    },
    updateFilteredTasks() {
      if (this.currentFilter === 'upcoming') {
        this.fetchUpcoming();
      } else if (this.currentFilter === 'today') {
        this.fetchToday();
      } else if (this.currentFilter === 'sticky') {
        this.fetchSticky();
      } else if (this.currentListId) {
        this.filteredTasks = this.tasks.filter(task => task.task_list_id === this.currentListId);
      } else {
        this.filteredTasks = this.tasks;
      }
    },
    closeModal(type) {
      if (type === 'task') {
        this.showAddTaskModal = false;
        this.resetNewTask();
      } else if (type === 'list') {
        this.showAddListModal = false;
        this.newListName = '';
      }
    },
    resetNewTask() {
      this.newTask = {
        title: '',
        description: '',
        task_list_id: this.currentListId || (this.lists.length > 0 ? this.lists[0].id : null),
        priority: 'medium',
        due_date: '',
        status: 'pending'
      };
    },
    formatDate(date) {
      if (!date) return 'No due date';
      return new Date(date).toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      });
    }
  }
}
</script>

<style scoped>
/* Task Board Layout */
.task-board {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
  margin-top: 20px;
}

/* Card Columns */
.task-card-column {
  background: #f8fafc;
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid #e2e8f0;
  min-height: 400px;
}

.card-header {
  padding: 16px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid #e2e8f0;
}

.card-header h3 {
  font-size: 16px;
  font-weight: 600;
  margin: 0;
}

.card-header.pending {
  background: #fef3c7;
  border-bottom-color: #f59e0b;
}

.card-header.in-progress {
  background: #dbeafe;
  border-bottom-color: #3b82f6;
}

.card-header.completed {
  background: #d1fae5;
  border-bottom-color: #10b981;
}

.badge {
  background: rgba(0,0,0,0.1);
  padding: 2px 10px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 600;
}

.card-body {
  padding: 16px;
  max-height: 500px;
  overflow-y: auto;
}

.card-body::-webkit-scrollbar {
  width: 4px;
}

.card-body::-webkit-scrollbar-thumb {
  background: #cbd5e0;
  border-radius: 2px;
}

/* Empty Card */
.empty-card {
  text-align: center;
  padding: 40px 20px;
  color: #a0aec0;
}

.empty-card p {
  font-size: 14px;
}

/* Task Items */
.task-item {
  background: white;
  border-radius: 8px;
  padding: 12px 16px;
  margin-bottom: 12px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.06);
  border-left: 3px solid transparent;
  transition: all 0.2s;
}

.task-item:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
  transform: translateY(-1px);
}

.task-item.sticky {
  border-left-color: #ed8936;
  background: #fffbeb;
}

.task-item.completed-task {
  opacity: 0.7;
}

.task-item.completed-task h4 {
  text-decoration: line-through;
  color: #a0aec0;
}

.task-item.completed-task .task-desc {
  text-decoration: line-through;
  color: #a0aec0;
}

.task-content {
  width: 100%;
}

.task-title {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 6px;
}

.task-title h4 {
  font-size: 14px;
  font-weight: 600;
  color: #2d3748;
  margin: 0;
  flex: 1;
}

.task-actions {
  display: flex;
  gap: 4px;
  flex-shrink: 0;
  margin-left: 8px;
}

.sticky-btn, .delete-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 14px;
  padding: 2px 4px;
  border-radius: 4px;
  transition: all 0.2s;
}

.sticky-btn:hover {
  background: #fef3c7;
  transform: scale(1.1);
}

.delete-btn {
  color: #fc8181;
}

.delete-btn:hover {
  background: #fff5f5;
  transform: scale(1.1);
}

.task-desc {
  font-size: 13px;
  color: #718096;
  margin: 4px 0 8px 0;
  line-height: 1.4;
}

.task-info {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  align-items: center;
}

.priority {
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 10px;
  font-weight: 600;
  text-transform: uppercase;
}

.priority.high {
  background: #fed7d7;
  color: #9b2c2c;
}

.priority.medium {
  background: #feebc8;
  color: #9c6b3e;
}

.priority.low {
  background: #c6f6d5;
  color: #22543d;
}

.due-date {
  font-size: 11px;
  color: #718096;
}

.task-list-name {
  font-size: 11px;
  color: #a0aec0;
  background: #edf2f7;
  padding: 2px 8px;
  border-radius: 12px;
}

/* Loading Spinner */
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255,255,255,0.8);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 999;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #e2e8f0;
  border-top: 4px solid #4299e1;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-overlay p {
  margin-top: 16px;
  color: #4a5568;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  backdrop-filter: blur(4px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  animation: fadeIn 0.2s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.modal {
  background: white;
  border-radius: 16px;
  padding: 32px;
  width: 520px;
  max-width: 92%;
  max-height: 90vh;
  overflow-y: auto;
  animation: slideUp 0.3s ease;
  box-shadow: 0 20px 60px rgba(0,0,0,0.2);
}

@keyframes slideUp {
  from { 
    opacity: 0;
    transform: translateY(20px);
  }
  to { 
    opacity: 1;
    transform: translateY(0);
  }
}

.modal h2 {
  margin-bottom: 24px;
  color: #2d3748;
  font-size: 24px;
  font-weight: 400;
}

.form-group {
  margin-bottom: 18px;
}

.form-group label {
  display: block;
  margin-bottom: 6px;
  color: #4a5568;
  font-size: 14px;
  font-weight: 600;
}

.form-group input,
.form-group textarea,
.form-group select {
  width: 100%;
  padding: 10px 14px;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 14px;
  transition: border-color 0.2s;
  font-family: inherit;
}

.form-group input:focus,
.form-group textarea:focus,
.form-group select:focus {
  outline: none;
  border-color: #4299e1;
  box-shadow: 0 0 0 3px rgba(66, 153, 225, 0.1);
}

.form-group textarea {
  min-height: 80px;
  resize: vertical;
}

.form-actions {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  margin-top: 24px;
}

.form-actions button {
  padding: 10px 28px;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.2s;
}

.form-actions button[type="button"] {
  background: #e2e8f0;
  color: #4a5568;
}

.form-actions button[type="button"]:hover {
  background: #cbd5e0;
}

.form-actions button[type="submit"] {
  background: #4299e1;
  color: white;
}

.form-actions button[type="submit"]:hover {
  background: #3182ce;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(66, 153, 225, 0.3);
}

/* Responsive */
@media (max-width: 1024px) {
  .task-board {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .task-board {
    grid-template-columns: 1fr;
  }
  
  .sidebar {
    width: 100%;
    height: auto;
    position: relative;
    padding: 16px;
  }
  
  .main-content {
    margin-left: 0;
    padding: 20px;
  }
  
  .modal {
    padding: 24px;
    width: 95%;
  }
}
</style>
