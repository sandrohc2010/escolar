<template>
  <v-container>
    <div>
      <v-toolbar flat color="white">
        <v-toolbar-title>Gerenciar Alunos</v-toolbar-title>
        <v-divider class="mx-2" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-btn color="orange" @click="gerarPDF">Gerar Relatório</v-btn>
      </v-toolbar>
    </div>
    <v-data-table
      :headers="headers"
      :items="alunos"
      :expanded.sync="expanded"
      item-key="nome"
      show-expand
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat color="white">
          <v-toolbar-title>Expandable Table</v-toolbar-title>
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
                      <v-text-field v-model="aluno.nome" label="Nome"></v-text-field>
                    </v-flex>
                    <v-flex xs12 lg6>
                      <div>
                        <v-select
                        :items="cursos"
                        v-model="aluno.curso"
                        item-text="nome"
                        item-value="id_curso"
                        label="Curso"
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
                            label="Data de Nascimento"
                            hint="MM/DD/YYYY format"
                            persistent-hint
                            prepend-icon="mdi-calendar"
                            @blur="aluno.nascimento = parseDate(dateFormatted)"
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-date-picker
                          v-model="aluno.nascimento"
                          no-title
                          @input="datePickerNascimento"
                        ></v-date-picker>
                      </v-menu>
                      <p>
                        Date in ISO format:
                        <strong>{{ aluno.nascimento }}</strong>
                      </p>
                      <v-flex xs12>
                        <v-text-field v-model="aluno.cep" v-mask="mask" label="CEP" @input="buscaCep"></v-text-field>
                      </v-flex>
                      <v-flex xs6>
                        <v-text-field v-model="aluno.estado" label="Estado"></v-text-field>
                      </v-flex>
                      <v-flex xs6>
                        <v-text-field v-model="aluno.cidade" label="Cidade"></v-text-field>
                      </v-flex>
                      <v-flex xs12>
                        <v-text-field v-model="aluno.bairro" label="Bairro"></v-text-field>
                      </v-flex>
                      <v-flex xs12>
                        <v-text-field v-model="aluno.logradouro" label="Logradouro"></v-text-field>
                      </v-flex>

                      <v-flex xs12>
                        <v-text-field v-model="aluno.numero" label="Número"></v-text-field>
                      </v-flex>

                      <v-flex xs12 lg6>
                        <v-menu
                          ref="menu"
                          v-model="menu"
                          :close-on-content-click="false"
                          transition="scale-transition"
                          offset-y
                          full-width
                          max-width="290px"
                          min-width="290px"
                        >
                          <template v-slot:activator="{ on }">
                            <v-text-field
                              v-model="criacaoFormatted"
                              label="Data de Criação"
                              hint="MM/DD/YYYY format"
                              persistent-hint
                              prepend-icon="mdi-calendar"
                              @blur="aluno.criacao = parseDate(criacaoFormatted)"
                              v-on="on"
                            ></v-text-field>
                          </template>
                          <v-date-picker
                            v-model="aluno.criacao"
                            no-title
                            @input="datePickerCriacao"
                          ></v-date-picker>
                        </v-menu>
                      </v-flex>
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
      <template v-slot:expanded-item="{ item, headers }">
        <td :colspan="headers.length">
          <v-layout wrap>
            <v-flex xs6 my-2>Estado: {{item.estado}}</v-flex>
            <v-flex xs6 my-2>Cidade: {{item.cidade}}</v-flex>
            <v-flex xs12 my-2>Bairro: {{item.bairro}}</v-flex>
            <v-flex xs6 my-2>Logradouro: {{item.logradouro}}</v-flex>
            <v-flex xs6 my-2>CEP: {{item.cep}}</v-flex>
            <v-flex xs6 my-2>Número: {{item.numero}}</v-flex>
          </v-layout>
        </td>
      </template>
      <template v-slot:item.data_nascimento="{ item }">
        {{formatDate(item.data_nascimento)}}
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
import JSPDF from 'jspdf';
import 'jspdf-autotable';
import { mask } from 'vue-the-mask';
import { url } from "../config";

//axios.defaults.headers.common['Access-Control-Allow-Origin'] = '*';

export default {
  directives: {
    mask,
  },
  data() {
    return {
      cursos: [],
      alunos: [],
      modal: false,
      dialog: false,
      dateFormatted: this.formatDate(new Date().toISOString().substr(0, 10)),
      criacaoFormatted: this.formatDate(new Date().toISOString().substr(0, 10)),
      menu1: false,
      menu: false,
      headers: [
        {
          text: "Código",
          align: "left",
          sortable: false,
          value: "id_aluno"
        },
        { text: "Aluno", value: "nome" },
        { text: "Curso", value: "curso" },
        { text: 'Data Nascimento', value: 'data_nascimento' },
        { text: "Actions", value: "action", sortable: false },
      ],
      editedIndex: -1,
      aluno: {
        nome: '',
        curso: '',
        criacao: '',
        estado: '',
        cidade: '',
        bairro: '',
        logradouro: '',
        numero: '',
        cep: '',
        nascimento: '',
      },
      selectedID: -1,
      expanded: [],
      mask: '#####-###',
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
    buscaCep() {
      if (this.aluno.cep.length === 9) {
        const { cep } = this.aluno;
        axios.get(`https://api.postmon.com.br/v1/cep/${cep.replace('-', '')}`)
          .then((result) => {
            const r = result.data;
            this.aluno.estado = r.estado_info.nome;
            this.aluno.cidade = r.cidade;
            this.aluno.bairro = r.bairro;
            this.aluno.logradouro = r.logradouro;
            console.log(r);
          });
      }
    },
    formatDate(date) {
      if (!date) return null;

      const [year, month, day] = date.split("-");
      return `${day}/${month}/${year}`;
    },
    datePickerNascimento() {
      this.menu1 = true;
      this.dateFormatted = this.formatDate(this.aluno.nascimento);
    },
    datePickerCriacao() {
      this.menu = true;
      this.criacaoFormatted = this.formatDate(this.aluno.criacao);
    },
    parseDate(date) {
      if (!date) return null;

      const [day, month, year] = date.split("/");
      console.log(date);
      return `${year}-${month.padStart(2, "0")}-${day.padStart(2, "0")}`;
    },
    initialize() {
      axios
        .get(`${url}alunos`)
        .then((result) => {
          this.alunos = result.data;
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
      this.editedIndex = this.alunos.indexOf(item);
      this.aluno.nome = item.nome;
      this.aluno.criacao = item.data_criacao;
      this.criacaoFormatted = this.formatDate(item.data_criacao);
      this.aluno.nascimento = item.data_nascimento;
      this.dateFormatted = this.formatDate(item.data_nascimento);
      this.aluno.curso = item.id_curso;
      this.aluno.estado = item.estado;
      this.aluno.cidade = item.cidade;
      this.aluno.bairro = item.bairro;
      this.aluno.logradouro = item.logradouro;
      this.aluno.numero = item.numero;
      this.aluno.cep = item.cep;
      this.dialog = true;
      this.selectedID = item.id_aluno;
    },

    deleteItem(item) {
      axios.get(`${url}alunos/${item.id_aluno}`)
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
      // eslint-disable-next-line
      for (const item in this.aluno) {
        this.aluno[item] = '';
      }
    },

    save() {
      const formData = new FormData();
      // eslint-disable-next-line
      for (const item in this.aluno) {
        formData.append(item, this.aluno[item]);
      }
      console.log(this.aluno);
      if (this.editedIndex > -1) {
        axios.post(`${url}alunos/${this.selectedID}`, formData, {
          headers: {
            'Content-Type': 'application/json',
          },
        })
          .then((result) => {
            console.log(result.data);
            this.initialize();
            this.editedIndex = -1;
            this.clear();
          })
          .catch((e) => {
            console.log(e);
          });
      } else {
        axios
          .post(`${url}alunos`, formData, {
            headers: {
              'Content-Type': 'application/json',
            },
          })
          .then((result) => {
            console.log(result);
            this.initialize();
            this.clear();
          })
          .catch((e) => {
            console.log(e);
          });
      }
      this.clear();
      this.close();
    },
    gerarPDF() {
      let dia = new Date();
      const dd = String(dia.getDate()).padStart(2, '0');
      const mm = String(dia.getMonth() + 1).padStart(2, '0'); //January is 0!
      const yyyy = dia.getFullYear();

      dia = `${dd}/${mm}/${yyyy}`;
      const doc = new JSPDF();
      const arr = [];
      // eslint-disable-next-line
      for (let item in this.alunos) {
        arr.push([this.alunos[item].nome, this.alunos[item].curso, this.alunos[item].professor]);
      }
      doc.text('Relação de Alunos', 10, 10);
      doc.text(`Gerado em: ${dia}`, 10, 17);
      doc.autoTable({
        margin: { top: 30 },
        head: [['Nome do Aluno', 'Curso', 'Professor']],
        body: arr,
      });
      console.log(dia);
      doc.save(`Relatório_alunos_${dia}`);
    },
  },
};
</script>
