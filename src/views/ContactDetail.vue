<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button></ion-back-button>
        </ion-buttons>
        <ion-buttons slot="end">
        <ion-button @click="deleteContact(contact.contactId)">
          <ion-icon :icon="trashBinOutline" size="large" color="primary"></ion-icon>
        </ion-button>
        </ion-buttons>
        <ion-title>Kontakt Details</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-list v-if="contact">
        <ion-item>
          <ion-label>Vorname: {{ contact.name!["given"] }}</ion-label>
        </ion-item>
        <ion-item>
          <ion-label>Nachname: {{ contact.name!["family"] }}</ion-label>
        </ion-item>
        <ion-item v-if="contact.phones?.length">
          <ion-label>Telefon: {{ contact.phones[0].number }}</ion-label>
        </ion-item>
        <ion-item v-if="contact.emails?.length">
          <ion-label>E-Mail: {{ contact.emails[0].address }}</ion-label>
        </ion-item>
        <ion-item v-if="contact.birthday">
          <ion-label>Geburtstag: {{ contact.birthday }}</ion-label>
        </ion-item>
      </ion-list>
      <ion-list v-else>
        <ion-item>
          <ion-label>Kontakt nicht gefunden</ion-label>
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
    IonBackButton,
    
  } from "@ionic/vue";
  import { defineComponent, onMounted } from "vue";
  import { useRoute } from 'vue-router';
  import { Contacts, ContactPayload} from "@capacitor-community/contacts";
  import { trashBinOutline } from 'ionicons/icons';
  
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
      IonBackButton,
    
    },
  
    setup() {
        
      return{trashBinOutline}
        
    },
  
    data() {

      return{
        contact: {} as ContactPayload,
      }
    },
    async created() {
      const route = useRoute();
      const contactId = route.params.contactId as string;
      await this.getSingleContact(contactId);
        return {
            contactId,
        };
    },
    methods: {

      async getSingleContact (contactId: string) {
      try {
        const result = await Contacts.getContact({
          contactId: contactId,
          projection: {
            name: true,
            phones: true,
            emails: true,
            birthday: true,
          }
        });
        this.contact = result.contact;
        
      } catch (error) {
        console.error("Error loading contact", error);
      }
    },

    async deleteContact (contactId: string) {
      try {
          await Contacts.deleteContact({
            contactId: contactId,
          });
          //ROUTER
          
      } catch (error) {
        console.error("Error deleting contact", error);
      }
    },

    },
  });
  </script>
  
  <style scoped>

  </style>