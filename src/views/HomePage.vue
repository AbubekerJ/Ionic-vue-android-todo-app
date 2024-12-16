<template>
  <ion-page>
     
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title class="ion-justify-content-between">
          <div class="flex items-center gap-2">
            <img src="/et-logo.png" alt="Logo" style="height: 40px" />
            <h1>Ethiotelcom</h1>
          </div>
          <router-link to="/login">Login </router-link>
        </ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Ethiotelcom</ion-title>
       
        </ion-toolbar>
      </ion-header>


      <ion-refresher slot="fixed" @ionRefresh="handleRefresh($event)">
      <ion-refresher-content></ion-refresher-content>
    </ion-refresher>
    
      <ion-card class="ion-margin-top">
        <ion-card-header>
          <ion-card-title>Task Manager</ion-card-title>
          <ion-card-subtitle
            >Organize, Prioritize, and Achieve Goals</ion-card-subtitle
          >
        </ion-card-header>

        <ion-card-content>
          Take control of your tasks with ease. Create new tasks, update
          existing ones, or delete those you’ve completed.
        </ion-card-content>
      </ion-card>

      <div class="p-4">
        <form @submit.prevent="handleSubmit">
          <ion-item>
            <ion-input
              label="Task"
              placeholder="Add task"
              v-model="newTask"
            ></ion-input>
          </ion-item>
          <ion-button expand="block" type="submit" color="success"
            >Add Task</ion-button
          >
        </form>
        <ion-list>
          <ion-card>
            <ion-card-content>
              <ion-item v-for="(t, index) in task" :key="index">
                <ion-label>{{ t.title }}</ion-label>

                <ion-button
                  slot="end"
                  id="open-loading-delete"
                  color="danger"
                  @click="handleDelete(t)"
                  >Delete</ion-button
                >

              
                <ion-button
                  slot="end"
                  color="primary"
                  @click="handleEdit(t.title, index)"

                  >Edit</ion-button
                >
              </ion-item>
            </ion-card-content>
          </ion-card>
        </ion-list>

        <ion-modal
          :is-open="isModalOpen"
          @ionModalDidDismiss="closeModal"
          class="ion-align-items-center"
        >
          <div class="">
            <h2 class="text-2xl font-bold p-4">Edit Task</h2>
            <ion-item class="ion-padding">
              <ion-input
                label="Edit Task : "
                placeholder="Edit your task"
                v-model="editTask"
              ></ion-input>
            </ion-item>

            <ion-button
              id="open-loading-update"
              expand="block"
              color="success"
              class="ion-padding-horizontal"
              @click="handleUpdate"
              >Update</ion-button
            >
          
            <ion-button
              expand="block"
              color="medium"
              class="ion-padding-horizontal"
              @click="closeModal"
              >Cancel</ion-button
            >
          </div>
        </ion-modal>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonInput,
  IonItem,
  IonModal,
  IonLabel,
  IonButton,
  IonList,
  loadingController,
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardSubtitle,
  IonCardTitle,
  IonRefresher,
   IonRefresherContent
  
} from "@ionic/vue";
import { ref,onMounted } from "vue";


import axios from "axios";


interface Task {
  id: number;
  title: string;
}


const task = ref<Task[]>([]);
const newTask = ref<string>("");
const editTask = ref<string>("");
const editingIndex= ref<any> (null);
const isModalOpen = ref<boolean>(false);

const fetchDate = async()=>{
    const response = await axios.get(
    "https://675c2955fe09df667f62db95.mockapi.io/todo/todos"
  );
  task.value = response.data;
  }
onMounted(async() => {
  await fetchDate()
});

const handleSubmit = async () => {
  if (newTask.value.trim() !== "") {
    try {
      const response = await axios.post(
        "https://675c2955fe09df667f62db95.mockapi.io/todo/todos",
        { title: newTask.value }
      );

      task.value.push(response.data);
      newTask.value = "";
    } catch (error) {
      console.error("Error adding task:", error);
    }
  }
};

const handleEdit = (taskToEdit: string, index: number | null) => {
  editTask.value = taskToEdit;
  editingIndex.value = index;
  isModalOpen.value = true;
};
const closeModal = () => {
  isModalOpen.value = false;
};

const handleUpdate = async () => {
  if (editTask.value.trim() !== "" && editingIndex.value !== null) {

    const loading = await loadingController.create({
    message: 'Updating Task...',
  });
  await loading.present();
    try {
      const updatedTask = {
        ...task.value[editingIndex.value],
        title: editTask.value,
      };
      await axios.put(
        `https://675c2955fe09df667f62db95.mockapi.io/todo/todos/${updatedTask.id}`,
        updatedTask
      );
      task.value[editingIndex?.value] = updatedTask;
      closeModal();
    } catch (error) {
      console.error("Error updating task:", error);
    }finally {
    loading.dismiss();
  }
  }
};

const handleDelete = async (t: Task) => {
  const loading = await loadingController.create({
    message: 'Deleting Task...',
  });
  await loading.present();

  try {
    await axios.delete(
      `https://675c2955fe09df667f62db95.mockapi.io/todo/todos/${t.id}`
    );
    task.value = task.value.filter((taskItem: Task) => taskItem.id !== t.id);
  } catch (error) {
    console.error("Error deleting task:", error);
  } finally {
    loading.dismiss();
  }
};


const handleRefresh =async (event: CustomEvent)=>{
  try {
    await fetchDate()
  } catch (error) {
    console.log(error)
  }finally{
    (event.target as HTMLIonRefresherElement).complete()
  }
}

</script>
