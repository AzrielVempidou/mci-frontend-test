<template>
  <div class="fluid">
    <label>Task</label>
    <InputText v-model="taskDescription"></InputText>
    <div class="fluid grid">
      <label> Duration </label>
      <InputText v-model="durationTask"></InputText>
      <Dropdown
        v-model="selectedDescDuration"
        :options="descriptionDuration"
        optionLabel="name"
      ></Dropdown>
      <Button @click="createTask()" label="Add New Task"></Button>
    </div>
  </div>
  <div class="fluid">
    <Checkbox v-model="active">
    </Checkbox>
    <label>
      Active
    </label>
  </div>
  <div>
    <DataTable :value="taskList" tableStyle="min-width: 50rem">
      <Column field="id" header="ID"></Column>
      <Column field="task" header="Task Description">
        <template #body="slotProps">
          <div v-if="editable">
            <InputText
              v-model="slotProps.data.task"
              :value="slotProps.data.task"
            ></InputText>
          </div>
          <div v-else>
            <span>
              {{ slotProps.data.task }}
            </span>
          </div></template
        >
      </Column>
      <Column field="duration" header="Duration"> </Column>
      <Column field="deletedAt" header="Deleted At"></Column>
      <Column header="Action">
        <template #body="slotProps">
          <div v-if="!editable">
            <Button label="Edit" @click="editText(slotProps.data.id)"></Button>
            <Button
              label="Soft Delete"
              @click="deleteTask(slotProps.data.id)"
            ></Button>
            <Button
              label="Hard Delete"
              @click="deleteTask(slotProps.data.id)"
            ></Button>
          </div>
          <div v-else>
            <Button label="Update" @click="updateTask(slotProps.data)"></Button>
            <Button label="Cancel" @click="cancelEdit()"></Button>
          </div>
        </template>
      </Column>
      <template #empty> No data found. </template>
    </DataTable>
  </div>
</template>

<script setup>
const taskList = ref();
const taskDescription = ref();
const durationTask = ref();
const editable = ref(false);
const taskDescriptionList = ref();
const descriptionDuration = ref([
  { name: "Seconds", code: "s" },
  { name: "Minute", code: "m" },
  { name: "Hour", code: "h" },
  { name: "Day", code: "d" },
  { name: "Week", code: "w" },
  { name: "Month", code: "m" },
  { name: "Year", code: "y" },
]);
const selectedDescDuration = ref();
onMounted(() => {
  loadData();
});
async function loadData() {
  try {
    const response = await $fetch(`/api/todos`);
    taskList.value = response;
  } catch (error) {
    console.error("Cannot Access Server");
  }
}
async function createTask() {
  // TODO : Implement Create Task to Backend
  try {
    const response = await $fetch(`/api/todos`, {
      method: 'POST',
      body: {
        task: taskDescription.value,
        duration : durationTask.value,
      }
    })
    taskList.value.push(response);
    taskDescription.value = '';
    durationTask.value = '';
  } catch (error) {
    console.log(error)
  }
}
async function editText() {
  editable.value = true;
}
async function cancelEdit() {
  editable.value = false;
}
async function updateTask(data) {
  // TODO : Implement Update Task to Backend
  try {
    const response = await $fetch(`/api/todos/${data.id}`, {
      method: 'PUT',
      body: data
    })
    const index = taskList.value.findIndex(task => task.id === data.id);
    if (index !== -1) {
      taskList.value[index] = response;
    }
    editable.value = false;
    router.push('/')
  } catch (error) {
    console.error('Failed to update task', error);
  }
}

async function softDeleteTask(id) {
  try {
    const response = await $fetch(`/api/todos/${id}`, {
      method: 'PATCH',
    });
    const index = taskList.value.findIndex(task => task.id === id);
    if (index !== -1) {
      taskList.value[index].deletedAt = response.deletedAt;
    }
  } catch (error) {
    console.error('Failed to soft delete task', error);
  }
}

async function deleteTask(id) {
  // TODO : Implement Condition Hard Delete
  try {
    const response = await $fetch(`/api/todos/${id}`, {
      method: "DELETE",
      body: [],
    });
    return response;
  } catch (error) {
    console.error(error);
  }
}
</script>

<style></style>
