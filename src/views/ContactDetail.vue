<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button></ion-back-button>
        </ion-buttons>
        <ion-buttons slot="end">
          <ion-button @click="showDeleteAlert(contact.contactId)">
            <ion-icon :icon="trashBinOutline" size="large" color="primary"></ion-icon>
          </ion-button>
        </ion-buttons>
        <ion-title>Kontakt Details</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-list v-if="contact">
        <ion-item v-if="contact.name?.given">
          <ion-label>Vorname: {{ contact.name.given }}</ion-label>
        </ion-item>
        <ion-item v-if="contact.name?.family">
          <ion-label>Nachname: {{ contact.name.family }}</ion-label>
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
    <ion-alert
      v-if="alertOptions"
      :is-open="isAlertOpen"
      :header="alertOptions.header"
      :message="alertOptions.message"
      :buttons="alertOptions.buttons"
      @didDismiss="handleAlertDismiss"
    ></ion-alert>
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
  IonBackButton,
  IonAlert,
} from "@ionic/vue";
import { defineComponent } from "vue";
import { useRouter, useRoute } from "vue-router";
import { Contacts, ContactPayload } from "@capacitor-community/contacts";
import { trashBinOutline } from "ionicons/icons";

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
    IonBackButton,
  },
  setup() {
    const router = useRouter();
    
    return { trashBinOutline, router };
  },
  data() {
    return {
      contact: {} as ContactPayload,
      isAlertOpen: false,
      alertOptions: null as any,
    };
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
    async getSingleContact(contactId: string) {
      try {
        const result = await Contacts.getContact({
          contactId: contactId,
          projection: {
            name: true,
            phones: true,
            emails: true,
            birthday: true,
          },
        });
        this.contact = result.contact;
      } catch (error) {
        console.error("Error loading contact", error);
      }
    },
    showDeleteAlert(contactId: string) {
      this.alertOptions = {
        header: "Kontakt löschen",
        message: "Möchten Sie diesen Kontakt wirklich löschen?",
        buttons: [
          {
            text: "Abbrechen",
            role: "cancel",
          },
          {
            text: "Löschen",
            handler: () => {
              this.deleteContact(contactId);
            },
          },
        ],
      };
      this.isAlertOpen = true;
    },
    handleAlertDismiss() {
      this.isAlertOpen = false;
    },
    async deleteContact(contactId: string) {
      try {
        await Contacts.deleteContact({
          contactId: contactId,
        });
        this.router.replace({name: 'Home'})
      } catch (error) {
        console.error("Error deleting contact", error);
      }
    },
  },
});
</script>

<style scoped>
</style>
