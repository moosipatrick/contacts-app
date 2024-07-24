<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Kontakte</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="showAddContactModal">
            <ion-icon :icon="add" size="large" color="primary"></ion-icon>
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-list>
        <ion-item v-for="contact in contacts" :key="contact.contactId" @click="pushContactDetail(contact)">
            <ion-label color="primary">
              <h2>{{ contact.name!["display"] }}</h2>
              <p>{{contact.phones?.[0]?.number}}</p>
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
              <ion-input
                placeholder="Vorname"
                v-model="newContact.given"
                @ionInput="onInput"
              ></ion-input>
            </ion-item>
            <ion-item>
              <ion-input
                placeholder="Nachname"
                v-model="newContact.family"
                @ionInput="onInput"
              ></ion-input>
            </ion-item>
            <ion-item>
              <ion-input
                type="tel"
                placeholder="Telefonnummer"
                v-model="newContact.phoneNumber"
                @ionInput="onInput"
              ></ion-input>
            </ion-item>
            <ion-item>
              <ion-input
                type="email"
                placeholder="E-Mail-Adresse"
                v-model="newContact.emailAddress"
                @ionInput="onInput"
              ></ion-input>
            </ion-item>
            <ion-item>
              <ion-label>Geburtstag</ion-label>
              <ion-input
                type="date"
                placeholder="Geburtstag"
                v-model="newContact.birthday"
                @ionInput="onInput"
              ></ion-input>
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
    },

   
    onInput(event: CustomEvent){
      console.log(event.detail.value);
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
            year: parseInt(this.newContact.birthday.slice(6)),
            month: parseInt(this.newContact.birthday.slice(3,4)),
            day: parseInt(this.newContact.birthday.slice(0,1)),
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
/* Optional styles for better visibility */
ion-item {
  margin-bottom: 10px;
}

ion-label {
  color: #000; /* Set label color for better readability */
}

ion-input {
  --padding-start: 0; /* Adjust padding if necessary */
  --padding-end: 0; /* Adjust padding if necessary */
}
</style>