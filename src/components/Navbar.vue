<template>
  <div>
    <v-toolbar>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>Escolar</v-toolbar-title>

      <v-spacer></v-spacer>

      <v-toolbar-items>
        <v-btn router to="/" text>Alunos</v-btn>
        <v-btn router to="/professores" text>Professores</v-btn>
        <v-btn router to="/cursos" text>Cursos</v-btn>
      </v-toolbar-items>
    </v-toolbar>
    <v-navigation-drawer v-model="drawer" absolute left temporary>
      <v-list dense>

        <v-list-item
          v-for="item in links"
          :key="item.text"
          link
          router :to="item.route"
        >
          <v-list-item-icon>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-icon>

          <v-list-item-content>
            <v-list-item-title>{{ item.text }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
  </div>
</template>

<script>
export default {
  name: "navbar",
  data() {
    return {
      drawer: false,
      links: [
        { icon: 'mdi-book-open-page-variant', text: 'Alunos', route: '/' },
        { icon: 'mdi-book-search', text: 'Professores', route: '/professores' },
        { icon: 'mdi-check-decagram', text: 'Cursos', route: '/cursos' },
      ],
    };
  },
};
</script>
