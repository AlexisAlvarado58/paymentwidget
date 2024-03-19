<template>
  <v-app>
    <v-app-bar
      app
      color="orange"
      dark
    >
  

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <PaymentTable/>
    </v-main>
  </v-app>
</template>

<script>
import PaymentTable from './components/PaymentTable';
import VueJsonToCsv from 'vue-json-to-csv';

export default {
  name: 'App',

  components: {
    PaymentTable,
  },

  data: () => ({
    //
  }),
};
</script>
