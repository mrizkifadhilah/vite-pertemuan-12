<template>
  <q-input v-model="text" label="Aktivitas" />
  <div class="q-mt-lg">
    <q-btn :disable="!text" color="white" text-color="black" label="Submit" @click="postDataAsyncAwait" />
  </div>

  <div class="q-mt-lg">
    <q-table title="My To Do List" :rows="filteredTodos">
      <template v-slot:top-right>
        <q-checkbox v-model="SHOW_ALL" label="Tampilkan Semua" />
      </template>
      <template v-slot:body-cell-id="props">
        <q-td :props="props">
          <q-btn v-if="props.row.status === 'done'" round color="negative" icon="delete" @click="deleteData(props.row.id)" />
          <q-btn v-if="props.row.status === 'open'" class="q-ml-sm" round color="blue" icon="check" @click="updateData(props.row.id)" />
        </q-td>
      </template>
    </q-table>
  </div>
</template>

<script setup>
import { onMounted, ref, computed } from "vue";
import { useQuasar } from "quasar";

const $q = useQuasar();
const text = ref("");
const todos = ref([]);
const SHOW_ALL = ref(false);

const postData = () => {
  fetch("http://localhost:4444/todos", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      activity: text.value,
      status: "open",
    }),
  }).then(() => {
    $q.notify({
      color: "positive",
      textColor: "white",
      icon: "cloud_done",
      message: "Data Berhasil di Submit",
      position: "center",
    });
    text.value = "";
    getData();
  });
};

const postDataAsyncAwait = async () => {
  const result = await fetch("http://localhost:4444/todos", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      activity: text.value,
      status: "open",
    }),
  })

  if (result.status === 201) {
    $q.notify({
      color: "positive",
      textColor: "white",
      icon: "cloud_done",
      message: "Data Berhasil di Submit",
      position: "center",
    });
    text.value = "";
    getData();
  }
};

const filteredTodos = computed(() => {
  if (SHOW_ALL.value) {
    return todos.value;
  } else {
    return todos.value.filter((todo) => todo.status === "open");
  } 
})

const getData = async () => {
  const response = await fetch("http://localhost:4444/todos");

  todos.value = await response.json();
};

const deleteData = (id) => {
  fetch(`http://localhost:4444/todos/${id}`, {
    method: "DELETE",
    headers: {
      "Content-Type": "application/json",
    },
  }).then(() => {
    getData();
  });
};

const updateData = (id) => {
  fetch(`http://localhost:4444/todos/${id}`, {
    method: "PATCH",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      status: "done",
    }),
  }).then(() => {
    getData();
    $q.notify({
      color: "positive",
      textColor: "white",
      icon: "cloud_done",
      message: "Aktivitas Telah Selesai",
      position: "center",
    });
  });
};

onMounted(() => {
  getData();
});
</script>
