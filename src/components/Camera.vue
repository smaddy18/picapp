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
            :height="80"
            v-bind="props"
            @click="showCameraModal">
                <div class="mt-2">
                    <v-icon
                    color="white"
                    icon="mdi-camera"
                    size="35"></v-icon>

                    <span class="text-h6 text-grey-lighten-2"> Lancer un enregistrement</span>
                </div>
            </v-sheet>
        </template>

        <v-sheet color="#E0E0E0" class="text-center" rounded>
            <div class="myContainer">
                <div class="myContainer__toolbar">
                    <v-toolbar
                    dark
                    color="rgba(217, 217, 217, 0.0)">
                        <v-btn icon class="close__btn" dark @click="closeCameraModal">
                            <v-icon>mdi-close</v-icon>
                        </v-btn>
                        <v-spacer></v-spacer>
                        <div class="text-center mr-3 text-grey-darken-2">
                            <p class="caption">Alpha: {{ computedCurrentAlpha }} - Beta: {{ beta }} - Gamma: {{ gamma }}</p>
                        </div>
                    </v-toolbar>
                </div>
                <div class="direction_arrows d-flex justify-space-between px-3">
                    <div>
                        <v-btn icon class="direction_arrows--left" v-if="shootingDirection === 'left' || shootingDirection === ''">
                            <v-icon>mdi-arrow-left-circle-outline</v-icon>
                        </v-btn>
                    </div>
                    <div>
                        <v-btn icon class="direction_arrows--right" v-if="shootingDirection === 'right' || shootingDirection === ''">
                            <v-icon>mdi-arrow-right-circle-outline</v-icon>
                        </v-btn>
                    </div>
                </div>
                <div class="myContainer__video">
                    <!-- <p class="text-h6">Alpha: {{ alpha }} - Beta: {{ beta }} - Gamma: {{ gamma }}</p>
                    <p class="text-h6">ComputedCurrentAlpha: {{ computedCurrentAlpha }} - ComputedPrevAlpha: {{ computedPrevAlpha }}</p>
                    <p class="text-h6">Photo array len: {{ photoArray.length }}</p>
                    <p class="text-h6">Shoot direction: {{ shootingDirection }}</p> -->
                    <video
                    autoplay
                    playsInline
                    ref="videoPlayer"
                    class="mt-0"></video>
                </div>
                <div class="button__div">
                    <v-row v-if="!isShootingStarted">
                        <v-col>
                            <v-btn
                            icon="mdi-calendar"
                            size="5rem"
                            color="rgba(97, 97, 97, 0.7)"
                            class="text-white"
                            variant="elevated"
                            :disabled="!readyForShooting"
                            @click="handleShootingStart">
                                Auto
                            </v-btn>
                        </v-col>
                    </v-row>
                </div>
                <canvas id="preview-canvas" ref="canvas"></canvas>
                <!-- <canvas id="canvas" ref="canvas"></canvas> -->
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
    let streaming = false

    const photoArray = ref([])


    /**-------METHODS------- */
    const showCameraModal = () => {
        if(props.isDimDataSet===true){
            cameraModal.value = true
            getVideo()
            requestOrientation()
        }else{
            emit('onDimDataNotSet')
            closeCameraModal()
        }
    }
    const closeCameraModal = () => {
        cameraModal.value = false
        //streaming = false
    }
    const savePhoto = () => {
        emit('onShootingEnded', photoArray.value)
        photoArray.value = []
        cameraModal.value = false
    }
    const getVideo = () => {
        //const width = 320
        //let height = 0 // à calculer automatiquement en fonction de la largeur du flux entrant
        // let streaming = false
        // alert("Test de la taille1 : w : "+width+ "h : " +height)

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

                        if(video.videoHeight > video.videoWidth){
                            height = (video.videoHeight / video.videoWidth) * width;
                        }else{
                            height = (video.videoWidth / video.videoHeight) * width;
                        }
                        // alert("Test de la taille1 : w : "+width+ " h : " +height+ "\nvh: "+video.videoHeight+ " vw: "+video.videoWidth)

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
    const emit = defineEmits(['onShootingEnded, onDimDataNotSet'])
    const props = defineProps({
        isDimDataSet: Boolean
    })

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
                // alert('initialA : ' +initialAlpha+ '\ncurrentA : '+currentAlpha+'\nprevA : '+prevAlpha)
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
            shootingDirection.value = ""
            currentAlpha = 0
            prevAlpha = 0
            isShootingAlreadyStarted = false

            // Enregistrement de la photo
            savePhoto()
        }
    }

    const handleShootingStart = () =>{
        isShootingStarted = true
        isShootingEnded = false
        takePhoto() // On prend la première photo dès le lancement
    }
</script>

<style scoped>
    .myContainer{
        position: relative;
        width: 100%;
        height: 100%;
    }
    .myContainer__video{
        position: fixed;
        width: 100%;
        height: 100%;
        z-index: 0;
    }
    .myContainer__toolbar{
        position: absolute;
        width: 100%;
        z-index: 2;
    }

    .direction_arrows{
        position: absolute;
        top: 50%;
        z-index: 2;
        width: 100%;
    }

    .direction_arrows--left, .direction_arrows--right{
        background-color: rgba(97, 97, 97, 0.3);
        color: white;
    }

    video{
        width: 100%;
        height: 100%;
        object-fit: fill;
    }

    .close__btn{
        background-color: rgba(97, 97, 97, 0.3);
        color: white;
    }

    .button__div{
        position: absolute;
        bottom: 0px;
        z-index: 3;
        width: 100%;
        height: 8rem;
    }

    #canvas{
        display: none;
    }

    #preview-canvas{
        /* background-color: cyan; */
        position: absolute;
        z-index: 2;
        bottom: 0px;
        left: 0px;
        width: 7rem;
        height: 7rem;
        margin-left: 2px;
        margin-bottom: 2px;
        background-image: linear-gradient(
            to bottom,
            rgba(230, 230, 230, 0.5),
            rgba(51, 51, 51, 0.5)
        ), url("../assets/image_icon.png");
        background-repeat: no-repeat;
        background-position: center;
    }
</style>