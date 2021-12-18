<template>
    <div class="container">
        <ResponseModal 
            :statusCode="resultStatus"
        />

        <div class="elementsCol">
            <div class="colContent">            
                <button @click="makeSelection()">Make selection</button>
                
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

                <input type="range" min="1" max="200" v-model="zoomValue">
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
                    :title="'Selection list'" 
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
        apiUrl: String
    },
    components: { ElementList, EditElement, ConvertPdfBtn, PdfToImage, ResponseModal },
    data(){
        return{
            isSelectionFinished: true,
            selectedElement: null,
            selectionList: [],

            pdfTemplate: null,
            pdfDimensions: null,

            resultStatus: null,

            zoomValue: 100, 
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
            .on('tap', (event) => {
                console.log(event)
                thisRef.selectElementOnClick(event.target)
                thisRef.shiftFocus(event.target)
            })
        },
        
        dragMoveListener (event) {
            const target = event.target;
            
            // keep the dragged position in the data-x/data-y attributes
            const x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx
            const y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy

            // update position data of object
            this.updatePositionData(target, { x, y })

            target.style.transform = 'translate(' + x + 'px, ' + y + 'px)'

            // update the position attributes
            target.setAttribute('data-x', x)
            target.setAttribute('data-y', y)
        },

        resizeHandler(event) {
            const target = event.target

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

            target.style.transform = 'translate(' + x + 'px,' + y + 'px)'

            target.setAttribute('data-x', x)
            target.setAttribute('data-y', y)
        },

        /* ------------------------------------------------------------ */
        
        // targetElement - html element
        // Sets the zIndex of the target element to be highest in the parent. 
        //  zIndexes of other elements get decreased by 1(lower boundary on 0)
        shiftFocus(targetElement){
            const elementList = document.querySelectorAll("div.pdfTemplate>div");

            Array.from(elementList).forEach(element => {
                let zIndex = element.style.zIndex || 0;
                element.style.zIndex = (zIndex - 1) < 0? 0: (zIndex - 1);
            })

            const topZIndex = elementList.length;
            targetElement.style.zIndex = topZIndex;
        },

        makeSelection(){
            if(!this.isSelectionFinished) return;
            this.isSelectionFinished = false;

            const updateSelection = this.updateSelection;

            function getCoordinates(event){
                const rect = event.target.getBoundingClientRect();

                coordinates = {
                    x: Math.trunc(event.clientX - rect.left),
                    y: Math.trunc(event.clientY - rect.top)
                }
            }
            function getDimensions(event){
                const rect = event.target.getBoundingClientRect();

                const width = Math.trunc(event.clientX - rect.left) - coordinates.x;
                const height = Math.trunc(event.clientY - rect.top) - coordinates.y;

                dimensions = {
                    "width": width,
                    "height": height
                }
                updateSelection(selection, dimensions)
            }
            
            const pdfTemplate = document.getElementById("pdfTemplate");
            
            let coordinates = { x: 0, y: 0 };
            let dimensions = { width: 0, height: 0 };
            let selection = "";

            const controller = new AbortController(); 
            let selectionStarted = false;

            this.togglePointerEvents(pdfTemplate, false);

            pdfTemplate.addEventListener("click", (event) => {
                if(selectionStarted){
                    pdfTemplate.removeEventListener("mousemove", getDimensions);
                    selection.classList.add("draggable", "selection");

                    this.togglePointerEvents(pdfTemplate, true);
                    this.saveNewSelection(selection, {...coordinates, ...dimensions})
                    this.isSelectionFinished = true;

                    controller.abort();
                    return 
                }
                selectionStarted = true;
                
                getCoordinates(event);
                selection = this.createSelection(coordinates);

                pdfTemplate.addEventListener('mousemove',  getDimensions);
            }, { signal: controller.signal })
        },

        // parentElement - html element who's children's pointer events will be toggled
        // boolValue - true(active pointer events) or false(inactive pointer events)
        // Used to make a selection over/inside other elements inside parent
        togglePointerEvents(parentElement, boolValue){
            Array.from(parentElement.children).forEach(child => child.style.pointerEvents = boolValue? "auto": "none")
        },

        // coordinates - { x: num, y: num }
        // Creates a new div, sets its position to the given coordinates inside parent, sets 
        //  its style and adds it to the dom  
        createSelection(coordinates){
            const newSelection = document.createElement("div");
            
            newSelection.dataset.x = coordinates.x;
            newSelection.dataset.y = coordinates.y;
            
            newSelection.style.transform = "translate(" + coordinates.x + "px, " + coordinates.y + "px)";

            newSelection.style.position = "absolute";
            newSelection.style.overflow = "hidden";
            newSelection.style.backgroundColor = "rgba(173,216,230, 0.25)";

            document.getElementById("pdfTemplate").appendChild(newSelection);

            this.shiftFocus(newSelection)

            return newSelection;
        },

        // selection - html element
        // dimensions - { width: num, height: num }
        // Updates the width and height of the given selection
        updateSelection(selection, dimensions){
            selection.style.width = dimensions.width + "px";
            selection.style.height = dimensions.height + "px";
        },

        // selection - html element
        // positionData - { x: num, y: num, width: num, height: num}
        // Set the data-index to the last element of the list and push the selection to the selection list
        saveNewSelection(selection, positionData){
            selection.dataset.index = this.selectionList.length;

            this.selectionList.push({
                index: this.selectionList.length,
                name: "Selection #" + this.selectionList.length,
                elementRef: selection,
                positionData: positionData,

                type: "singlelineText",
                variable: null,
                isStatic: true,
                staticContent: "",

                internalComponent: null,
            })

            this.selectElementOnClick(selection)
        },

        // element - js object to remove from dom/list
        // Remove element from dom and from the list. Update the index of each element in the list and because the
        //  element can only be deleted once its selected, set the selected element to null
        updateList(element){
            document.getElementById("pdfTemplate").removeChild(element.elementRef);
            let updatedList = this.selectionList.filter(elem => elem != element);

            updatedList.forEach((elem, i) => {
                elem.elementRef.dataset.index = i;
                elem.index = i;
            });
            this.selectionList = updatedList;

            this.selectedElement = null;
        },

        selectElementOnClick(elementDom){
            if(!elementDom) return;
            if(elementDom.classList.contains("internalComponent")) elementDom = elementDom.parentNode

            const element = this.selectionList.filter(element => element.elementRef == elementDom)[0];
            this.elementSelected(element);
        },

        elementSelected(element){
            if(this.selectedElement) this.selectedElement.elementRef.style.border = "";
            
            this.selectedElement = element;

            this.selectedElement.elementRef.style.border = "2px solid #add8e6";
        },

        // elem - html element
        // newData - either (x ,y) or (width, height)
        // Get the element from the list based on the data-index attribute. Replace the old (x ,y) or (width, height) with new ones
        updatePositionData(elem, newData){
            const element = this.selectionList[elem.dataset.index];
            Object.keys(newData).forEach(key => element.positionData[key] = newData[key])
        },

        setPdfTemplate(pdfTemplate, pdfDimensions){
            this.pdfTemplate = pdfTemplate
            this.pdfDimensions = pdfDimensions
        },

        converterResponse(resultStatus){
            this.resultStatus = resultStatus;

            setTimeout(() => {
                this.resultStatus = null
            }, 3000);
        },

        moveElement(modifyBy){
            if(!this.selectedElement) return;
            this.$refs.editElement.updateElementPosition("move", modifyBy)
        },

        resizeElement(modifyBy){
            if(!this.selectedElement) return;
            this.$refs.editElement.updateElementPosition("resize", modifyBy)
        },

        keyboardSupport(event){
            const modifyBy = 10;

            if(event.shiftKey){
                switch(event.key){
                    case "ArrowUp":
                        this.resizeElement({width: 0, height: -modifyBy});
                        break;
                    case "ArrowDown":
                        this.resizeElement({width: 0, height: modifyBy});
                        break;
                    case "ArrowLeft":
                        this.resizeElement({width: -modifyBy, height: 0});
                        break;
                    case "ArrowRight":
                        this.resizeElement({width: modifyBy, height: 0});
                        break;
                }

                return;
            }
            
            switch(event.key){
                case "ArrowUp":
                    this.moveElement({x: 0, y: -modifyBy});
                    break;
                case "ArrowDown":
                    this.moveElement({x: 0, y: modifyBy});
                    break;
                case "ArrowLeft":
                    this.moveElement({x: -modifyBy, y: 0});
                    break;
                case "ArrowRight":
                    this.moveElement({x: modifyBy, y: 0});
                    break;
            }
        },

        toggleColumn(columnSelector){
            const column = document.querySelector(`div.${columnSelector}`)
            column.style.transform = column.style.transform? "": "translateX(0)";

            const chevronArrow = document.querySelector(`div.${columnSelector} img`);
            const curRotation = chevronArrow.style.transform.match(/\d*/g).join("");
            chevronArrow.style.transform = `rotate(${(parseInt(curRotation) + 180) % 360}deg)`;
        },
    },
    mounted(){
        this.interaction();
        document.addEventListener('keydown', this.keyboardSupport)
    },
    beforeDestroy(){
        document.removeEventListener('keydown', this.keyboardSupport);
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

.container{
    @include flex(row, center, stretch);
    width: 100%;

    overflow: hidden;
    background-color: $mainColor;
}

.templateCol{
    @include section(95%, $mainColor);
    @include flex(column, center, center);

    overflow: hidden;
    
    & > .pdfTemplate{
        @include boxShadow();

        width: 210mm;
        min-height: 297mm;

        position: relative;

        background-color: #FFFFFF;
        background-position: center;
        background-repeat: no-repeat; 
        background-size: cover;
    }
}

.informationCol, .elementsCol{
    $contentColWidth: 90%;
    $transition: 0.4s ease-out;

    @include section(25%, $mainColor);
    @include flex(row, initial, initial);
    @include boxShadow();

    transition: $transition;
    transform: translateX($contentColWidth);

    &>.toggleColumn{
        width: calc(100% - #{$contentColWidth});
        
        &>button{
            @include toggleColButton;
        }
    }
    &>.colContent{
        width: $contentColWidth;
    }

    & img{
        padding: 0.25rem;
        transition: $transition;
    }

    &.elementsCol{
        transform: translateX(-$contentColWidth);
        &>.colContent>button{
            @include button(#1EA823, #27DB2C);
        }
    }
    &.informationCol{
        transform: translateX($contentColWidth);
    }
}


.toggleColumn{
    @include flex(row, center, stretch);
}

@media only screen and (max-width: 1200px){

}
</style>