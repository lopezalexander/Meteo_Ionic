<template>
  <ion-page>
    <!-- MARK: HEADER -->
    <ion-header>
      <ion-toolbar color="primary">
        <ion-title>Ma Météo</ion-title>
      </ion-toolbar>
    </ion-header>

    <!-- MARK: MAIN -->
    <ion-content id="ionContent" :fullscreen="true">
      <!-- <ion-header :collapse="condense">
        <ion-toolbar color="secondary">
          <ion-title>Ma Meteo second</ion-title>
        </ion-toolbar>
      </ion-header> -->

      <div id="mainContainer">
        <div id="dateContainer">
          <ion-item color="none" lines="none">
            <ion-label>{{ data.today }}</ion-label>
          </ion-item>
        </div>
        <!--  -->
        <!-- CITY SELECTION DIV -->
        <div id="selectionContainer">
          <ion-item color="none" lines="none">
            <ion-label>Ville</ion-label>
            <ion-select placeholder="Choisir" @ionChange="getCity($event.detail.value)">
              <ion-select-option value="Montréal">Montréal</ion-select-option>
              <ion-select-option value="Laval">Laval</ion-select-option>
              <ion-select-option value="Québec">Québec</ion-select-option>
              <ion-select-option value="Local">Position Actuelle</ion-select-option>
            </ion-select>
          </ion-item>
        </div>

        <!-- OUTPUT OF ASYNC REQUEST -->
        <div id="outputContainer">
          <!-- CITY LABEL -->
          <ion-item color="none" lines="none">
            <ion-label class=".ion-text-center">{{ data.ville }}</ion-label>
          </ion-item>
          <!-- TEMP -->
          <ion-item color="none" lines="none">
            <ion-label class=".ion-text-center">{{ data.temp }}°C</ion-label>
          </ion-item>
          <!-- IMAGE CLOUD -->
          <ion-item color="none" lines="none">
            <ion-img :src="data.imgCloud"></ion-img>
          </ion-item>
          <!-- WEATHER TYPE -->
          <ion-item color="none" lines="none">
            <ion-label class=".ion-text-center">{{ data.weatherType }}</ion-label>
          </ion-item>
          <!-- COORDINATES -->
          <ion-item color="none" lines="none">
            <ion-label class=".ion-text-center"> Latitude: {{ data.latitude }}</ion-label>
          </ion-item>
          <ion-item color="none" lines="none">
            <ion-label class=".ion-text-center"> Longitude: {{ data.longitude }}</ion-label>
          </ion-item>
        </div>
      </div>
    </ion-content>

    <!-- MARK: FOOTER -->
    <ion-footer>
      <ion-toolbar color="primary">
        <ion-title>© &nbsp;&nbsp;Edgardo Alexander Lopez & Adnan</ion-title>
      </ion-toolbar>
    </ion-footer>
  </ion-page>
</template>

MARK: SCRIPTS
<script lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonItem, IonSelect, IonSelectOption, IonFooter, IonLabel, IonImg } from "@ionic/vue";
import { defineComponent, ref } from "vue";
import { loadingController } from "@ionic/vue";
import { Geolocation } from "@capacitor/geolocation";
import { format, parseISO } from "date-fns";

export default defineComponent({
  name: "HomePage",
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonItem,
    IonSelect,
    IonSelectOption,
    IonFooter,
    IonLabel,
    IonImg,
  }, //END COMPONENTS
  methods: {
    async getCity(msg: string) {
      // console.log("TEST");
      // console.log("Selection=" + msg);

      const loading = await loadingController.create({
        message: "Attendre SVP...",
      });

      await loading.present();

      if (msg == "Local") {
        //console.log("LOCAL");

        const coordinates = await Geolocation.getCurrentPosition();
        //console.log("Current", coordinates);
        this.data.latitude = coordinates.coords.latitude;
        this.data.longitude = coordinates.coords.longitude;
      } else {
        //console.log("NOT LOCAL");

        //API KEY : ecfba498b4dd162afc691a02427e894a

        // GEOLOCATISATION API
        // http://api.openweathermap.org/geo/1.0/direct?q={city name} &limit={limit}&appid={API key}

        let url1 = `http://api.openweathermap.org/geo/1.0/direct?q=${msg}&limit=1&appid=ecfba498b4dd162afc691a02427e894a`;

        await fetch(url1)
          .then((response) => response.json())
          .then((data) => {
            // console.log("LOCATION DATA");
            // console.log(data);
            // console.log(data[0].lat);
            // console.log(data[0].lon);
            this.data.latitude = data[0].lat;
            this.data.longitude = data[0].lon;

            loading.dismiss();
          });
      }

      // WEATHER API
      //https://api.openweathermap.org/data/2.5/weather?lat={lat}&lon={lon}&appid={API key}

      let url2 = `https://api.openweathermap.org/data/2.5/weather?lat=${this.data.latitude}&lon=${this.data.longitude}&units=metric&appid=ecfba498b4dd162afc691a02427e894a`;
      fetch(url2)
        .then((response) => response.json())
        .then((data) => {
          console.log("WEATHER DATA");
          console.log(data);

          // GET CITY NAME
          this.data.ville = data.name;

          // GET WEATHER TYPE

          this.data.weatherType = data.weather[0].description;

          // GET TEMP
          this.data.temp = data.main.temp;

          // GET CLOUD IMAGE
          this.data.imgCloud = `assets/image/${data.weather[0].icon}.svg`;
          console.log(this.data.imgCloud);

          loading.dismiss();
        });
    },
  }, //END FUNCTIONS
  ionViewDidEnter() {
    console.log("Ion View Did Enter");
    // GET A DEFAULT WEATHER DATA, CURRENT LOCATION
    this.getCity("Local");

    // GET DATE INFORMATION AND FORMAT
    const dateFromIonDatetime = new Date().toISOString();
    const formattedString = format(parseISO(dateFromIonDatetime), "iiii, d MMM yyyy");

    this.data.today = formattedString;
  },
  setup() {
    const data = ref();

    data.value = {
      ville: "",
      latitude: "",
      longitude: "",
      temp: "",
      weatherType: "",
      imgCloud: "",
      today: "",
    };

    return { data };
  },
  // data() {
  //   return {
  //     ville: "",
  //   };
  // },
}); //END DefineComponent
</script>

MARK: STYLES
<style scoped>
#dateContainer {
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
  width: 100%;
}
#mainContainer {
  background-image: url("../../public/assets/image/sky.jpg");
  background-repeat: no-repeat;
  background-size: cover;
  height: 100%;
  display: flex;
  flex-direction: column;
  /*align-items: center;*/
}

#selectionContainer {
  width: 100%;
}

#outputContainer {
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

#footer {
  height: 100%;
  background-color: var(--ion-color-primary);
}

ion-item {
  --background: transparent;
  --color: black;
  font-weight: 700;
}

ion-select {
  --placeholder-opacity: 1;
}
</style>
