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

            <div class="elementPositioning">
                <button @click="positionElement('xStart')">X Start</button>
                <button @click="positionElement('yStart')">Y Start</button>
                <button @click="positionElement('xCenter')">X Center</button>
                <button @click="positionElement('yCenter')">Y Center</button>
                <button @click="positionElement('xEnd')">X End</button>
                <button @click="positionElement('yEnd')">Y End</button>
            </div>

            <div class="toggleMovement">
                <label for="moveToggle">Element can be moved:</label>
                <input id="moveToggle" type="checkbox" v-model="isMovable"/>
            </div>

            <h4>Static content</h4>
            
            <div>
                <label for="elementType">Select the element type: </label>
                <select id="elementType" v-model="elementType">
                    <option :key="type.value" v-for="type in contentType" :value="type.value">{{type.label}}</option>
                </select>
            </div>

            <div>
                <label for="isStatic">Is static value</label>
                <input id="isStatic" type="checkbox" v-model="element.isStatic"/>
            </div>

            <div v-if="element.isStatic">
                <div v-if="element.type == 'singlelineText'" class="staticInput">
                    <label for="singleLineTextInput">Input text: </label>
                    <input id="singleLineTextInput" type="text" v-model="staticContent"/>
                </div>
                <div v-if="element.type == 'paragraph'" class="staticInput">
                    <label for="paragraphInput">Input text: </label>
                    <textarea id="paragraphInput" v-model="staticContent"></textarea>
                </div>
                <div v-if="element.type == 'image'" class="staticInput">
                    <label for="imageURLInput">Image URL: </label>
                    <textarea id="imageURLInput" v-model="staticContent"></textarea>
                </div>
            </div>
            <div v-else>
                <label for="variables">Choose a variable:</label>
                <select id="variables" v-model="element.variable">
                    <option :key="variable.value" v-for="variable in templateVariables" :value="variable.value">{{variable.label}}</option>
                </select>
            </div>

            <button @click="deleteElement(element.index)"> Delete </button>
        </div>
        
        <div v-else>
            <h4>No element selected</h4>
        </div>

    </div>
</template>

<script>
import ImageUpload from "./imageUpload.vue"
import axios from "axios";
import Vue from "vue"

export default {
    props:{
        apiUrl: String,
        element: Object
    },
    components: { ImageUpload },
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

            contentType: [
                {label: "Image", value: "image"},
                {label: "Text", value: "singlelineText"},
                {label: "Paragraf", value: "paragraph"},
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
            try{
                const responce = await axios.get(`${this.apiUrl}/variables`)
                this.formatVariables(responce.data.variables);
            }catch(error){
                console.log(error)
            }
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

        // Position selected element based on the button press
        positionElement(position){
            const positionData = this.positionData;
            const pdfTemplate = document.querySelector("div.pdfTemplate")

            const positionElements = {
                xStart: () => positionData.x = 0,
                yStart: () => positionData.y = 0,

                xCenter: () => positionData.x = (pdfTemplate.offsetWidth - positionData.width) / 2,
                yCenter: () => positionData.y = (pdfTemplate.offsetHeight - positionData.height) / 2,

                xEnd: () => positionData.x = this.xCordMax,
                yEnd: () => positionData.y = this.yCordMax,
            }
            positionElements[position]();

            this.updateElementPosition();
        },

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
        },

        updateStaticContent(){
            this.element.staticContent = this.staticContent;

            if(this.elementType == "image") return this.element.internalComponent.setImageURL(this.staticContent);

            this.element.elementRef.innerHTML = this.staticContent || "";
        },

        // Dynamically create an instance of the ImageUpload component and mount it as a child
        //  of the selected element
        elementTypeImage(){
            const child = document.createElement("div");
            this.element.elementRef.appendChild(child)

            const imageUploadConstructor = Vue.extend(ImageUpload)
            const imageUploadInstance = new imageUploadConstructor({})

            this.element.internalComponent = imageUploadInstance.$mount(child);
        },

        // Destroy the internal component of the selected element and remove it from the DOM 
        destroyComponent(){
            document.querySelector("div.internalComponent").remove();
            
            this.element.internalComponent.$destroy();
            this.element.internalComponent = null;
        },
    },
    async mounted(){
        await this.getVariables()
    },
    watch:{
        // When selected element gets changed, check its movable 
        element(){
            if(!this.element) return;

            this.checkIfMovable();

            this.staticContent = this.element.staticContent;
            this.elementType = this.element.type;
            this.positionData = this.element.positionData;

            this.newElementMounted = true;
            setTimeout(() => this.newElementMounted = false, 100)
        },

        isMovable(){
            this.toggleMovement()
        },

        elementType(){
            if(!this.elementType || this.newElementMounted) return; 
            if(this.element.internalComponent) this.destroyComponent();

            this.element.type = this.elementType;

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
@import "../assets/style.scss";

.positionData{
    @include flex(column, initial, initial);
}



.staticInput{
    label{
        display: block;
    }
    textarea {
        width: 100%;
        min-height: 8rem;
        resize: none;
    }
}
</style>