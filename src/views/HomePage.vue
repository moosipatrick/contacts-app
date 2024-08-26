<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Kontakte</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="showAddContactModal">
            <ion-icon :icon="add" size="large"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-list>
        <ion-item v-for="contact in contacts" :key="contact.contactId" @click="pushContactDetail(contact)">
          <ion-label>
            <h2>{{ contact.name!["display"] }}</h2>
          </ion-label>
        </ion-item>
      </ion-list>
      <!-- Kontaktformular -->
      <ion-modal :is-open="isAddContactModalOpen" @will-dismiss="closeAddContactModal">
        <ion-header>
          <ion-toolbar>
            <ion-buttons slot="start">
              <ion-button @click="closeAddContactModal">Abbrechen</ion-button>
            </ion-buttons>
            <ion-title>Neuer Kontakt</ion-title>
            <ion-buttons slot="end">
              <ion-button @click="createNewContact">Speichern</ion-button>
            </ion-buttons>
          </ion-toolbar>
        </ion-header>
        <ion-content>
          <ion-item>
            <ion-label class="floating_label">Vorname</ion-label>
          </ion-item>
          <ion-item>
            <ion-input v-model="newContact.given"></ion-input>
          </ion-item>
          <ion-item>
            <ion-label class="floating_label">Nachname</ion-label>
          </ion-item>
          <ion-item>
            <ion-input v-model="newContact.family"></ion-input>
          </ion-item>
          <ion-item>
            <ion-label class="floating_label">Telefonnummer</ion-label>
          </ion-item>
          <ion-item>
            <ion-input type="tel" v-model="newContact.phoneNumber"></ion-input>
          </ion-item>
          <ion-item>
            <ion-label class="floating_label">Email</ion-label>
          </ion-item>
          <ion-item>
            <ion-input type="email" v-model="newContact.emailAddress"></ion-input>
          </ion-item>
          <ion-item>
            <ion-label class="floating_label">Geburtsdatum</ion-label>
          </ion-item>
          <ion-item>
            <ion-input type="date" v-model="newContact.birthday"></ion-input>
          </ion-item>
        </ion-content>
      </ion-modal>      
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
} from "@ionic/vue";
import { defineComponent } from "vue";
import { Contacts, PhoneType, EmailType, ContactPayload} from "@capacitor-community/contacts";
import { add } from 'ionicons/icons';
import { useRouter } from "vue-router";

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
  },

  setup() {
    const router = useRouter();
    return { add, router }
  },

  data() {
    return {
      contacts: [] as ContactPayload[],
      isAddContactModalOpen: false,
      newContact: {
        given: '',
        family: '',
        phoneNumber: '',
        emailAddress: '',
        birthday: '',
      },
    };
  },
  async created() {
    await this.retrieveListOfContacts();
  },
  watch:{
    // Watch for route changes / Es wird auf diese Art gelöst, da sich bei jedem löschen die IDs der übrigen KOntakte ändern kann????
    '$route': 'retrieveListOfContacts'
  },

  methods: {
    async retrieveListOfContacts () {
      try {
        const result = await Contacts.getContacts({
          projection: {
            // Specify which fields should be retrieved.
            name: true,
            phones: true,
            emails: true,
            birthday: true,
          },
        });
        this.contacts = result.contacts;
        this.sortContactsByFirstName();
      } catch (error) {
        console.error("Error loading contacts", error);
      }
    },

    sortContactsByFirstName() {
      // Function to sort contacts by given name
      this.contacts.sort((a, b) => {
        console.log(a);
        if (a.name!['given']?.toLowerCase()! < b.name!['given']?.toLowerCase()!) {
          return -1;
        } else if (a.name!['given']?.toLowerCase()! > b.name!['given']?.toLowerCase()!) {
          return 1;
        } else {
          return 0;
        }
      });
    },

    showAddContactModal() {
      this.isAddContactModalOpen = true;
    },
    closeAddContactModal() {
      this.isAddContactModalOpen = false;
      this.newContact.phoneNumber = '';
      this.newContact.birthday = '';
      this.newContact.emailAddress = '';
      this.newContact.given = '';
      this.newContact.family = '';
    },

    pushContactDetail(contact: ContactPayload){ //Eigentlich wird nur contactId benötigt / routing geht mit diesem Ansatz nur mit Strings
      this.router.push({name: 'ContactDetail', params: {contactId: contact.contactId} });
    },

    async createNewContact () {
      const res = await Contacts.createContact({
        contact: {
          name: {
            given: this.newContact.given,
            family: this.newContact.family,
          },
          birthday: {
            year: parseInt(this.newContact.birthday.slice(0,4)),
            month: parseInt(this.newContact.birthday.slice(5,7)),
            day: parseInt(this.newContact.birthday.slice(8,10)),
          },
          phones: [
            {
              type: PhoneType.Mobile,
              label: 'mobile',
              number: this.newContact.phoneNumber,
            },
          ],
          emails: [
            {
              type: EmailType.Home,
              label: 'private',
              address: this.newContact.emailAddress,
            },
          ],
        },
      });
      await this.retrieveListOfContacts ();
      this.isAddContactModalOpen = false;
      this.newContact.phoneNumber = '';
      this.newContact.birthday = '';
      this.newContact.emailAddress = '';
      this.newContact.given = '';
      this.newContact.family = '';
    },
  },
});
</script>

<style scoped>
ion-title {
  color: white;
  font-style: normal;
  font-size: x-large;
  font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
ion-modal ion-title {
  font-size: large;
  text-align: center;
}
ion-modal ion-button {
  color: white;
  font-size: small;
  font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
ion-toolbar {
  --background: #3880ff;
}
ion-icon {
  color: white;
}
ion-label h2 {
  color: black;
  font-style: normal;
  font-size: large;
  font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

ion-modal ion-item {
  --border-width: 0px; /* Entfernt die Border-Breite */
  --border-color: transparent; /* Setzt die Border-Farbe auf transparent */
  --border-style: none; /* Setzt den Border-Stil auf 'none' */
}
ion-input {
  border-width: 2px; /* Breite der Border */
  border-style: solid; /* Stil der Border (z.B. solid, dashed) */
  border-color: #a9aeb8; /* Farbe der Border */
  border-radius: 5px; /* Optionale Abrundung der Ecken */
}
.floating_label {
  height: 100%;
  display: flex;
  align-items: end;
}
</style>
