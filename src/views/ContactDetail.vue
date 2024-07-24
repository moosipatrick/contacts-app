<template>
    <ion-page>
      <ion-header>
        <ion-toolbar>
          <ion-buttons slot="start">
            <ion-back-button></ion-back-button>
          </ion-buttons>
          <ion-title>Detailansicht</ion-title>
        </ion-toolbar>
      </ion-header>
  
      <ion-content>
        <ion-list>
          <ion-item>
            <ion-label position="stacked">Vorname</ion-label>
            <ion-input v-model="contactId" readonly></ion-input>
          </ion-item>
          <ion-item>
            <ion-label position="stacked">Nachname</ion-label>
            <!-- <ion-input v-model="contact.lastName" readonly></ion-input> -->
          </ion-item>
          <ion-item>
            <ion-label position="stacked">Telefonnummer</ion-label>
            <!-- <ion-input v-model="contact.phoneNumber" readonly></ion-input> -->
          </ion-item>
          <ion-item>
            <ion-label position="stacked">E-Mail-Adresse</ion-label>
            <!-- <ion-input v-model="contact.emailAddress" readonly></ion-input> -->
          </ion-item>
          <ion-item>
            <ion-label position="stacked">Geburtstag</ion-label>
            <!-- <ion-input v-model="contact.birthday" readonly></ion-input> -->
          </ion-item>
        </ion-list>
      </ion-content>
    </ion-page>
  </template>
  
  <script lang="ts">
  import {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonList,
    IonItem,
    IonLabel,
    IonIcon,
    IonModal,
    IonInput,
    IonBackButton
  } from "@ionic/vue";
  import { defineComponent, onMounted } from "vue";
  import { useRoute } from 'vue-router';
  
  export default defineComponent({
    components: {
      IonContent,
      IonHeader,
      IonPage,
      IonTitle,
      IonToolbar,
      IonList,
      IonItem,
      IonLabel,
      IonIcon,
      IonModal,
      IonInput,
      IonBackButton
    },
  
    setup() {
        const route = useRoute();
        const contactId = route.params.contactId as string;
        return {
            contactId,
        };
    },
  
    data() {
     
    },
    async created() {
      
    },
    methods: {

    },
  });
  </script>
  
  <style scoped>

  </style>