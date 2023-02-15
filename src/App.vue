<template>
  <v-app>

    <v-container class="lighten-5">
      <v-row justify="center">
        <div class="qrcode-stream-wrapper">
          <qrcode-stream @decode="onDecode"></qrcode-stream>
        </div>
      </v-row>
      <v-row no-gutters class="mb-12" justify="end">
        <!-- <v-col md="8">
          <v-text-field v-model="search" append-icon="mdi-magnify" label="Search" single-line hide-details ></v-text-field>
        </v-col> -->
        <v-col md="2" justify="end">
          <v-btn elevation="2" color="primary" @click="exportData">Export</v-btn>
        </v-col>
      </v-row>
      <v-row justify="center">
        <v-col md="12">
          <v-data-table :headers="headers" :items="itens" ></v-data-table>  <!-- :search="search" -->
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
// import HelloWorld from './components/HelloWorld';
import { ExportToCsv } from 'export-to-csv';
import { QrcodeStream } from 'vue-qrcode-reader'
import axios from "axios";

export default {
  name: 'App',

  components: { QrcodeStream },

  data: () => ({
    // search: '',
    headers: [
      { text: 'Desc Item', value: 'nome', },
      { text: 'Quantidade', value: 'quantidade' },
      { text: 'Valor Unitario', value: 'valor_unitario' },
      { text: 'Estabelecimento', value: 'Estabelecimento' },
      { text: 'Data', value: 'Data' },
    ],
    itens: [],
    notas: []
  }),

  methods: {

    onDecode(qrcode) {

      if ( this.notas.indexOf(qrcode) == -1 ) {

        const URL = 'https://tjpfroexsosjadmawpyt.functions.supabase.co/nfce'

        axios.post(URL, { url: qrcode }, {
          headers: {
            'Content-Type': 'application/json',
            Authorization: "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InRqcGZyb2V4c29zamFkbWF3cHl0Iiwicm9sZSI6ImFub24iLCJpYXQiOjE2NjQ5OTY2OTcsImV4cCI6MTk4MDU3MjY5N30.PIuXu5O3KCO3edZb6rjiol6vdw-fu-UIZHRU9frcKDg"
          },
        }).then( response => {
          let Estabelecimento = response.data['Estabelecimento']
          let Data =  response.data['Data']
          response.data.Itens.forEach(e => {
            this.itens.push( { nome: e['nome'], quantidade: e['quantidade'], valor_unitario: e['valor_unitario'], Estabelecimento: Estabelecimento, Data: Data })
          });
          this.notas.push(qrcode)
        })
        .catch( error => { console.error(error) });

      }

    },

    exportData() {

      const csvExporter = new ExportToCsv({
        fieldSeparator: ',',
        quoteStrings: '"',
        decimalSeparator: '.',
        showLabels: true,
        useTextFile: false,
        useBom: false,
        useKeysAsHeaders: true
      });

      csvExporter.generateCsv(this.itens);

    },

  }
};

</script>