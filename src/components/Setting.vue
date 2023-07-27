<template>
    <v-dialog
    v-model="settingModal"
    persistent>

        <template v-slot:activator="{ props }">
            <v-sheet
            class="pa-3"
            color="#00B960"
            rounded="xl"
            elevation="14"
            v-bind="props"
            @click="showSettingModal">
                <v-icon
                color="white"
                icon="mdi-tools"
                size="35"
                class="mb-12"></v-icon>

                <p class="text-h6 text-grey-lighten-2">Tous mes réglages</p>
            </v-sheet>
        </template>

        <v-sheet color="#E0E0E0" class="pa-5 text-center" rounded>
            <v-icon
            color="green-accent-3"
            icon="mdi-tools"
            size="35"
            class="mb-1"></v-icon>
            <p class="text-h6 text-green-accent-3">TOUS MES REGLAGES</p>

            <v-form class="mt-3" v-model="form" @submit.prevent="handleSubmit">
                <div>
                    <div class="text-caption text-start">Intervalle de prise de vue (en degré)</div>
                    <v-slider
                    v-model="sliderValue"
                    :min="20"
                    :max="60"
                    :step="5"
                    show-ticks="always"
                    tick-size="5"
                    thumb-label="always"
                    prepend-icon="mdi-speedometer"></v-slider>
                </div>
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
    props: ['curPhotoInterval'],
    data(){
        return {
            settingModal: false,
            loading: false,
            form: false,
            photoInterval: 0,
            sliderValue : 0
        }
    },
    methods:{
        handleCanceling(){
            this.settingModal = false
            this.sliderValue = this.photoInterval
        },
        showSettingModal(){
            this.settingModal = true
        },
        handleSubmit(){
            if (!this.form) return

            this.loading = true

            setTimeout(() => {
                this.loading = false
                this.settingModal = false
                this.photoInterval = this.sliderValue
                this.$emit('onSettingValidated', this.photoInterval)
            }, 2000)
        },
    },
    mounted(){
        this.photoInterval= this.curPhotoInterval
        this.sliderValue = this.curPhotoInterval
    }
}
</script>