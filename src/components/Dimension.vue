<template>
    <v-dialog
    v-model="dimensionModal"
    persistent>

        <template v-slot:activator="{ props }">
            <v-sheet
            class="pa-3"
            color="#7A00DB"
            rounded="xl"
            elevation="14"
            v-bind="props"
            @click="showDimModal">
                <v-icon
                color="white"
                icon="mdi-ruler-square"
                size="35"
                class="mb-12"></v-icon>

                <p class="text-h6 text-grey-lighten-2">Dimensions de la pièce</p>
            </v-sheet>
        </template>

        <v-sheet color="#E0E0E0" class="pa-5 text-center" rounded>
            <v-icon
            color="purple"
            icon="mdi-ruler-square"
            size="35"
            class="mb-1"></v-icon>
            <p class="text-h6 text-purple">DIMENSIONS DE LA PIECE</p>

            <v-form class="mt-3" v-model="form" @submit.prevent="handleSubmit">
                <v-row>
                    <v-col cols="12">
                        <v-text-field
                        class="text-purple"
                        label="Longueur de la pièce (m)*"
                        type="number"
                        required
                        hint="Ex: 4.60"
                        :readonly="loading"
                        :rules="[required]"
                        clearable
                        v-model="length"></v-text-field>
                    </v-col>
                    <v-col cols="12">
                        <v-text-field
                        class="text-purple"
                        label="Largeur de la pièce (m)*"
                        type="number"
                        required
                        hint="Ex: 3.00"
                        :readonly="loading"
                        :rules="[required]"
                        clearable
                        v-model="width"></v-text-field>
                    </v-col>
                    <v-col cols="12">
                        <v-text-field
                        class="text-purple"
                        label="hauteur de la pièce (m)*"
                        type="number"
                        required
                        hint="Ex: 2.5"
                        :readonly="loading"
                        :rules="[required]"
                        clearable
                        v-model="height"></v-text-field>
                    </v-col>
                </v-row>
                <v-row>
                    <v-col>
                        <v-btn
                        block
                        size="large"
                        color="red"
                        variant="elevated"
                        @click="handleCanceling">
                            Annuler
                        </v-btn>
                    </v-col>
                    <v-col>
                        <v-btn
                        :disabled="!form"
                        :loading="loading"
                        block
                        size="large"
                        color="success"
                        type="submit"
                        variant="elevated">
                            Valider
                        </v-btn>
                    </v-col>
                </v-row>
            </v-form>
        </v-sheet>
    </v-dialog>
</template>

<script>
export default {
    props: ['DimDialogState'],
    data(){
        return {
            dimensionModal: false,
            loading: false,
            form: false,
            length: null,
            width: null,
            height: null
        }
    },
    methods:{
        handleCanceling(){
            this.dimensionModal = false
        },
        showDimModal(){
            this.dimensionModal = true
        },
        handleSubmit(){
            if (!this.form) return

            this.loading = true

            setTimeout(() => {
                this.loading = false
                const data = {
                    registrationNo : Date(),
                    length: this.length,
                    width: this.width,
                    height: this.height
                }
                this.dimensionModal = false
                this.$emit('onFormValidated', data)
                this.length = null
                this.width = null
                this.height = null
            }, 2000)
        },
        required (v) {
            return !!v || 'Ce champ est obligatoire'
        },
    }
}
</script>