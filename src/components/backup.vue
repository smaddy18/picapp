<script setup>
const myf = () =>{
    if(isShootingStarted === true){
            if(!isShootingAlreadyStarted){
                initialAlpha = alpha.value
                currentAlpha = alpha.value
                prevAlpha = alpha.value

                isShootingAlreadyStarted = true
            }else{
                currentAlpha = alpha.value
            
                // ici, on transforme/convertit la valeur courante de alpha
                // en une valeur comprise entre [0, 359]
                if(currentAlpha >= initialAlpha){
                    computedCurrentAlpha.value = currentAlpha - initialAlpha
                }else{
                    computedCurrentAlpha.value = (360 + currentAlpha) - initialAlpha
                }

                // On vérifie si une deuxième photo a été prise :
                //  - si oui, l'utilisateur EST TENUE de garder le même sens que celui où la 2e a été prise
                //  - sinon, l'utilisateur peut toujours aller dans le sens qu'il veut -> gauche ou droite
                if(photoArray.value.length === 1){
                    // On a pas encore choisi de sens
                    if(computedCurrentAlpha.value >= 20 && computedCurrentAlpha.value <= 340){
                        if(computedCurrentAlpha < 180){ // 360/2 -> on voit si on est dans première moitié de 360
                            // On prend la 2e photo et on garde le sens GAUCHE 
                            takePhoto()
                            shootingDirection = "left"
                            computedPrevAlpha.value = 20
                        }else{
                            //On prend la 2e photo et on garde le sens DROIT
                            takePhoto()
                            shootingDirection = "right"
                            computedPrevAlpha.value = 340
                        }
                    }
                }else{
                    if(shootingDirection === "left"){
                        makeCapture = false
                        if(computedCurrentAlpha.value >= computedPrevAlpha.value){
                            if(computedCurrentAlpha.value - computedPrevAlpha.value >= 20){
                                makeCapture = true

                                computedPrevAlpha.value += 20
                                //this.props.onReadyForShooting(result, this.capture)
                                takePhoto()
                                return
                            }
                        }
                    }else if(shootingDirection === "right"){
                        makeCapture = false
                        if(computedCurrentAlpha.value <= computedPrevAlpha.value){
                            if(computedPrevAlpha.value - computedCurrentAlpha.value >= 20){
                                makeCapture = true

                                computedPrevAlpha.value -= 20
                                //this.props.onReadyForShooting(result, this.capture)
                                takePhoto()
                                return
                            }
                        }
                    }

                    if((computedCurrentAlpha.value === 359 && shootingDirection === "left") ||
                        (computedCurrentAlpha.value === 0 && shootingDirection === "right")
                    ){ // Shooting ended
                        alert("SHOOTING SUCCESSFULLY ENDED")
                        computedCurrentAlpha.value = 0
                        computedPrevAlpha.value = 0
                        isShootingStarted = false
                        shootingDirection = ""
                        photoArray.value = []
                        currentAlpha = 0
                        prevAlpha = 0

                        //prepare our json object for sending
                        /*const roomData = {
                            registrationNo : Date(),
                            length : this.state.roomLength,
                            width : this.state.roomWidth,
                            height : this.state.roomHeight
                        }*/
                        //alert("BEFORE SENDING")
                        //this.props.onShootingEnded(roomData)
                    }
                }

                //this.props.onReadyForShooting(result, this.capture)
            }
        }
}
</script>