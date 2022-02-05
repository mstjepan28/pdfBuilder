<template>
    <div>
        <div v-if="element">
            <h2>Edit element</h2>

            <input type="text" class="textInput changeName" v-model="element.name">

            <h3>Position data</h3>

            <div class="positionData">
                <div class="elementX">                
                    <label for="positionDataX">X</label>
                    <input type="number" id="positionDataX" v-model="positionData.x">
                </div>

                <div class="elementY">
                    <label for="positionDataY">Y</label>
                    <input type="number" id="positionDataY" v-model="positionData.y">
                </div>

                <div class="ElementW">
                    <label for="positionDataWidth">W</label>
                    <input type="number" id="positionDataWidth" v-model="positionData.width">
                </div>

                <div class="elementH">
                    <label for="positionDataHeight">H</label>
                    <input type="number" id="positionDataHeight" v-model="positionData.height">
                </div>
            </div>

            <div class="toggleMovement">
                <ToggleSwitch
                    class="toggleWrapper"
                    ref="movementToggle"
                    :labels="['Movement enabled', 'Movement disabled']" 
                    :isDisabled="!isMovable" 
                    :toggleID="'movementToggle'"
                    @toggled="toggleMovement"
                />

                <ToggleSwitch
                    class="toggleWrapper"
                    ref="staticToggle"
                    :labels="['Static content', 'Static content']" 
                    :isDisabled="!element.isStatic" 
                    :toggleID="'staticToggle'"
                    @toggled="toggleStaticContent"
                />
            </div>

            <h2>Data type: </h2>
            <div class="dataTypeSelection">
                <label for="elementText" class="primaryButton" :class="{activeType: elementType == 'singlelineText'}"> 
                    Text <input type="radio" id="elementText" name="elementType" value="singlelineText" v-model="elementType">
                </label>
                <label for="elementParagraph" class="primaryButton" :class="{activeType: elementType == 'paragraph'}"> 
                    Paragraph <input type="radio" id="elementParagraph" name="elementType" value="paragraph" v-model="elementType">
                </label>
                <label for="elementImage" class="primaryButton" :class="{activeType: elementType == 'image'}"> 
                    Image <input type="radio" id="elementImage" name="elementType" value="image" v-model="elementType">
                </label>
            </div>

            <h2>Static content</h2>
            <div v-if="element.isStatic">
                <div v-if="element.type == 'singlelineText'" class="staticInput">
                    <h3><label for="singleLineTextInput">Input text: </label></h3>
                    <input id="singleLineTextInput" class="textInput" type="text" v-model="staticContent" placeholder="Write single line text here..."/>
                </div>
                <div v-if="element.type == 'paragraph'" class="staticInput">
                    <h3><label for="paragraphInput">Input text: </label></h3>
                    <textarea id="paragraphInput" class="paragraphInput" v-model="staticContent" placeholder="Write your paragraph here..."></textarea>
                </div>
                <div v-if="element.type == 'image'" class="staticInput">
                    <h3><label for="imageURLInput">Image URL: </label></h3>
                    <textarea id="imageURLInput" class="paragraphInput" v-model="staticContent" placeholder="Insert link to an image here..."></textarea>
                </div>
            </div>
            <div v-else class="variableDropdown">
                <h3><label for="variables">Choose a variable:</label></h3>
                <select id="variables" v-model="element.variable">
                    <option :key="variable.value" v-for="variable in templateVariables" :value="variable.value">{{variable.label}}</option>
                </select>
            </div>

            <DeleteButton 
                class="deleteButton"
                :confirmBeforeDelete="true"
                @delete="deleteElement"
            />
        </div>
        
        <div v-else class="nothingSelected">
            <h2>No element selected</h2>
        </div>

    </div>
</template>

<script>
import ImageUpload from "./imageUpload.vue";
import ToggleSwitch from "./ToggleSwitch.vue";
import DeleteButton from "./deleteButton.vue";

import axios from "axios";
import Vue from "vue"

export default {
    props:{
        apiUrl: {
            type: String,
            required: true
        },
        element: {
            type: Object,
            required: false
        }
    },
    components: { ImageUpload, ToggleSwitch, DeleteButton },
    data(){
        return{
            isMovable: null,
            positionData: {},

            staticContent: "",
            elementType: "",

            templateVariables: [
                {label: "ID", value: "id"},
                {label: "First name", value: "first_name"},
                {label: "Last name", value: "last_name"},
                {label: "Email", value: "email"}    
            ],
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
        async getVariables(){
            document.documentElement.style.cursor = "wait";

            try{
                const response = await axios.get(`${this.apiUrl}/variables`)
                this.formatVariables(response.data.variables);
            }catch(error){
                const status = error.message == "Network Error"? 408: 500;
                this.$emit("openResponse", status);
            }
            
            document.documentElement.style.cursor = "";
        },

        formatVariables(variableList){
            this.templateVariables = variableList.reduce((resultArray, variable) => {
                if(variable == "id") return resultArray;

                let label = variable.split("_")
                
                let temp = label[0]
                label[0] = temp.charAt(0).toUpperCase() + temp.slice(1)

                resultArray.push({
                    label: label.join(" "),
                    value: variable
                })

                return resultArray
            }, [])
        },

        // ************************************************************ //

        // If element has class 'draggable' then its draggable
        checkIfMovable(){
            if(!this.element.elementRef) return;
            const newMovementState = this.element.elementRef.classList.contains("draggable");

            // If the movement state has change, trigger the movement button to change its internal state
            if(this.isMovable != newMovementState) 
                this.$refs.movementToggle.changeState(true);

            this.isMovable = newMovementState
        },
        
        // Add/remove the 'draggable' class from element and in that way toggle its movement
        toggleMovement(){
            const elemRef = this.element.elementRef;
            this.isMovable = !this.isMovable;
            
            const toggle ={
                true: () => elemRef.classList.add("draggable"),
                false: () => elemRef.classList.remove("draggable")
            }
            toggle[this.isMovable]();
        },

        // ************************************************************ //

        // Validate inputted position data. If its not valid set the value to the upper/lower bound.
        validatePositionData(){
            Object.keys(this.positionData).forEach(key => {
                let value = parseInt(this.positionData[key])
                value = value < 0? 0: value;
                
                this.positionData[key] = value;
            });

            const  clamp = (num, min, max) => Math.min(Math.max(num, min), max);
            
            this.positionData.x = clamp(this.positionData.x, 0, this.xCordMax);
            this.positionData.y = clamp(this.positionData.y, 0, this.yCordMax);
            this.positionData.width = clamp(this.positionData.width, 0, this.widthDimMax);
            this.positionData.height = clamp(this.positionData.height, 0, this.heightDimMax);
        },
        
        modifyPositionData(modifyBy){
            if(!this.isMovable) return;
            Object.keys(modifyBy).forEach(key => this.positionData[key] += modifyBy[key]);
            this.validatePositionData();
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

        // ************************************************************ //

        toggleStaticContent(newElement=null){
            if(this.element.internalComponent) 
                this.destroyComponent();
            
            const internalState = this.$refs.staticToggle.toggleState

            if(this.element.isStatic == internalState) 
                return
            else if(newElement)
                this.$refs.staticToggle.changeState(true);
            else
                this.element.isStatic = internalState;

            if(this.elementType == "image" && this.element.isStatic) 
                this.elementTypeImage();
        },

        updateStaticContent(){
            this.element.staticContent = this.staticContent;

            if(this.elementType == "image") 
                return this.element.internalComponent.setImageURL(this.staticContent);

            this.element.elementRef.innerHTML = this.staticContent || "";
        },

        // ************************************************************ //

        // Dynamically create an instance of the ImageUpload component and mount it as a child
        //  of the selected element
        elementTypeImage(){
            if(!this.element.isStatic) return;

            const child = document.createElement("div");
            this.element.elementRef.appendChild(child)

            const imageUploadConstructor = Vue.extend(ImageUpload)
            const imageUploadInstance = new imageUploadConstructor({ propsData: { id: this.element.id } })

            this.element.internalComponent = imageUploadInstance.$mount(child);
        },

        // Destroy the internal component of the selected element and remove it from the DOM 
        destroyComponent(){
            document.getElementById(this.element.id).remove();
            
            this.element.internalComponent.$destroy();
            this.element.internalComponent = null;
        },

        // ************************************************************ //

        deleteElement(){
            this.$emit("deleteElement", this.element);
        },
    },
    async mounted(){
        await this.getVariables()
    },
    watch:{
        // When selected element gets changed, check its movable 
        element(){
            if(!this.element) return;

            this.newElementMounted = true;

            this.staticContent = this.element.staticContent;
            this.elementType = this.element.type;
            this.positionData = this.element.positionData;

            setTimeout(() => {
                this.newElementMounted = false;
                
                this.checkIfMovable();
                this.toggleStaticContent(true);
            }, 100)
        },

        elementType(){
            if(!this.elementType || this.newElementMounted) return;
            if(this.element.internalComponent) this.destroyComponent();

            this.element.type = this.elementType;

            this.staticContent = ""
            this.element.elementRef.innerHTML = ""

            const elementTypeHandler = {
                image: () => this.elementTypeImage(),
                singlelineText: () => this.element.elementRef.style.wordBreak = "keep-all",
                paragraph: () => this.element.elementRef.style.wordBreak = "break-all",
                "": () => {},
            }
            elementTypeHandler[this.elementType]();
        },

        staticContent(){
            this.updateStaticContent();
        },

        // Deep watcher for position values
        positionData:{
            handler(){
                this.updateElementPosition()
            },
            deep: true
        }
    },
}
</script>

<style lang="scss" scoped>
@import "./styles/style.scss";

.activeType{
    color: $primaryColor;
    background: $blueHighlight !important;
}

h2{
    margin-top: 1rem;
}
h3{
    margin-top: 1rem;
    margin-bottom: 0.5rem;
}

.changeName{
    font-size: 20px;
    font-weight: bold;

    padding: 1rem 0 0.25rem 0.25rem;

    background-color: transparent;

    &:focus{
        background-color: $secondaryColor;
    }
}

.positionData{
    width: 100%;

    display: grid;
    grid-template: repeat(2, 1fr);
    gap: 1rem;

    padding: 1rem;
    
    border-radius: 8px;
    background: $secondaryColor;
    
    .elementX { 
        grid-area: 1 / 1 / 2 / 2; 
    }
    .elementY { 
        grid-area: 1 / 2 / 2 / 3; 
    }
    .elementW { 
        grid-area: 2 / 1 / 3 / 2; 
    }
    .elementH { 
        grid-area: 2 / 2 / 3 / 3; 
    }

    &>div{
        display: flex;
        gap: 0.25rem;

        font-weight: bold;

        &>label{
            width: 1.5rem;
        }

        &>input{
            width: 100%;

            font-weight: bold;

            outline: none;
            border: none;
            border-bottom: 1px solid black ;
            background: none;
        }
    }
}

.toggleWrapper{
    @include flex(row, space-between, center);
    width: 100%;

    &:first-child{
        margin: 0.5rem 0;
    }
}

.dataTypeSelection{
    @include flex(row, space-around, center);
    column-gap: 0.25rem;
    margin-top: 0.5rem;

    &>label{
        @include flex(column, center, center);
        &>input{
            display: none !important;
        }
    }
}

.variableDropdown{
    select{
        width: 100%;

        font-size: 16px;
        font-weight: bold;

        padding: 0.25rem;

        border-radius: 8px;
        border: none;
        background: $secondaryColor;
        
        &>option:hover{
            background: $secondaryColor;
        }
    }
}

.deleteButton{
    margin-top: 2.5rem;
}

.nothingSelected{
    @include flex(row, center, center);
    width: 100%;

    padding: 6rem 0;
    margin-top: 2rem;

    border-radius: 8px;
    background: $secondaryColor;

    &>h2{
        margin: 0;
    }
}
</style>