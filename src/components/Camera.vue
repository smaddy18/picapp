<template>
    <v-dialog
    v-model="cameraModal"
    persistent
    fullscreen
    :scrim="false"
    transition="dialog-bottom-transition">

        <template v-slot:activator="{ props }">
            <v-sheet
            class="pa-3"
            color="#DB004F"
            rounded="xl"
            elevation="14"
            v-bind="props"
            @click="showCameraModal">
                <v-icon
                color="white"
                icon="mdi-camera"
                size="35"
                class="mb-12"></v-icon>

                <p class="text-h6 text-grey-lighten-2">Lancer un enregistrement</p>
            </v-sheet>
        </template>

        <v-sheet color="#E0E0E0" class="pa-2 text-center" rounded>
            <div class="myContainer">
                <div class="myContainer__toolbar">
                    <v-toolbar
                    dark
                    color="primary">
                        <v-btn icon dark @click="closeCameraModal">
                            <v-icon>mdi-close</v-icon>
                        </v-btn>
                        <v-spacer></v-spacer>
                        <v-toolbar-items>
                            <v-btn variant="text" @click="savePhoto">
                                Sauvegarder
                            </v-btn>
                        </v-toolbar-items>
                    </v-toolbar>
                </div>
                <div class="mt-2 myContainer__video">
                    <p class="text-h6">Alpha: {{ alpha }} - Beta: {{ beta }} - Gamma: {{ gamma }}</p>
                    <p class="text-h6">ComputedCurrentAlpha: {{ computedCurrentAlpha }} - ComputedPrevAlpha: {{ computedPrevAlpha }}</p>
                    <p class="text-h6">Photo array len: {{ photoArray.length }}</p>
                    <p class="text-h6">Shoot direction: {{ shootingDirection }}</p>
                    <video
                    autoplay
                    playsInline
                    ref="videoPlayer"
                    class="mt-0"></video>
                </div>
                <v-row v-if="!isShootingStarted">
                    <v-col>
                        <v-btn
                        block
                        size="large"
                        color="green"
                        variant="elevated"
                        :disabled="!readyForShooting"
                        @click="handleShootingStart">
                            Commencer
                        </v-btn>
                    </v-col>
                </v-row>
                <canvas id="canvas" ref="canvas"></canvas>
            </div>
        </v-sheet>
    </v-dialog>
</template>

<script setup>
    import { ref, onMounted } from 'vue'

    const BETA_UP_LIMIT = 100
    const BETA_DOWN_LIMIT = 70
    const GAMMA_UP_LIMIT = 100
    const GAMMA_DOWN_LIMIT = -10

    /**-------DATA------- */
    let cameraModal = ref(false)
    const videoPlayer = ref(null)
    const canvas = ref(null)
    const width = 320
    let height = 0  // à calculer automatiquement en fonction de la largeur du flux entrant
    let hasPhoto = false

    const photoArray = ref([])


    /**-------METHODS------- */
    const showCameraModal = () => {
        /*DeviceOrientationEvent.requestPermission()
        .then((permission) => {
            if(permission === 'granted'){
                //alert('konan fabrice : permission : '+ permission)
                window.addEventListener('deviceorientation', handleOrientationEvent, false);
            }else{
                //output.textContent = "konan fabrice : The user denied permission";
                //this.outputContainer.textContent = "konan fabrice : The user denied permission";
                alert('konan fabrice : The user denied permission')
            }
        }).catch(console.error)*/


        cameraModal.value = true
        getVideo()
        requestOrientation()
    }
    const closeCameraModal = () => {
        cameraModal.value = false
    }
    const savePhoto = () => {
        emit('onShootingEnded', photoArray.value)
        photoArray.value = []
        cameraModal.value = false
    }
    const getVideo = () => {
        //const width = 320
        //let height = 0 // à calculer automatiquement en fonction de la largeur du flux entrant
        let streaming = false

        // Vérifie si le navigateur prend en charge les médias
        navigator.mediaDevices
        .getUserMedia({
            video : {facingMode : 'environment'}
        })
        .then(stream => {
            let video = videoPlayer.value
            video.srcObject = stream
            //video.play()

            video.addEventListener(
                'canplay',
                (ev) => {
                    if (!streaming) {
                    height = (video.videoHeight / video.videoWidth) * width;

                    video.setAttribute("width", width);
                    video.setAttribute("height", height);
                    canvas.value.setAttribute("width", width);
                    canvas.value.setAttribute("height", height);
                    streaming = true;
                    }
                },
                false
            );
        })
        .catch(err => {
            console.error('Erreur lors de l\'accès à la caméra :', err);
        })
    }

    const takePhoto = () => {
        let video = videoPlayer.value
        let photo = canvas.value

        photo.width = width
        photo.height = height

        let context = photo.getContext('2d')
        context.drawImage(video, 0, 0, width, height)
        hasPhoto = true

        // Ajout de l'image capturée dans le Array d'images
        // On prend le soin de convertir d'abord l'image au format .jpeg
        const dataURL = photo.toDataURL('image/jpeg');
        photoArray.value = [...(photoArray.value), dataURL]
    }

    const requestOrientation = () => {
        DeviceOrientationEvent.requestPermission()
        .then((permission) => {
            if(permission === 'granted'){
                window.addEventListener('deviceorientation', (e) => {
                    handleOrientationEvent(e)
                }, false);
            }
        }).catch(console.error)
    }

    // Orientation data
    let alpha = ref(0)
    let beta = ref(90)
    let gamma = ref(0)
    let initialAlpha = 0
    let currentAlpha = 0
    let prevAlpha = 0

    let computedCurrentAlpha = ref(0) // Conversion de initialAlpha en une valeur entre [0 - 359]
    let computedPrevAlpha = ref(0)   // idem pour prevAlpha

    let isShootingAlreadyStarted = false
    let isShootingStarted = false
    let isShootingEnded = true

    let makeCapture = false
    let readyForShooting = ref(false)
    let shootingDirection = ref("")
    const emit = defineEmits(['onShootingEnded'])

    const handleOrientationEvent = (event) => {
        //alert('konan fabrice : In handleOrientationEvent function. Alpha : '+ Math.floor(event.alpha, 2))
        //const output2 = document.createElement('div');
        // Gérer les données d'orientation ici
        //console.log('Orientation des données :', frontToBack);
        //output2.textContent = "konan fabrice : beta : "+Math.floor(event.beta, 2)+ " gamma : "+Math.floor(event.gamma, 2)+" alpha : "+ Math.floor(event.alpha, 2);
        //this.outputContainer.textContent = "konan fabrice : beta : "+Math.floor(event.beta, 2)+ " gamma : "+Math.floor(event.gamma, 2)+" alpha : "+ Math.floor(event.alpha, 2);

        alpha.value = Math.floor(event.alpha, 2)
        beta.value = Math.floor(event.beta, 2)
        gamma.value = Math.floor(event.gamma, 2)


        readyForShooting.value = (beta.value >= BETA_DOWN_LIMIT && beta.value <= BETA_UP_LIMIT) && (gamma.value >= GAMMA_DOWN_LIMIT && gamma.value <= GAMMA_UP_LIMIT)
        
        if(isShootingStarted === true){
            if(!isShootingAlreadyStarted){
                initialAlpha = alpha.value
                currentAlpha = alpha.value
                prevAlpha = alpha.value
                isShootingAlreadyStarted = true
                alert('initialA : ' +initialAlpha+ '\ncurrentA : '+currentAlpha+'\nprevA : '+prevAlpha)
            }else{
                currentAlpha = alpha.value
            }
        }

        if(currentAlpha >= initialAlpha && !isShootingEnded){
            computedCurrentAlpha.value = currentAlpha - initialAlpha
        }else if(currentAlpha < initialAlpha && !isShootingEnded){
            computedCurrentAlpha.value = (360 + currentAlpha) - initialAlpha
        }

        if(photoArray.value.length < 2 && !isShootingEnded){
            if(computedCurrentAlpha.value >= 20 && computedCurrentAlpha.value < 180){
                takePhoto()
                shootingDirection.value = "left"
                computedPrevAlpha.value = 20
            }else if(computedCurrentAlpha.value <= 340 && computedCurrentAlpha.value >= 180){
                takePhoto()
                shootingDirection.value = "right"
                computedPrevAlpha.value = 340
            }
        }

        if(shootingDirection.value === "left" && !isShootingEnded){
            if(computedCurrentAlpha.value >= computedPrevAlpha.value){
                let alphaOffset = computedCurrentAlpha.value - computedPrevAlpha.value
                if(alphaOffset >= 20 && alphaOffset < 30){
                    takePhoto()
                    computedPrevAlpha.value += 20
                }
            }
        }else if(shootingDirection.value === "right" && !isShootingEnded){
            if(computedCurrentAlpha.value <= computedPrevAlpha.value){
                let alphaOffset = computedPrevAlpha.value - computedCurrentAlpha.value
                if(alphaOffset >= 20 && alphaOffset < 30){
                    takePhoto()
                    computedPrevAlpha.value -= 20
                }
            }
        }

        //const LEFT_END = computedCurrentAlpha.value === 359 && shootingDirection.value === "left" && computedPrevAlpha >= 340
        const LEFT_END = computedCurrentAlpha.value === 359 && shootingDirection.value === "left"
        //const RIGHT_END = computedCurrentAlpha.value === 1 && shootingDirection.value === "right" && computedPrevAlpha <= 20
        const RIGHT_END = computedCurrentAlpha.value === 1 && shootingDirection.value === "right"
        
        if((LEFT_END || RIGHT_END) && !isShootingEnded){ // Shooting ended
            alert("Enregistrement terminé")
            //emit('onShootingEnded', photoArray.value)

            computedCurrentAlpha.value = 0
            computedPrevAlpha.value = 0
            isShootingStarted = false
            isShootingEnded = true
            shootingDirection = ""
            //photoArray.value = []
            currentAlpha = 0
            prevAlpha = 0
        }
    }

    const handleShootingStart = () =>{
        isShootingStarted = true
        isShootingEnded = false
        takePhoto() // On prend la première photo dès le lancement
    }
</script>

<style scoped>
    /* .myContainer{
        position: relative;
    } */
    .myContainer__video{
        width: 100%;
        height: 100%;
        /* position: absolute; */
        background-color: red;
        z-index: 0;
    }
    .myContainer__toolbar{
        /* position: absolute; */
        width: 100%;
        z-index: 2;
    }

    video{
        border: 1px solid black;
        box-shadow: 5px 5px 3px black;
        width: 100%;
        height: 100%;
        object-fit: cover;
    }

    /*#canvas{
        display: none;
    }*/
</style>