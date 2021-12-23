<template>
    <div class="container">
        <ResponseModal
            ref="responseModal"
        />

        <div class="elementsCol">
            <div class="colContent">            
                <button class="primaryButton" @click="drawSelection()" :class="{activeDrawing: drawHandler}">
                    {{drawButtonText}}
                </button>
                
                <ConvertPdfBtn 
                    :apiUrl="apiUrl" 
                    :selectionList="selectionList" 
                    :pdfTemplate="pdfTemplate" 
                    :pdfDimensions="pdfDimensions"
                    @converterResponse="converterResponse"
                />

                <PdfToImage 
                    :apiUrl="apiUrl" 
                    @pdfUploaded="setPdfTemplate"
                />

                <label for="zoomValue">Zoom value: </label>
                <input id="zoomValue" type="range" min="1" max="200" v-model="zoomValue">
            </div>

            <div class="toggleColumn">
                <button @click="toggleColumn('elementsCol')"> 
                    <img src="./svg/ChevronArrow.svg" alt="Chevron arrow" style="transform: rotate(90deg)"> 
                </button>
            </div>
        </div>

        <div class="templateCol">
            <div id="pdfTemplate" class="pdfTemplate"></div>
        </div>

        <div class="informationCol">
            <div class="toggleColumn">
                <button @click="toggleColumn('informationCol')"> 
                    <img src="./svg/ChevronArrow.svg" alt="Chevron arrow" style="transform: rotate(270deg)"> 
                </button>
            </div>

            <div class="colContent">
                <ElementList 
                    title="Selection list: "
                    :elementList="selectionList"
                    @elementSelected="elementSelected"
                />
                
                <EditElement
                    ref="editElement"
                    :apiUrl="apiUrl" 
                    :element="selectedElement" 
                    @deleteElement="updateList"
                />
            </div>
        </div>
    </div>
</template>

<script>
import ElementList   from "./elementList.vue";
import EditElement   from "./editElement.vue";
import ConvertPdfBtn from "./convertPdfBtn.vue"
import PdfToImage    from "./pdfToImage.vue";
import ResponseModal from "./responseModal.vue";

import interact      from "interactjs";

export default {
    props:{
        apiUrl: {
            type: String,
            required: true
        }
    },
    components: { ElementList, EditElement, ConvertPdfBtn, PdfToImage, ResponseModal },
    data(){
        return{
            drawHandler: null,

            selectedElement: null,
            selectionList: [],

            pdfTemplate: null,
            pdfDimensions: null,

            zoomValue: 100,
        }
    },
    computed:{
        drawButtonText(){
            return this.drawHandler? "Disable drawing": "Enable drawing"
        }
    },
    methods:{
        /* ------------------------------------------------------------ */

        interaction(){
            const thisRef = this;

            interact('.draggable')
            .resizable({
                inertia: false,
                
                // resize from all edges and corners
                edges: { left: true, right: true, bottom: true, top: true },

                modifiers: [
                    // Element cannot grow outside of the parent
                    interact.modifiers.restrictEdges({ outer: 'parent' }),
                ],
                
                listeners: {
                    move: (event) => thisRef.resizeHandler(event)
                },
            })
            .draggable({
                inertia: false,
                
                modifiers: [
                    interact.modifiers.snap({
                        targets: [
                            // Snap elements in place when dragged
                            interact.snappers.grid({ x: 10, y: 10 })
                        ],
                        range: Infinity,
                        relativePoints: [ { x: 0, y: 0 } ]
                    }),

                    // Keeps the element inside the parent
                    interact.modifiers.restrictRect({ restriction: 'parent' })
                ],

                listeners: {
                    move: (event) => thisRef.dragMoveListener(event),
                },

            })
        },
        
        dragMoveListener (event) {
            const target = event.target;
            
            // keep the dragged position in the data-x/data-y attributes
            const x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx
            const y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy

            // update position data of object
            this.updatePositionData(target, {x, y})
            target.style.transform = `translate(${x}px, ${y}px)`

            // update the position attributes
            target.setAttribute('data-x', x)
            target.setAttribute('data-y', y)
        },

        resizeHandler(event) {
            const target = event.target;

            // Get the previous coordinates of the object
            let x = (parseFloat(target.getAttribute('data-x')) || 0)
            let y = (parseFloat(target.getAttribute('data-y')) || 0)

            // update the element's style
            const width = event.rect.width;
            const height = event.rect.height;

            this.updatePositionData(target, { width, height })

            target.style.width = width + 'px';
            target.style.height = height + 'px';

            // translate when resizing from top or left edges
            x += event.deltaRect.left
            y += event.deltaRect.top

            target.style.transform = `translate(${x}px, ${y}px)`

            target.setAttribute('data-x', x)
            target.setAttribute('data-y', y)
        },

        /* ------------------------------------------------------------ */
        disableDrawing(){
            const pdfTemplate = document.getElementById("pdfTemplate");

            pdfTemplate.removeEventListener("click", this.drawHandler);
            pdfTemplate.style.cursor = "";
            
            this.togglePointerEvents(pdfTemplate, true);
            this.drawHandler = null;
        },

        drawSelection(){
            if(this.drawHandler) return this.disableDrawing();

            let positionData = { x: 0, y: 0, width: 0, height: 0 };
            
            let selection = null;
            let drawingStarted = false;

            const thisRef = this;
            const pdfTemplate = document.getElementById("pdfTemplate");

            /* ------------------------------------------------------------ */

            function accountForZoom(value){
                return Math.trunc(value / (thisRef.zoomValue / 100))
            }

            function updateSelectionPosition(event){
                let width  = accountForZoom(event.offsetX) - positionData.x
                let height = accountForZoom(event.offsetY) - positionData.y
                
                selection.style.left = width < 0? `${width}px` : "";
                selection.style.top = height < 0? `${height}px`: "";

                positionData.width = Math.abs(width);
                positionData.height = Math.abs(height);
                
                selection.style.width  = `${positionData.width}px`;
                selection.style.height = `${positionData.height}px`;
            }

            this.drawHandler = function (event){
                if(drawingStarted){
                    drawingStarted = false;

                    thisRef.saveNewSelection(selection, positionData);
                    positionData = { x: 0, y: 0, width: 0, height: 0 };

                    pdfTemplate.removeEventListener("mousemove", updateSelectionPosition);
                }
                else{
                    drawingStarted = true;
                    
                    positionData.x = accountForZoom(event.offsetX);
                    positionData.y = accountForZoom(event.offsetY);

                    selection = thisRef.createSelection(positionData);
    
                    pdfTemplate.addEventListener('mousemove',  updateSelectionPosition);
                }
            }
            
            /* ------------------------------------------------------------ */

            this.togglePointerEvents(pdfTemplate, false);
            pdfTemplate.style.cursor = "crosshair"

            pdfTemplate.addEventListener("click", this.drawHandler)

            /* ------------------------------------------------------------ */
        },

        // parentElement - html element who's children's pointer events will be toggled
        // boolValue - true(active pointer events) or false(inactive pointer events)
        // Used to make a selection over/inside other elements inside parent
        togglePointerEvents(parentElement, boolValue){
            Array.from(parentElement.children).forEach(child => child.style.pointerEvents = boolValue? "auto": "none")
        },

        // positionData - { x: num, y: num, width: num, height: num}
        // Creates a new div, sets its position to the given coordinates inside parent, sets 
        //  its style and adds it to the dom  
        createSelection(positionData){
            const newSelection = document.createElement("div");
            
            this.setCoordinates(newSelection, positionData);

            newSelection.style.position = "absolute";
            newSelection.style.overflow = "hidden";
            newSelection.style.pointerEvents = "none"; // needed when shrinking the selection
            newSelection.style.backgroundColor = "rgba(173,216,230, 0.25)";

            document.getElementById("pdfTemplate").appendChild(newSelection);

            this.shiftFocus(newSelection)

            return newSelection;
        },

        // selection - html element
        // positionData - { x: num, y: num, width: num, height: num}
        // Set the data-index to the last element of the list and push the selection to the selection list
        saveNewSelection(selection, positionData){
            selection.classList.add("draggable", "selection");

            this.readjustPosition(selection, positionData);

            this.selectionList.push({
                id: Date.now() + "" + Math.ceil(Math.random() * 1000),
                name: "Selection #" + this.selectionList.length,
                elementRef: selection,
                positionData: positionData,

                type: "singlelineText",
                variable: null,
                isStatic: true,
                staticContent: "",

                internalComponent: null,
            })

            const element = this.getElementFromList(selection);
            this.elementSelected(element);
        },

        // element - html element
        // positionData - { x: num, y: num, width: num, height: num}
        // Calculate the top and left offset into x and y coordinates and update coordinates
        readjustPosition(element, positionData){
            const top = this.getNumValue(element.style.top);
            const left = this.getNumValue(element.style.left);
            const translate = this.getNumValue(element.style.transform, true);

            positionData.x = translate[0] - left;
            positionData.y = translate[1] - top;

            this.setCoordinates(element, positionData)

            element.style.top = ""
            element.style.left = ""
        },

        // element - html element
        // positionData - { x: num, y: num, width: num, height: num}
        // Set the coordinates of a element
        setCoordinates(element, positionData){
            element.dataset.x = positionData.x;
            element.dataset.y = positionData.y;
            
            element.style.transform = `translate(${positionData.x}px, ${positionData.y}px)`;  
        },

        /* ------------------------------------------------------------ */

        // element - js object to remove from dom/list
        // Remove element from dom and from the list. Update the index of each element in the list and because the
        //  element can only be deleted once its selected, set the selected element to null
        updateList(element){
            document.getElementById("pdfTemplate").removeChild(element.elementRef);
            this.selectionList = this.selectionList.filter(elem => elem != element);

            this.selectedElement = null;
        },

        getElementFromList(element){
            return this.selectionList.filter(elem => elem.elementRef == element)[0];
        },

        // event - click on event
        // click on event handler, if clicked on a selection, open selection in editor, else empty 
        //  selection from editor
        selectElementOnClick(event){
            if(this.drawHandler) return;
            let element = event.target
            
            if(element.classList.contains("internalComponent")) element = element.parentNode

            const isSelection = element.classList.contains("selection")
            if(!isSelection){
                if(this.selectedElement) this.selectedElement.elementRef.style.boxShadow = "";
                this.selectedElement = null;

                return;
            };

            element = this.getElementFromList(element);
            this.elementSelected(element);
        },

        // element - html element
        // switch out old selected element with new one
        elementSelected(element){  
            if(this.selectedElement) 
                this.selectedElement.elementRef.style.boxShadow = ""
            
            this.selectedElement = element;
            this.selectedElement.elementRef.style.boxShadow = "inset 0px 0px 0px 2px #add8e6"

            this.shiftFocus(element.elementRef)
        },

        // targetElement - html element
        // Sets the zIndex of the target element to be highest in the parent. 
        //  zIndexes of other elements get decreased by 1(lower boundary on 0)
        shiftFocus(element){
            const elementList = document.querySelectorAll("div.pdfTemplate>div");

            Array.from(elementList).forEach(elem => {
                const zIndex = elem.style.zIndex || 0;
                elem.style.zIndex = (zIndex - 1) < 0? 0: (zIndex - 1);
            })

            const topZIndex = elementList.length;
            element.style.zIndex = topZIndex;
        },

        // elem - html element
        // newData - either (x ,y) or (width, height)
        // Get the element from the list based on the data-index attribute. Replace the old (x ,y) or (width, height) with new ones
        updatePositionData(element, newData){
            const elementObj = this.getElementFromList(element)
            Object.keys(newData).forEach(key => elementObj.positionData[key] = newData[key])
        },

        setPdfTemplate(pdfTemplate, pdfDimensions){
            this.pdfTemplate = pdfTemplate
            this.pdfDimensions = pdfDimensions
        },

        converterResponse(resultStatus){
            this.$refs.responseModal.setStatus(resultStatus);
        },

        modifyPositionData(modifyBy){
            if(!this.selectedElement) return;
            this.$refs.editElement.modifyPositionData(modifyBy)
        },

        keyboardSupport(event){
            const modifyBy = 10;

            if(event.shiftKey){
                switch(event.key){
                    case "ArrowUp":
                        this.modifyPositionData({x: 0, y: 0, width: 0, height: -modifyBy});
                        break;
                    case "ArrowDown":
                        this.modifyPositionData({x: 0, y: 0, width: 0, height: modifyBy});
                        break;
                    case "ArrowLeft":
                        this.modifyPositionData({x: 0, y: 0, width: -modifyBy, height: 0});
                        break;
                    case "ArrowRight":
                        this.modifyPositionData({x: 0, y: 0, width: modifyBy, height: 0});
                        break;
                }

                return;
            }

            switch(event.key){
                case "Escape":
                    this.disableDrawing();
                    break;
                case "ArrowUp":
                    this.modifyPositionData({x: 0, y: -modifyBy, width: 0, height: 0});
                    break;
                case "ArrowDown":
                    this.modifyPositionData({x: 0, y: modifyBy, width: 0, height: 0});
                    break;
                case "ArrowLeft":
                    this.modifyPositionData({x: -modifyBy, y: 0, width: 0, height: 0});
                    break;
                case "ArrowRight":
                    this.modifyPositionData({x: modifyBy, y: 0, width: 0, height: 0});
                    break;
            }
        },

        toggleColumn(columnSelector){
            const column = document.querySelector(`div.${columnSelector}`)
            if(column.classList.contains("shrinkElement")) 
                column.classList.remove("shrinkElement")
            else 
                column.classList.add("shrinkElement")


            const chevronArrow = document.querySelector(`div.${columnSelector} img`); 
            const curRotation = this.getNumValue(chevronArrow.style.transform) // Get cur rotation
            chevronArrow.style.transform = `rotate(${(curRotation + 180) % 360}deg)`; // Add 180 deg to make the arrow point the other way
        },

        // property - String
        // returnArray - Boolean
        // Looks for numbers in string and returns them, if returnArray is true it returns an 
        //  array of numbers else it returns every value concatenated together
        getNumValue(stringValue, returnArray=false){
            if(returnArray){
                return stringValue.match(/\d*/g).reduce((result, value) => {
                    if(value) result.push(parseInt(value))
                    return result
                }, [])
            }
            return parseInt(stringValue.match(/\d*/g).join("")) || 0
        }
    },
    mounted(){
        this.interaction();

        document.addEventListener('keydown', this.keyboardSupport);
        document.getElementById("pdfTemplate").addEventListener("click", this.selectElementOnClick)
    },
    beforeDestroy(){
        document.removeEventListener('keydown', this.keyboardSupport);
        document.removeEventListener("click", this.selectElementOnClick);
    },
    watch:{
        zoomValue(){
            const pdfTemplate = document.getElementById("pdfTemplate");
            pdfTemplate.style.zoom = `${this.zoomValue}%`;
        }
    },
}
</script>

<style lang="scss" scoped>
@import "./styles/style.scss";

.selected{
    box-shadow: inset 0px 0px 0px 10px #add8e6;
}

.activeDrawing{
    color: $secondaryColor;
    background: $highlightColor;
}

.container{
    @include flex(row, center, stretch);
    width: 100%;

    overflow: hidden;
    background-color: $primaryColor;
}

.templateCol{
    @include section(auto, $primaryColor);
    @include flex(row, center, center);

    flex-grow: 1;

    overflow: auto;
    
    & > .pdfTemplate{
        @include boxShadow();

        min-width: calc(210mm * 1);
        aspect-ratio: 1/1.414;

        position: relative;

        background-color: $primaryColor;
        background-position: center;
        background-repeat: no-repeat; 
        background-size: cover;
    }
}

.informationCol, .elementsCol{
    $transition: 0.4s ease-out;

    @include section(30%, $primaryColor);
    @include flex(row, initial, initial);
    
    min-width: 18.5rem;
    max-width: 22.5rem;

    transition: $transition;

    &>.toggleColumn{
        @include flex(row, center, stretch);
        width: 2.5rem;
        min-width: 2.5rem;
        
        &>button{
            @include toggleColButton;
        }
    }
    &>.colContent{
        width: 20rem;
        min-width: 16rem;
        padding: 0.5rem;

        overflow: hidden;
    }

    & img{
        padding: 0.25rem;
        transition: $transition;
    }
}

.elementsCol > .colContent{
    & > *{
        margin-bottom: 1rem;
    }
}

.shrinkElement{
    width: 2.5rem;
    min-width: 0rem;

    &>.colContent{
        width: 0;
        min-width: 0;
        padding: 0;
    }

    &.informationCol>.colContent{
        transition: 4s;
    }
    &.elementsCol>.colContent{
        transition: 0.4s;
    }
}

</style>