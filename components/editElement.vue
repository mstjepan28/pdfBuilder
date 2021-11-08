<template>
    <div>
        <div v-if="element">
            <h2>Edit element</h2>
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

            <button @click="deleteElement(element.index)"> Delete </button>
        </div>
        
        <div v-else>
            <h4>No element selected</h4>
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
            positionData: {
                x: 0,
                y: 0,
                width: 0,
                height: 0,
            },
        }
    },
    computed:{
        /* ------------------------------------------------------------ */

        xCordMax(){
            const pdfTemplate = document.getElementById("pdfTemplate");
            const xCordMax = pdfTemplate.offsetWidth - this.positionData.width || 0;

            return  xCordMax < 0? 0: xCordMax; 
        },
        yCordMax(){
            const pdfTemplate = document.getElementById("pdfTemplate");
            const yCordMax = pdfTemplate.offsetHeight - this.positionData.height || 0;

            return  yCordMax < 0? 0: yCordMax; 
        },
        widthDimMax(){
            const pdfTemplate = document.getElementById("pdfTemplate");
            return pdfTemplate.offsetWidth - this.positionData.x || 0;
        },
        heightDimMax(){
            const pdfTemplate = document.getElementById("pdfTemplate");
            return pdfTemplate.offsetHeight - this.positionData.y || 0;
        },

        /* ------------------------------------------------------------ */
    },
    methods:{
        // If element has class 'draggable' then its draggable
        checkIfMovable(){
            if(!this.element.elementRef) return;
            this.isMovable = this.element.elementRef.classList.contains("draggable");
        },
        
        // Add/remove the 'draggable' class from element and in that way toggle its movement
        toggleMovement(){
            const elemRef = this.element.elementRef;
            
            const toggle ={
                true: () => elemRef.classList.add("draggable"),
                false: () => elemRef.classList.remove("draggable")
            }
            toggle[this.isMovable]();
        },

        // Validate inputed position data. If its not valid set the value to the upper/lower bound.
        validatePositionData(){
            Object.keys(this.positionData).forEach(key => {
                let value = parseInt(this.positionData[key])
                value = value < 0? 0: value;
                
                this.positionData[key] = value;
            });

            this.positionData.x = this.positionData.x > this.xCordMax? this.xCordMax: this.positionData.x;
            this.positionData.y = this.positionData.y > this.yCordMax? this.yCordMax: this.positionData.y;
            this.positionData.width = this.positionData.width > this.widthDimMax? this.widthDimMax: this.positionData.width;
            this.positionData.height = this.positionData.height > this.heightDimMax? this.heightDimMax: this.positionData.height;
        },

        // Update elements position based on the input values
        updateElementPosition(){
            if(!this.element || !this.element.positionData) return;

            this.validatePositionData();
            const element = this.element.elementRef;

            element.style.width = this.positionData.width + "px";
            element.style.height = this.positionData.height + "px";
            element.style.transform = 'translate(' + this.positionData.x + 'px,' + this.positionData.y + 'px)';
        
            element.dataset.x = this.positionData.x;
            element.dataset.y = this.positionData.y;
        },

        deleteElement(){
            this.$emit("deleteElement", this.element);
        }
    },
    watch:{
        // When selected element gets changed, check its movable 
        element(){
            if(!this.element) return;

            this.checkIfMovable();
            this.positionData = this.element.positionData;
        },

        isMovable(){
            this.toggleMovement()
        },

        // Deep watcher for position values
        positionData:{
            handler(){
                this.updateElementPosition()
            },
            deep: true
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