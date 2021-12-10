<template>
  	<div class="modalBackground" @click="closeResponceModal()">
        <div class="modalContainer" @click.stop>
            <div class="statusIndicatorContainer">
                <div class="statusIndicator">
                    <div class="fail">
                        <img src="./svg/FailIndicator.svg" alt="Fail indicator" :width="svgSize" :height="svgSize">
                    </div>
                    
                    <div class="fillElement"></div>

                    <div class="pending">
                        <LoadingIndicator/>
                    </div>

                    <div class="success">
                        <img src="./svg/SuccessIndicator.svg" alt="Success indicator" :width="svgSize" :height="svgSize">
                    </div>
                </div>
            </div>

            <div class="controls">
                <button @click="rotateStatusIndicator('success')"> Rotate success </button>
                <button @click="rotateStatusIndicator('pending')"> Rotate pending </button>
                <button @click="rotateStatusIndicator('fail')"> Rotate fail </button>
            </div>
        </div>
    </div>
</template>

<script>

import LoadingIndicator from "./LoadingIndicator.vue"

export default {
    props:{
        resultStatus: String
    },
    components: { LoadingIndicator },
    data(){
        return{
            svgSize: "150px"
        }
    },
    methods:{
        closeResponceModal(){
            const modal = document.querySelector(".modalBackground");
            modal.style.display = "none";
        },

        rotateStatusIndicator(rotate){
            const rotationLookup = {
                success: 225,
                pending: 135,
                fail: 45
            }
            //const roationAngle = rotationLookup[this.resultStatus];
            const roationAngle = rotationLookup[rotate];

            document.querySelectorAll(".statusIndicator>div").forEach(status => {
                status.style.transform = `rotate(-${roationAngle}deg)`;
                //status.style.visibility = status.classList.contains(this.resultStatus)? "visible": "hidden";
                status.style.visibility = status.classList.contains(rotate)? "visible": "hidden";
            });

            const statusIndicator = document.querySelector(".statusIndicator");
            statusIndicator.style.transform = `rotate(${roationAngle}deg)`;
        },
    },
    watch:{
        resultStatus(){
            this.handleStatusChange();
        }
    }
}
</script>

<style lang="scss" scoped>
@import "../assets/style.scss";

.modalBackground{
    width: 100vw;
    height: 100vh;

    display: none;
    align-items: center;
    justify-content: center;

    position: absolute;
    z-index: 9999;

    background-color: rgba(128,128,128,0.75);
    
    animation: fadeIn 0.4s linear;
}

.modalContainer{
    width: 750px;
    min-height: 400px;

    display: flex;
    flex-direction: column;
    align-items: center;

    border-radius: 8px;
    background: white;

    animation: scaleUp 0.4s linear;
}

.statusIndicatorContainer{
    $transitionSpeed: 0.75s;

    width: 100%;
    max-height: 250px;

    display: flex;
    justify-content: center;

    overflow: hidden;

    .statusIndicator{
        $indicatorContainerSize: 550px;
    
        width: $indicatorContainerSize;
        height: $indicatorContainerSize;
    
        display: flex;
        flex-wrap: wrap;

        position: relative;
        bottom: -50px;

        transition: $transitionSpeed;
        transform: rotate(135deg);
    
        &>div{
            $boxSize: $indicatorContainerSize / 2;
    
            width: $boxSize;
            height: $boxSize;
    
            visibility: hidden;

            display: flex;
            justify-content: center;
            align-items: center;

            transition: $transitionSpeed;
            transform: rotate(-135deg);
        }
        &>.pending{
            visibility: visible;
        }
    }
}

@keyframes fadeIn{
	from {
        opacity: 0
    } 
	to {
        opacity: 1
    }
}

@keyframes scaleUp{
    0%{
        transform:scale(.5);
    }
    100%{
        transform:scale(1);
    }
}
.controls{
    width: 100%;
}

</style>
