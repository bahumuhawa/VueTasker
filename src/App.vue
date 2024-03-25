<script setup>
import { onMounted, ref } from 'vue'

// Define model for task list form
const tasksModel = ref({
  tasks: [], // holds the list of tasks
  editing: null, // keeps track of the task being edited
})

// Function to create a new task and store it in localStorage
const createTask = (data, form$) => {
  addToStorage(form$.data) // adds the task to localStorage
  syncFromStorage() // updates the `tasksModel` based on localStorage

  form$.reset() // resets the form after adding the task
}

// Function to add a new task to localStorage
const addToStorage = (data) => {
  let storageData = localStorage.getItem('tasks')
  storageData = storageData ? JSON.parse(storageData) : []

  storageData.push(data)
  localStorage.setItem('tasks', JSON.stringify(storageData))
}

// Function to synchronize `tasksModel` with localStorage
const syncFromStorage = () => {
  let tasks = localStorage.getItem('tasks')

  tasksModel.value = {
    tasks: tasks ? JSON.parse(tasks) : []
  }
}

// Function to synchronize `tasksModel.tasks` with localStorage
const syncToStorage = () => {
  localStorage.setItem('tasks', JSON.stringify(tasksModel.value.tasks))
}

// Function to set the task to be edited
const edit = (index) => {
  tasksModel.value.editing = index
}

// Function to cancel editing of a task
const cancel = (index) => {
  tasksModel.value.editing = null
  syncFromStorage()
}

// Function to save the edited task
const save = () => {
  syncToStorage()
  tasksModel.value.editing = null
}

// Synchronize `tasksModel` with localStorage on page load
onMounted(() => {
  syncFromStorage()
})
</script>


<template>
  <div class="page">
    <div class="container">
      <h1>VueTasker</h1>

      <!-- Add a task -->
      <Vueform size="lg" :endpoint="createTask">
        <!-- Task input -->
        <TextElement name="task" placeholder="Add a task" floating="Task name" rules="required" />

        <!-- Task type -->
        <RadiogroupElement name="type" :items="['Personal', 'Business']" view="tabs" default="Personal" />

        <!-- Submit button -->
        <ButtonElement name="button" align="right" submits>
          Submit
        </ButtonElement>
      </Vueform>

      <hr class="divider" />

      <!-- Task list -->
      <Vueform v-model="tasksModel" sync>

        <!-- List of tasks -->
        <ListElement name="tasks" :controls="{ add: false }" :add-class="{ handle: 'task-sort-handle' }" sort
          @sort="syncToStorage" @remove="syncToStorage">
          <template #default="{ index }">

            <!-- Task wrapper -->
            <ObjectElement :name="index"
              :add-class="['task-container', tasksModel.tasks[index].type === 'Personal' ? 'is-personal' : 'is-business']">

              <!-- Edit button -->
              <ButtonElement :label="`#${index + 1} - ${tasksModel.tasks[index].task}`" name="edit" align="right"
                :conditions="[['editing', '!=', index]]" :columns="{ label: 8 }" @click="edit(index)">
                Edit
              </ButtonElement>

              <!-- Task input when editing -->
              <TextElement name="task" :conditions="[['editing', index]]" :columns="6" />

              <!-- Task type when editing -->
              <RadiogroupElement name="type" view="tabs" :conditions="[['editing', index]]" :columns="2"
                :items="{ 'Personal': 'P', 'Business': 'B' }" />

              <!-- Cancel editing -->
              <ButtonElement name="cancel" :conditions="[['editing', index]]" :columns="2" danger full @click="cancel">
                Cancel
              </ButtonElement>

              <!-- Save the edited task -->
              <ButtonElement name="save" :conditions="[['editing', index]]" :columns="2" full @click="save">
                Save
              </ButtonElement>

            </ObjectElement>
          </template>
        </ListElement>

        <!-- Hidden element to store which task is being edited -->
        <HiddenElement name="editing" />
      </Vueform>
    </div>
  </div>
</template>

<style lang="scss">
.page {
  background: #f5f5f5;
  width: 100%;
  min-height: 100vh;
  padding-top: 2rem;
}

.container {
  max-width: 600px;
  margin: 0 auto;
}

h1 {
  font-size: 48px;
  margin-bottom: 2rem;
  font-weight: 600;
  color: #07bf9b;
}

.divider {
  margin: 2rem 0;
  border-color: #cccccc;
}

.task-container {
  background: #ffffff;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

  &.is-personal {
    border-left: 3px solid #ff6600;
  }

  &.is-business {
    border-left: 3px solid #0077cc;
  }
}

.task-wrapper {
  display: flex;
  align-items: center;
}

.vf-list-handle.task-sort-handle {
  top: 1rem;
}

/* Button styles */
button {
  background-color: #ff6600;
  color: white;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #e65c00;
}

/* Form input styles */
input[type="text"] {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  transition: border-color 0.3s ease;
}

input[type="text"]:focus {
  border-color: #ff6600;
}
</style>
