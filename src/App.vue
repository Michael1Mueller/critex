<template>
  <v-app>
    <v-main>
      <v-container fluid>
        <v-row>
          <v-col cols="6">
            <v-responsive class="mx-auto" max-width="500">
              <v-form ref="form">
                <v-text-field
                  v-model="form.firstName"
                  clearable
                  label="Vorname"
                ></v-text-field>
                <v-text-field
                  v-model="form.lastName"
                  clearable
                  label="Nachname"
                ></v-text-field>
                Geburtsdatum
                <v-container>
                  <v-row justify="space-around">
                    <v-date-picker
                      v-model="form.birthDate"
                      color="primary"
                    ></v-date-picker>
                  </v-row>
                </v-container>
                <v-combobox
                  v-model="form.hobbies"
                  clearable
                  label="Hobbies"
                  multiple
                  chips
                  placeholder="Add hobbies"
                ></v-combobox>
                <v-btn
                  class="mt-4"
                  color="primary"
                  @click="submitForm"
                >
                  Hinzufügen
                </v-btn>
              </v-form>
            </v-responsive>
          </v-col>

          <v-col cols="6">
            <v-responsive class="mx-auto" max-width="500">

              <v-row>
                <v-col cols="8">
                  <v-text-field
                    v-model="searchId"
                    label="ID"
                    prepend-icon="mdi-magnify"
                  ></v-text-field>
                </v-col>
                <v-col cols="4">
                  <v-btn
                    class="mt-4"
                    color="primary"
                    @click="searchPerson"
                  >
                    Suchen
                  </v-btn>
                </v-col>
              </v-row>

              <v-row v-if="searchedPerson">
                <v-col cols="12">
                  <v-card>
                    <v-card-title>
                      ID: {{ searchedPerson.personID }}
                    </v-card-title>
                    <v-card-text>
                      <p><strong>Vorname:</strong> {{ searchedPerson.firstName }}</p>
                      <p><strong>Nachname:</strong> {{ searchedPerson.lastName }}</p>
                      <p><strong>Geburtsdatum:</strong> {{ searchedPerson.birthdate }}</p>
                      <p><strong>Hobbies:</strong> {{ searchedPerson.hobbies.join(', ') }}</p>
                    </v-card-text>
                  </v-card>
                </v-col>
              </v-row>

              <v-row v-if="searchedPerson">
                <v-col cols="12">
                  <v-text-field
                    v-model="updateform.firstName"
                    label="Neuer Vorname"
                    clearable
                  ></v-text-field>
                  <v-text-field
                    v-model="updateform.lastName"
                    label="Neuer Nachname"
                    clearable
                  ></v-text-field>
                  Geburtsdatum
                  <v-container>
                    <v-row justify="space-around">
                      <v-date-picker
                        v-model="updateform.birthDate"
                        color="primary"
                      ></v-date-picker>
                    </v-row>
                  </v-container>
                  <v-combobox
                    v-model="updateform.hobbies"
                    clearable
                    label="Neue Hobbies"
                    multiple
                    chips
                    placeholder="Hobbys hinzufügen"
                  ></v-combobox>
                  <v-btn
                    class="mt-4"
                    color="primary"
                    @click="updatePerson"
                  >
                    Aktualisieren
                  </v-btn>
                </v-col>
                
              </v-row>

              <v-row>
                <v-col cols="8">
                  <v-text-field
                    v-model="deleteId"
                    label="ID"
                    prepend-icon="mdi-delete"
                  ></v-text-field>
                </v-col>
                <v-col cols="4">
                  <v-btn
                    class="mt-4"
                    color="primary"
                    @click="deletePerson"
                  >
                    Löschen
                  </v-btn>
                </v-col>
              </v-row>

            </v-responsive>
          </v-col>
        </v-row>
        
        <v-data-table
          :headers="headers"
          :items="persons"
          item-key="personID"
          class="mt-5"
          dense
        >
        </v-data-table>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>

import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      updateform: {
        firstName: "",
        lastName: "",
        birthDate: null,
        hobbies: []
      },
      form: {
        firstName: "",
        lastName: "",
        birthDate: null,
        hobbies: []
      },
      deleteId: "",
      searchId: "",
      searchedPerson: null,
      persons: [],
      headers: [
        { text: "ID", value: "personID" },
        { text: "Vorname", value: "firstName" },
        { text: "Nachname", value: "lastName" },
        { text: "Geburtsdatum", value: "birthdate" },
        { text: "Hobbies", value: "hobbies" }
      ]
    };
  },
  methods: {
    async searchPerson(){
      const personID = this.searchId;
      if (!personID) {
        alert("Bitte geben Sie eine ID ein.");
        return;
      }
      try{
        const response = await axios.get(`http://localhost:8086/person/${personID}`);
        this.searchedPerson = response.data;
        this.updateform.firstName = this.searchedPerson.firstName;
        this.updateform.lastName = this.searchedPerson.lastName;
        this.updateform.birthDate = new Date(this.searchedPerson.birthdate);
        this.updateform.hobbies = this.searchedPerson.hobbies;
      } catch(error){
        this.searchedPerson = null;
        alert(`Fehler: ${error.response?.data?.error}`)
      }
    },

    async deletePerson(){
      const personID = this.deleteId;
      if (!personID) {
        alert("Bitte geben Sie eine ID ein.");
        return;
      }
      try{
        await axios.delete(`http://localhost:8086/person/${personID}`);
        this.fetchPersons();
        alert(`Person mit ID ${personID} erfolgreich gelöscht`)
        this.deleteId = "";
      } catch(error){
        alert(`Fehler: ${error.response?.data?.error}`)
      }
    },

    async updatePerson(){
      const personID = this.searchId;
      if (!personID) {
        alert("Bitte geben Sie eine ID ein.");
        return;
      }
      try {
        const payload = {
          firstName: this.updateform.firstName,
          lastName: this.updateform.lastName,
          birthdate: this.updateform.birthDate,
          hobbies: this.updateform.hobbies
        };
        const response = await axios.put(`http://localhost:8086/person/${personID}`, payload);
        console.log("Response:", response.data);
        this.clearUpdateForm();
        this.searchedPerson = null;
        this.searchId = "";
        alert(`Person mit id: ${personID} erfolgreich bearbeitet`)
        this.fetchPersons();
      } catch (error) {
        alert(`Fehler: ${error.response?.data?.error}`)
      }
    },
    
    async submitForm() {
      const errors = [];

      if (!this.form.firstName) {
        errors.push("Vorname ist Pflicht.");
      }

      if (!this.form.lastName) {
        errors.push("Nachname ist Pflicht.");
      }

      if (!this.form.birthDate) {
        errors.push("Geburtsdatum ist Pflicht.");
      }

      if (!this.form.hobbies || this.form.hobbies.length === 0) {
        errors.push("Mindestens ein Hobby ist Pflicht.");
      }

      if (errors.length > 0) {
        alert(`Fehler:\n${errors.join('\n')}`);
        return
      } 
      try {
        const payload = {
          firstName: this.form.firstName,
          lastName: this.form.lastName,
          birthdate: this.form.birthDate,
          hobbies: this.form.hobbies
        };

        const response = await axios.post("http://localhost:8086/person", payload);

        console.log("Response:", response.data);
        this.clearForm();
        this.fetchPersons();
        alert("Person erfolgreich hinzugefügt!");
      } catch (error) {
        console.error("Fehler beim Hinzufügen der Person:", error.response?.data || error.message);
        alert(`Fehler: ${error.response?.data?.error}`);
      }
    },

    clearForm(){
      this.form.firstName = "";
      this.form.lastName = "";
      this.form.birthDate = null;
      this.form.hobbies = [];
    },
    clearUpdateForm() {
      this.updateform.firstName = "";
      this.updateform.lastName = "";
      this.updateform.birthDate = null;
      this.updateform.hobbies = [];
    },

    async fetchPersons() {
      try {
        const response = await axios.get("http://localhost:8086/persons");
        this.persons = response.data;
        console.log(this.persons)
      } catch (error) {
        console.error("Fehler beim Laden der Personen:", error.response?.data || error.message);
        alert("Fehler beim Laden der Personen.");
      }
    }
  },
  created(){
    this.fetchPersons();
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
