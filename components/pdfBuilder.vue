<template>
    <div class="container">
        <ResponceModal 
            :resultStatus="resultStatus"
        />

        <div class="elementsCol">
            <button @click="makeSelection()">Make selection</button>
            <button @click="openResponceModal()"> Open modal </button>
            
            <ConvertPdfBtn 
                :apiUrl="apiUrl" 
                :selectionList="selectionList" 
                :pdfTemplate="pdfTemplate" 
                :pdfDimensions="pdfDimensions"
            />
            <PdfToImage 
                :apiUrl="apiUrl" 
                @pdfUploaded="setPdfTemplate"
            />
        </div>

        <div class="templateCol">
            <div id="pdfTemplate" class="pdfTemplate"></div>
        </div>

        <div class="informationCol">
            <ElementList 
                :title="'Selection list'" 
                :elementList="selectionList" 
                @elementSelected="elementSelected"
            />
            
            <EditElement 
                :apiUrl="apiUrl" 
                :element="selectedElement" 
                @deleteElement="updateList"
            />
        </div>
    </div>
</template>

<script>
import ElementList from "./elementList.vue";
import EditElement from "./editElement.vue";
import ConvertPdfBtn from "./convertPdfBtn.vue"
import PdfToImage from "./pdfToImage.vue";
import ResponceModal from "./responceModal.vue";

import interact from "interactjs";

export default {
    props:{
        apiUrl: String
    },
    components: { ElementList, EditElement, ConvertPdfBtn, PdfToImage, ResponceModal },
    data(){
        return{
            snapGrid: { x: 16, y: 16 },
            
            isSelectionFinished: true,
            selectedElement: null,
            selectionList: [],

            pdfTemplate: null,
            pdfDimensions: null,

            resultStatus: "pending",
        }
    },
    methods:{
        /* ------------------------------------------------------------ */

        interaction(){
            const thisRef = this;

            interact('.draggable')
            .resizable({
                inertia: true,
                
                // resize from all edges and corners
                edges: { left: true, right: true, bottom: true, top: true },

                listeners: {
                    move: event => thisRef.resizeHandler(event)
                },
                modifiers: [
                    // Element cannot grow outside of the parent
                    interact.modifiers.restrictEdges({ outer: 'parent' }),
                ],

            })
            .draggable({
                inertia: true,
                
                modifiers: [
                    interact.modifiers.snap({
                        targets: [
                            // Snap elements in place when dragged
                            interact.snappers.grid(thisRef.snapGrid)
                        ],
                        range: Infinity,
                        relativePoints: [ { x: 0, y: 0 } ]
                    }),

                    // Keeps the element inside the parent
                    interact.modifiers.restrictRect({ restriction: 'parent' })
                ],

                listeners: {
                    move: event => thisRef.dragMoveListener(event),
                },

            })
            .on('tap', (event) => {
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
        // Sets the zIndex of the taget element to be highest in the parent. 
        //  zIndexes of other elements get decresed by 1(lower boundery on 0)
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

                dimesions = {
                    "width": width,
                    "height": height
                }
                updateSelection(selection, dimesions)
            }
            
            const pdfTemplate = document.getElementById("pdfTemplate");
            
            let coordinates = { x: 0, y: 0 };
            let dimesions = { width: 0, height: 0 };
            let selection = "";

            const controller = new AbortController(); 
            let selectionStarted = false;

            this.togglePointerEvents(pdfTemplate, false);

            pdfTemplate.addEventListener("click", (event) => {
                if(selectionStarted){
                    pdfTemplate.removeEventListener("mousemove", getDimensions);
                    selection.classList.add("draggable", "selection");

                    this.togglePointerEvents(pdfTemplate, true);
                    this.saveNewSelection(selection, {...coordinates, ...dimesions})
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

        // parentElement - html element whos childrens pointer events will be toggled
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
        updateSelection(selection, dimesions){
            selection.style.width = dimesions.width + "px";
            selection.style.height = dimesions.height + "px";
        },

        // selection - html element
        // positionData - { x: num, y: num, widht: num, height: num}
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
        // Remove element from dom and from the list. Update the index of each elemet in the list and because the
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
        // Get the element from the list based on the data-index atribute. Replace the old (x ,y) or (width, height) with new ones
        updatePositionData(elem, newData){
            const element = this.selectionList[elem.dataset.index];
            Object.keys(newData).forEach(key => element.positionData[key] = newData[key])
        },

        setPdfTemplate(pdfTemplate, pdfDimensions){
            this.pdfTemplate = pdfTemplate
            this.pdfDimensions = pdfDimensions
        },

        openResponceModal(){
            const modal = document.querySelector(".modalBackground");
            modal.style.display = "flex";
        }
    },
    mounted(){
        this.interaction();
    }
}
</script>

<style lang="scss" scoped>
@import "../assets/style.scss";

.selection{
    background: red !important;
}

.container{
    width: 100vw;
    height: 100vh;

    @include flex(row, center, center);
}

.templateCol{
    @include section(60vw, $lightGray);
    @include flex(column, center, center);
    
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

.informationCol{
    @include section(20vw, $mainColor);

    padding: 1rem;

    color: white;
}

.elementsCol{
    @include section(20vw, $mainColor);
    @include flex(column, initial, initial);

    padding: 1rem;

    button{
        @include button(#1EA823, #27DB2C);
    }
}

</style>