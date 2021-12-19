<template>
    <button type="button"  @click="toggle()">
        <h3 v-if="labels" class="switchLabel" :class="{activeColor: toggleState}">{{toggleLabel}}</h3>

        <div class="switch" :class="{activeBackground: toggleState}">
            <span class="circle" :id="'circle' + toggleID"></span>
        </div>
    </button>
</template>

<script>
export default {
    props: {
        labels: Array,
        toggleID: String,
        isDisabled: Boolean,
    },
    data(){
        return{
            toggleState: false,
            toggleEnabled: false,
        }
    },
    computed:{
        toggleLabel(){
            return this.toggleState? this.labels[0]: this.labels[1];
        },
    },
    methods:{
        toggle(){
            if(!this.toggleEnabled) return;
            
            this.disableToggle();
            this.changeState();

            this.$emit("toggled");
        },

        // Disable the toggle for 0.4s / duration of the animation
        disableToggle(){
            this.toggleEnabled = false;
            setTimeout(() => this.toggleEnabled = true, 400);
        },
        
        // Change the visuals and internal state
        changeState(automaticTrigger=false){
            const toggleSwitch  = document.getElementById("circle" + this.toggleID);
            if(automaticTrigger) this.toggleTransition(toggleSwitch);
            
            this.toggleState = !this.toggleState;
            toggleSwitch.style.left = this.toggleState? "18px": "3px";
        },

        toggleTransition(toggleSwitch){
            toggleSwitch.style.transition = "none";
            setTimeout(() => toggleSwitch.style.transition = "", 40000)
        }
    },
    // If the initial state is 'enabled' trigger the toggle function
    mounted(){
        if(!this.isDisabled) this.changeState(true);
        this.toggleEnabled = true;
    }
}
</script>

<style lang="scss" scoped>
// **** Change colors here **** //
@import "./styles/style.scss";

$primary: $primaryColor;
$secondary: $secondaryColor;
$active: $blueHighlight;
$disabled: $highlightColor;
// *************************** //

$SliderSize: 15px;

$SwitchWidth: 36px;
$SwitchHeight: 20px;

button, button:hover, button:focus{
    border: none;
    outline: none;
    background: none;

    cursor: pointer;
}

.activeBackground{
    background: $active !important;
}
.activeColor{
    color: $active !important;
}

.switch{
    width: $SwitchWidth;
    height: $SwitchHeight;

    position: relative;

    border-radius: 9999px;

    background: $disabled;
}

.circle{
    $top: 50%;
    $left: 3px;

    width: $SliderSize;
    height: $SliderSize;

    position: absolute;
    top: $top;
    left: $left;

    transition: .4s;
    transform: translateY(-$top);

    border-radius: 9999px;

    background: $primary;
}

.switchLabel{
    color: $disabled;
    margin-right: 1rem;
}

</style>