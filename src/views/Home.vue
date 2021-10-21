<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Por onde andei?</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <div class='container'>
          <h1>Coordenadas atuais</h1>
          <p>Latitude: {{  latitude  }}</p>
          <p>Longitude: {{  longitude  }}</p>
          <ion-button expand="block" @click="buscaCidade()">REGISTRAR POSIÇÃO</ion-button>
      </div>
      <div>
      <ion-grid>
        <ion-row>
          <ion-col>
            <ul v-for="(cidade, index) in list" v-bind:key="index">
              <li>{{cidade}}</li>
            </ul>
          </ion-col>
        </ion-row>
      </ion-grid>
      </div>
    </ion-content>
  </ion-page>
</template>

<script>
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonGrid, IonRow, IonCol, alertController } from '@ionic/vue';
import { defineComponent } from 'vue';
import { Geolocation } from '@capacitor/geolocation';
import { Http } from '@capacitor-community/http';
import { Storage } from '@ionic/storage'

export default defineComponent({
  name: 'Home',
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonButton,
    IonGrid, 
    IonRow,
    IonCol
  },
  data() {
    return {
      localStorage: new Storage(),
      latitude: 0,
      longitude: 0,
      coordenadas: '',
      cidade: '',
      list: []
    }
  },
  ionViewWillEnter(){
    this.printCurrentPosition();

  },
  created: async function () {
    await this.localStorage.create();

  },
  mounted: async function() {
    const list = await this.localStorage.get('list');
      if (JSON.parse(list) == null) {
        this.list = [];
      }
      else
        this.list = JSON.parse(list);
  },   

  methods: {
    printCurrentPosition: async function() {
      const coordinates = await Geolocation.getCurrentPosition();
      console.log('Current position: ', coordinates);

        this.latitude = coordinates.coords.latitude;
        this.longitude = coordinates.coords.longitude; 
    },
    buscaCidade: async function() {
        const ACCESS_KEY = '8cf68e16c982b92f6b8a0793785c61be';

        const options = {
          url: `http://api.positionstack.com/v1/reverse?access_key=${ACCESS_KEY}&query=${this.latitude},${this.longitude}&limit=1`
        };

        const response = await Http.get(options);
        console.log('Resposta recebida: ', response);

        this.cidade = response.data.data[0].locality + ',' + response.data.data[0].region_code;
          this.coordenadas = this.latitude + ' , ' + this.longitude;
          this.list.push("===========================");
          this.list.push("Localidade: " + this.cidade);
          this.list.push("Coordenadas: " + this.coordenadas);
          await this.localStorage.set('list', JSON.stringify(this.list));
          console.log(this.list);
          const mensagem = this.cidade;
          const alerta = await alertController
           .create({
          cssClass: 'my-custom-class',
          header: 'Posição registrada!',
          message: mensagem,
          buttons: ['OK'],
        });
          await alerta.present();
    },
  }
});
</script>

<style scoped>
.container {
  text-align: center;
}
</style>