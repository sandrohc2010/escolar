<template>
  <v-container>
    <div>
      <v-toolbar flat color="white">
        <v-toolbar-title>Gerenciar Professores</v-toolbar-title>
        <v-divider class="mx-2" inset vertical></v-divider>
        <v-spacer></v-spacer>
      </v-toolbar>
    </div>
    <v-data-table :headers="headers" :items="professores" sort-by="calories" class="elevation-1">
      <template v-slot:top>
        <v-toolbar flat color="white">
          <v-toolbar-title>My CRUD</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on }">
              <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container grid-list-md>
                  <v-layout wrap>
                    <v-flex xs12>
                      <v-text-field v-model="professor.nome" label="Nome"></v-text-field>
                    </v-flex>
                    <v-flex xs12 lg6>
                      <v-menu
                        ref="menu1"
                        v-model="menu1"
                        :close-on-content-click="false"
                        transition="scale-transition"
                        offset-y
                        full-width
                        max-width="290px"
                        min-width="290px"
                      >
                        <template v-slot:activator="{ on }">
                          <v-text-field
                            v-model="dateFormatted"
                            label="Date"
                            hint="MM/DD/YYYY format"
                            persistent-hint
                            prepend-icon="mdi-calendar"
                            @blur="professor.nascimento = parseDate(dateFormatted)"
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-date-picker
                          v-model="professor.nascimento"
                          no-title
                          @input="menu1 = true"
                        ></v-date-picker>
                      </v-menu>
                      <p>
                        Date in ISO format:
                        <strong>{{ professor.nascimento }}</strong>
                      </p>
                    </v-flex>
                  </v-layout>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
                <v-btn color="blue darken-1" text @click="save">Save</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template v-slot:item.data_nascimento="{ item }">
        {{formatDate(item.data_nascimento)}}
      </template>
      <template v-slot:item.data_criacao="{ item }">
        {{formatDate(item.data_criacao)}}
      </template>
      <template v-slot:item.action="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)">mdi-pencil</v-icon>
        <v-icon small @click="deleteItem(item)">mdi-delete</v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize">Reset</v-btn>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
import axios from "axios";
import { url } from "../config";

//axios.defaults.headers.common['Access-Control-Allow-Origin'] = '*';

export default {
  data() {
    return {
      professores: [],
      modal: false,
      dialog: false,
      dateFormatted: this.formatDate(new Date().toISOString().substr(0, 10)),
      menu1: false,
      menu: false,
      headers: [
        {
          text: "Código",
          align: "left",
          sortable: false,
          value: "id_professor"
        },
        { text: "Nome", value: "nome" },
        { text: "Data Nascimento", value: "data_nascimento" },
        { text: "Data Criacao", value: "data_criacao" },
        { text: "Actions", value: "action", sortable: false }
      ],
      desserts: [],
      editedIndex: -1,
      editedItem: {
        name: "",
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0
      },
      defaultItem: {
        name: "",
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0
      },
      professor: {
        nome: "",
        nascimento: "",
      },
      selectedID: -1,
    };
  },

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
    computedDateFormatted() {
      return this.formatDate(this.date);
    },
  },

  async beforeMount() {
    await this.initialize();
  },

  methods: {
    formatDate(date) {
      if (!date) return null;

      const [year, month, day] = date.split("-");
      return `${day}/${month}/${year}`;
    },
    parseDate(date) {
      if (!date) return null;

      const [day, month, year] = date.split("/");
      console.log(date);
      return `${year}-${month.padStart(2, "0")}-${day.padStart(2, "0")}`;
    },
    initialize() {
      axios
        .get(`${url}professores`)
        .then((result) => {
          this.professores = result.data;
          console.log(result.data);
        })
        .catch((e) => {
          console.log(e);
        });
    },

    editItem(item) {
      this.editedIndex = this.professores.indexOf(item);
      this.professor.nome = item.nome;
      this.professor.nascimento = item.data_nascimento;
      this.selectedID = item.id_professor;
      this.dateFormatted = this.formatDate(item.data_nascimento);
      this.dialog = true;
    },

    deleteItem(item) {
      axios.get(`${url}professores/${item.id_professor}`)
        .then((result) => {
          console.log(result.data);
          this.initialize();
        })
        .catch((e) => {
          console.log(e);
        });
    },

    close() {
      this.dialog = false;
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      }, 300);
    },

    save() {
      const formData = new FormData();
      // eslint-disable-next-line
      for (const item in this.professor) {
        formData.append(item, this.professor[item]);
      }
      console.log(this.professor);
      if (this.editedIndex > -1) {
        axios.post(`${url}professores/${this.selectedID}`, formData, {
          headers: {
            'Content-Type': 'application/json',
          },
        })
          .then((result) => {
            console.log(result.data);
            this.initialize();
          })
          .catch((e) => {
            console.log(e);
          });
      } else {
        axios
          .post(`${url}professores`, formData, {
            headers: {
              'Content-Type': 'application/json',
            },
          })
          .then((result) => {
            console.log(result);
            this.initialize();
          })
          .catch((e) => {
            console.log(e);
          });
      }
      this.close();
    },
  },
};
</script>
