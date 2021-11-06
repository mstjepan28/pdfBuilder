<template>
    <div>
        <h2>Edit element</h2>
        <div v-if="element.type == 'selection'">
            <h4>Position data</h4>
            
            <div class="positionData">
                X: <input type="number" v-model="positionData.x">
                Y: <input type="number" v-model="positionData.y">

                Width: <input type="number" v-model="positionData.width">
                Height: <input type="number" v-model="positionData.height">
            </div>

            <div class="toggleMovement">
                <label for="moveToggle">Element can be moved:</label>
                <input id="moveToggle" type="checkbox" v-model="isMovable"/>
            </div>
        
        </div>
        <div v-else>
            
        </div>
    </div>
</template>

<script>
export default {
    props:{
        element: Object
    },
    data(){
        return{
            isMovable: null,
            positionData: {},

        }
    },
    methods:{
        checkIfMovable(){
            if(!this.element.elementRef) return;
            this.isMovable = this.element.elementRef.classList.contains("draggable");
        },
        
        toggleMovement(){
            const elemRef = this.element.elementRef;
            
            const toggle ={
                true: () => elemRef.classList.add("draggable"),
                false: () => elemRef.classList.remove("draggable")
            }
            toggle[this.isMovable]();
        },


    },
    watch:{
        element(){
            this.checkIfMovable();
            this.positionData = this.element.positionData;
        },
        isMovable(){
            this.toggleMovement()
        },
        positionData(){
            Object.keys(this.positionData).forEach(key => this.positionData[key] = parseInt(this.positionData[key]))
        }
    }
}
</script>

<style lang="scss" scoped>
@import "../assets/style.scss";

.positionData{
    @include flex(column, initial, initial);
}

</style>