<template>
  <v-container>
    <div>
      <v-toolbar flat color="white">
        <v-toolbar-title>Gerenciar Cursos</v-toolbar-title>
        <v-divider class="mx-2" inset vertical></v-divider>
        <v-spacer></v-spacer>
      </v-toolbar>
    </div>
    <v-data-table :headers="headers" :items="cursos" sort-by="calories" class="elevation-1">
      <template v-slot:top>
        <v-toolbar flat color="white">
          <v-toolbar-title>My CRUD</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on }">
              <v-btn color="primary" dark class="mb-2" v-on="on">Cadastrar Curso</v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>

              <v-card-text>
                <v-container grid-list-md>
                  <v-layout wrap>
                    <v-flex xs12>
                      <v-text-field v-model="curso.nome" label="Nome"></v-text-field>
                    </v-flex>
                    <v-flex xs12 lg6>
                      <div>
                        <v-select
                        :items="professores"
                        v-model="curso.professor"
                        item-text="nome"
                        item-value="id_professor"
                      ></v-select>
                      </div>
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
                            @blur="curso.criacao = parseDate(dateFormatted)"
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-date-picker
                          v-model="curso.criacao"
                          no-title
                          @input="datePicker"
                        ></v-date-picker>
                      </v-menu>
                      <p>
                        Date in ISO format:
                        <strong>{{ curso.criacao }}</strong>
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
      cursos: [],
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
          value: "id_curso"
        },
        { text: "Curso", value: "nome" },
        { text: "Professor", value: "professor" },
        { text: "Data Criacao", value: "data_criacao" },
        { text: "Actions", value: "action", sortable: false }
      ],
      editedIndex: -1,
      curso: {
        nome: '',
        professor: '',
        criacao: '',
      },
      selectedID: -1,
    };
  },

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Cadastrar" : "Editar";
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
    datePicker() {
      this.menu1 = true;
      this.dateFormatted = this.formatDate(this.curso.criacao);
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

      axios
        .get(`${url}cursos`)
        .then((result) => {
          this.cursos = result.data;
          console.log(result.data);
        })
        .catch((e) => {
          console.log(e);
        });
    },

    editItem(item) {
      this.editedIndex = this.cursos.indexOf(item);
      this.curso.nome = item.nome;
      this.curso.criacao = item.data_criacao;
      this.curso.professor = item.id_professor;
      this.dateFormatted = this.formatDate(item.data_criacao);
      this.selectedID = item.id_curso;
      this.dialog = true;
    },

    deleteItem(item) {
      axios.get(`${url}cursos/${item.id_curso}`)
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
      this.clear();
      setTimeout(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      }, 300);
    },

    clear() {
      this.curso.nome = '';
      this.curso.professor = '';
      this.curso.criacao = '';
      this.dateFormatted = '';
    },

    save() {
      const formData = new FormData();
      // eslint-disable-next-line
      for (const item in this.curso) {
        formData.append(item, this.curso[item]);
      }
      console.log(this.curso);
      if (this.editedIndex > -1) {
        axios.post(`${url}cursos/${this.selectedID}`, formData, {
          headers: {
            'Content-Type': 'application/json',
          },
        })
          .then((result) => {
            console.log(result.data);
            this.initialize();
            this.editedIndex = -1;
          })
          .catch((e) => {
            console.log(e);
          });
      } else {
        axios
          .post(`${url}cursos`, formData, {
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
      this.clear();
      this.close();
    },
  },
};
</script>
