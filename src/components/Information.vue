<template>
    <v-dialog
    v-model="informationModal">

        <template v-slot:activator="{ props }">
            <v-sheet
            class="pa-3"
            color="#008CDB"
            rounded="xl"
            elevation="14"
            :height="80"
            v-bind="props"
            @click="toggleShowInfoModal">
                <div class="mt-2">
                    <v-icon
                    color="white"
                    icon="mdi-information"
                    size="35"></v-icon>

                    <span class="text-h6 text-grey-lighten-2"> Guide d'utilisation</span>
                </div>
            </v-sheet>
        </template>

        <v-sheet color="#E0E0E0" class="pa-5 text-center" rounded>
            <v-icon
            color="light-blue-darken-1"
            icon="mdi-information"
            size="35"
            class="mb-1"></v-icon>
            <p class="text-h6 text-light-blue-darken-1">NOTE D'INFORMATION</p>

            <ul>
                <li class="text-start" v-for="item in items" :key="item.id">{{ item.name }}</li>
            </ul>
        </v-sheet>
    </v-dialog>
</template>

<script>
export default {
    data(){
        return {
            informationModal: false,
            loading: false,
            items: [
                {name: "Assurez-vous de bien positionner la caméra au format PORTRAIT", id:"1"},
                {name: "Assurez-vous que le cadre autour du flux vidéo est au vert -> signe que la caméra est bien positionnée", id:"2"},
                {name: "Assurez-vous de tourner dans le sens anti-horaire -> vers votre gauche", id:"3"},
                {name: "Les Images sont alors prises chaque 20°, chose que vous pouvez changer dans LES REGLAGES", id:"4"},
                {name: "Remplir obligatoirement les champs de la section DIMENSIONS DE LA PIECE", id:"5"},
                {name: "Si vous rencontrez un dysfonctionnement dans l'applications, contactez moi par email : smaddy.kouame@gmail.com", id:"6"},
            ]
        }
    },
    methods:{
        toggleShowInfoModal(){
            this.informationModal = true
        },
    }
}
</script>