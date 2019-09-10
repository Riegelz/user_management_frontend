<template>
  <v-app>
    <v-app-bar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>USER</span>
        <span class="font-weight-light">MANAGEMENT</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn
        text
        href="https://github.com/vuetifyjs/vuetify/releases/latest"
        target="_blank"
      >
      </v-btn>
    </v-app-bar>

    <v-content>
      <Datatable/>
    </v-content>
  </v-app>
</template>

<script>
import Datatable from './components/Datatable';

export default {
  name: 'App',
  components: {
    Datatable,
  },
  data: () => ({
    //
  }),
};
</script>
