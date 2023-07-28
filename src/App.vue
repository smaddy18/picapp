<template>
	<v-layout>
		<v-app-bar :elevation="2" rounded="xl" color="#333333" class="text-white" >
			<template v-slot:prepend>
				<v-icon>mdi-scan-helper</v-icon>
			</template>

			<v-app-bar-title>Room Scan</v-app-bar-title>

			<template v-slot:append>
				<v-btn icon="mdi-dots-vertical"></v-btn>
			</template>
		</v-app-bar>

		<v-main>
			<v-container fluid>
				<h1 class="text-h5 text-white">Bienvenue sur <span class="font-weight-bold text-">Room Scan</span></h1>
				<p class="mt-4 text-subtitle-1 text-grey-lighten-2">Pour commencer, renseigner les différentes informations demandées plus bas</p>

				<!-- <p class="text-h6">Room imensions: {{ Object.keys(roomDimensions).length }}</p>
				<p class="text-h6">Photo array len: {{ photos.length }}</p> -->

				<v-row class="my-4">
					<v-col cols="12">
						<Information />
					</v-col>
					<v-col cols="12">
						<Dimension @onFormValidated="handleFormValidated"/>
					</v-col>
					<v-col cols="12">
						<Camera :isDimDataSet="isDimensionDataSet" @onDimDataNotSet="handleDimDataNotSet" @onShootingEnded="handleShootingEnded"/>
					</v-col>
					<!-- <v-col cols="6">
						<Setting :curPhotoInterval="photoInterval" @onSettingValidated="handleSettingValidated"/>
					</v-col> -->
				</v-row>
				<div class="text-center mt-5 text-grey-darken-2">
					<p class="text-h6">Room Scan v1.0.0</p>
				</div>
			</v-container>
			<div class="text-center ma-2">
				<v-snackbar
				v-model="snackbar"
				:timeout="5000"
				color="success">
					Données enregistrées avec succès
				</v-snackbar>
			</div>
			<div class="text-center ma-2">
				<v-snackbar
				v-model="dimensionSnackbar"
				:timeout="5000"
				color="warning">
					Renseigner d'abord les dimensions de la pièce
				</v-snackbar>
			</div>
		</v-main>
	</v-layout>
</template>

<script>
import Dimension from './components/Dimension.vue'
import Information from './components/Information.vue'
import Camera from './components/Camera.vue'
import Setting from './components/Setting.vue'

import axios from 'axios'

export default {
  components: { Dimension, Information, Setting, Camera },
	data (){
		return {
			sheets: [
				{title: "Note d'information", sheetIcon: "mdi-information", bgColor:"#008CDB"},
				{title: "Dimensions de la pièce", sheetIcon: "mdi-ruler-square", bgColor:"#7A00DB"},
				{title: "Lancer un enregistrement", sheetIcon: "mdi-camera", bgColor:"#DB004F"},
				{title: "Tous mes réglages", sheetIcon: "mdi-tools", bgColor:"#00B960"},
			],
			photos: [],
			roomDimensions: {
				registrationNo : Date(),
				length: 4,
				width: 3,
				height: 2
			},
			photoInterval: 20,
			infoModal: false,
			dimensionModal: false,
			cameraModal: false,
			settingModal: false,
			isDimensionDataSet: false,
			snackbar: false,
			dimensionSnackbar: false
		}
	},
	methods: {
		handleFormValidated(data){
			console.log(data)
			this.roomDimensions = {...data}
			this.isDimensionDataSet = true
			//alert("dimension len : " +Object.keys(this.roomDimensions).length)
		},
		handleShootingEnded(photoArray){
			this.photos = [...photoArray]
			this.sendDataToAPI()
			this.snackbar = true
			// alert("photo len : " +photoArray.length)
		},
		handleSettingValidated(data){
			this.photoInterval = data
		},

		handleDimDataNotSet(){
			this.dimensionSnackbar = true
		},

		sendDataToAPI () {
            // Préparez les données à envoyer à l'API, y compris la photo et les autres informations
            const data = {...this.roomDimensions, images : this.photos}

			axios
			.post('https://picapp-api-3.vercel.app/roompic/upload/', data)
			.then(response => {
				alert("Données enregistrées avec succès")
				console.log('Response : '+ response)
			})
			.catch(error => {
				//alert("Error: "+error)
				if (error.response) {
					// The request was made and the server responded with a status code
					// that falls out of the range of 2xx
					alert(error.response.status);
					alert(error.response.data);
					alert(error.response.headers);
				} else if (error.request) {
					// The request was made but no response was received
					// `error.request` is an instance of XMLHttpRequest in the browser and an instance of
					// http.ClientRequest in node.js
					alert(JSON.stringify(error.toJSON(), null, 4));
				} else {
					// Something happened in setting up the request that triggered an Error
					alert('Error'+ error.message);
				}
			})
        },
	}
}
</script>

<style>

</style>