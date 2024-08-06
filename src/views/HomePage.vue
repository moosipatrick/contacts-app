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
        <ion-content class="modal-content">
          <div class="form-group">
            <div class="floating-label-wrapper">
              <ion-input
                class="floating-label-input"
                type="text"
                placeholder=" "
                v-model="newContact.given"
              ></ion-input>
              <label class="floating-label">Vorname</label>
            </div>
          </div>
          <div class="form-group">
            <div class="floating-label-wrapper">
              <ion-input
                class="floating-label-input"
                type="text"
                placeholder=" "
                v-model="newContact.family"
              ></ion-input>
              <label class="floating-label">Nachname</label>
            </div>
          </div>
          <div class="form-group">
            <div class="floating-label-wrapper">
              <ion-input
                class="floating-label-input"
                type="tel"
                placeholder=" "
                v-model="newContact.phoneNumber"                
              ></ion-input>
              <label class="floating-label">Telefonnummer</label>
            </div>
          </div>
          <div class="form-group">
            <div class="floating-label-wrapper">
              <ion-input
                class="floating-label-input"
                type="email"
                placeholder=" "
                v-model="newContact.emailAddress"
              ></ion-input>
              <label class="floating-label">E-Mail-Adresse</label>
            </div>
          </div>
          <div class="form-group">
            <div class="floating-label-wrapper">
              <ion-input
                class="floating-label-input"
                type="date"
                placeholder=" "
                v-model="newContact.birthday"
              ></ion-input>
              <label class="floating-label">Geburtstag</label>
            </div>
          </div>
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

ion-icon {
  color: #3880ff;
}

ion-label h2 {
  color: black;
  font-weight: bold;
  font-style: normal;
  font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
/* General styles for the modal content */
.modal-content {
  padding: 20px;
  background: #f9f9f9;
  margin-top: 60px; /* Sicherstellen, dass der Inhalt nicht vom Header überlappt wird */
  overflow-y: auto; /* Ermöglicht das Scrollen, wenn der Inhalt den sichtbaren Bereich überschreitet */
}

/* General styles for form spacing */
.form-group {
  margin-bottom: 20px;
}

/* Floating label wrapper */
.floating-label-wrapper {
  position: relative;
  margin-bottom: 20px;
}

/* Floating label input styling */
.floating-label-input {
  border: 2px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  background: #fff;
  width: 100%;
  box-sizing: border-box;
  font-size: 16px;
  transition: border-color 0.3s, box-shadow 0.3s;
  position: relative;
  z-index: 1; /* Ensure input is above the label */
}

/* Hide the default placeholder text */
.floating-label-input::placeholder {
  color: transparent;
}

/* Floating label styling */
.floating-label {
  position: absolute;
  top: 16px;
  left: 12px;
  font-size: 16px;
  color: #aaa;
  transition: all 0.2s ease-in-out;
  pointer-events: none;
  background: #f9f9f9; /* Same background as input to cover any overlap */
  padding: 0 4px; /* Add padding to ensure the label is not cut off */
  z-index: 0; /* Ensure label is below the input */
}

/* Label animation */
.floating-label-input:focus ~ .floating-label,
.floating-label-input:not(:placeholder-shown) ~ .floating-label {
  top: -12px; /* Adjust the top value to move the label above the input */
  font-size: 12px;
  color: #007bff; /* Change color when focused */
}

/* Add focus border and shadow */
.floating-label-input:focus {
  border-color: #007bff;
  box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
}
</style>
