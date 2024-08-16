<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-button @click="goBackToOverview">
            <ion-icon :icon="arrowBackOutline" slot="start" color="light"></ion-icon>
          </ion-button>
        </ion-buttons>
        <ion-title>Kontakt Details</ion-title>
        <ion-buttons slot="end">
          <ion-button fill="clear" @click="showDeleteAlert(contact.contactId)">
            <ion-icon :icon="trashBinOutline" size="large" color="light"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <ion-card>
        <ion-card-header>
          <ion-card-title>{{ contact.name?.given }} {{ contact.name?.family }}</ion-card-title>
        </ion-card-header>
        
        <ion-card-content>
          <ion-item lines="none">
            <ion-icon :icon="phonePortraitOutline" slot="start"></ion-icon>
            <ion-label>{{ contact.phones?.[0]?.number || 'Keine Telefonnummer' }}</ion-label>
            
            <!-- Icon zum Anrufen der Telefonnummer -->
            <ion-icon 
              :icon="callOutline" 
              slot="end" 
              @click="callNumber(contact.phones?.[0]?.number ?? undefined)" 
              style="margin-right: 10px;">
            </ion-icon>

            <!-- Icon zum Kopieren der Telefonnummer in die Zwischenablage -->
            <ion-icon 
              :icon="copyOutline" 
              slot="end" 
              @click="copyToClipboard(contact.phones?.[0]?.number ?? undefined)">
            </ion-icon>
          </ion-item>

          <ion-item lines="none">
            <ion-icon :icon="mailOutline" slot="start"></ion-icon>
            <ion-label>{{ contact.emails?.[0]?.address || 'Keine E-Mail-Adresse' }}</ion-label>

            <!-- Icon zum Öffnen der Standard-E-Mail-App -->
            <ion-icon 
              :icon="mailOutline" 
              slot="end" 
              @click="sendEmail(contact.emails?.[0]?.address ?? undefined)" 
              style="margin-right: 10px;">
            </ion-icon>

            <!-- Icon zum Kopieren der E-Mail-Adresse in die Zwischenablage -->
            <ion-icon 
              :icon="copyOutline" 
              slot="end" 
              @click="copyToClipboard(contact.emails?.[0]?.address ?? undefined)">
            </ion-icon>
          </ion-item>

          <ion-item lines="none">
            <ion-icon :icon="giftOutline" slot="start"></ion-icon>
            <ion-label>{{ contact.birthday ? `${formatDate(contact.birthday.day as number)}.${formatDate(contact.birthday.month as number)}.${contact.birthday.year}` : 'Kein Geburtsdatum' }}</ion-label>
          </ion-item>
        </ion-card-content>
      </ion-card>

      <ion-toast
        v-if="showErrorToast"
        :is-open="showErrorToast"
        message="Fehler beim Laden des Kontakts"
        duration="2000"
        color="danger"
        @didDismiss="showErrorToast = false"
      ></ion-toast>
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
  IonButtons,
  IonButton,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonCardContent,
  IonItem,
  IonLabel,
  IonIcon,
  IonAlert,
  IonToast,
} from "@ionic/vue";
import { defineComponent } from "vue";
import { useRouter, useRoute } from "vue-router";
import { Contacts, ContactPayload } from "@capacitor-community/contacts";
import { trashBinOutline, phonePortraitOutline, mailOutline, arrowBackOutline, giftOutline, callOutline, copyOutline } from "ionicons/icons";
import { Clipboard } from '@capacitor/clipboard';
import { Toast } from '@capacitor/toast';

export default defineComponent({
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonButtons,
    IonButton,
    IonCard,
    IonCardHeader,
    IonCardTitle,
    IonCardSubtitle,
    IonCardContent,
    IonItem,
    IonLabel,
    IonIcon,
    IonAlert,
    IonToast,
  },
  setup() {
    const router = useRouter();

    // Funktion zum Formatieren von Datumswerten
    const formatDate = (date: number) => {
      var new_date = date.toString();

      if (new_date.length < 2) {
        return "0"+date
      } else {
        return date
      }
    };
    
    return { 
      trashBinOutline, 
      phonePortraitOutline, 
      mailOutline, 
      arrowBackOutline,
      giftOutline, 
      router,
      callOutline,
      copyOutline,
      formatDate 
    };
  },
  data() {
    return {
      contact: {} as ContactPayload,
      isAlertOpen: false,
      alertOptions: null as any,
      showErrorToast: false,
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
        this.showErrorToast = true;
      }
    },
    goBackToOverview() {
      // Navigiert zur Übersichtsseite, z.B. zur Route "Home"
      this.router.push({ name: 'Home' });
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
        this.router.replace({ name: 'Home' });
      } catch (error) {
        console.error("Error deleting contact", error);
        this.showErrorToast = true;
      }
    },

    callNumber(number: string | undefined) {
      if (number) {
        window.open(`tel:${number}`, '_system');
      } else {
        this.showToast('Keine Telefonnummer verfügbar');
      }
    },
    async copyToClipboard(text: string | undefined) {
      if (text) {
        try {
          await Clipboard.write({
            string: text
          });
          this.showToast('Telefonnummer in die Zwischenablage kopiert');
        } catch (error) {
          this.showToast('Fehler beim Kopieren der Telefonnummer');
        }
      } else {
        this.showToast('Keine Telefonnummer verfügbar');
      }
    },
    sendEmail(email: string | undefined) {
      if (email) {
        window.open(`mailto:${email}`, '_system');
      } else {
        this.showToast('Keine E-Mail-Adresse verfügbar');
      }
    },
    async showToast(message: string) {
      await Toast.show({
        text: message,
        duration: 'short', // 'short' or 'long'
        position: 'bottom' // 'bottom', 'top', or 'center'
      });
    }
  }
});
</script>

<style scoped>
ion-title {
  font-size: x-large;
  font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
ion-toolbar {
  --color: white;
  --background: #3880ff;
}

ion-card {
  margin: 0 auto;
  max-width: 600px;
}

ion-card-title {
  font-size: x-large;
  font-weight: bold;
  font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

ion-item {
  --padding-start: 16px;
  --inner-padding-end: 0;
  margin-bottom: 10px;
}

ion-item ion-icon {
  color: #3880ff;
}
</style>
