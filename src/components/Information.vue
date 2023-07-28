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
            <p class="text-h6 text-light-blue-darken-1">GUIDE D'UTILISATION</p>

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
                {name: "Vous devez commencer par renseigner les champs de la section DIMENSIONS DE LA PIECE", id:"1"},
                {name: "Ensuite, cliquez sur LANCER UN ENREGISTREMENT pour lancer la caméra", id:"2"},
                {name: "Veuillez à bien positionner la caméra au format PORTRAIT", id:"3"},
                {name: "Sur l'interface, vous avez le bouton AUTO qui sert à lancer l'enregistrement", id:"4"},
                {name: "Ce bouton est désactivé par défaut et le reste jusqu'à ce que la caméra soit dans une POSITION OPTIMALE", id:"5"},
                {name: "Lorsque la caméra est bien positonnée et que le bouton devient ACTIF, vous pouvez alors cliquer pour lancer l'enregistrement", id:"6"},
                {name: "Lorsque les deux flèches sont présentes, cela signifie que vous pouvez faire l'enregistrement en allant dans le sens que vous voulez (à votre gauche ou à votre droite)", id:"7"},
                {name: "Une fois que vous avez choisi un sens, la flèche à l'opposée disparaît ce qui signifie que vous devez conserver ce sens jusqu'à la fin", id:"8"},
                {name: "Les Images sont alors prises chaque 20°", id:"9"},
                {name: "Si vous rencontrez un dysfonctionnement dans l'applications, contactez moi par email : smaddy.kouame@gmail.com", id:"10"},
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