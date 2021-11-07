<template>
    <div class="container">
        <div class="elementsCol">
            <button @click="addElement()"> Add Element </button>
            <button @click="makeSelection()">Make selection</button>
            <button @click="convertToPdf()"> Convert </button>
        </div>

        <div class="templateCol">
            <div id="pdfTemplate" class="pdfTemplate"></div>
        </div>

        <div class="informationCol">
            <div>
                <ElementList :title="'Element list'" :elementList="elementList"  @elementSelected="elementSelected"/>
                <ElementList :title="'Selection list'" :elementList="selectionList" @elementSelected="elementSelected"/>
            </div>
            <div>
                <EditElement :element="selectedElement" @deleteElement="updateList"/>
            </div>
        </div>
    </div>
</template>

<script>
import ElementList from "./elementList.vue"
import EditElement from "./editElement.vue"

import interact from "interactjs";
import jsPDF from "jspdf";

export default {
    components: { ElementList, EditElement },
    data(){
        return{
            snapGrid: { x: 16, y: 16 },
            
            isSelectionFinished: true,
            selectedElement: null,
            selectionList: [],
            elementList: [],
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
                autoScroll: true,
                
                modifiers: [
                    interact.modifiers.snap({
                        targets: [
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
            .on('tap', (event) => thisRef.shiftFocus(event.target))
        },
        
        dragMoveListener (event) {
            const target = event.target;
            
            // keep the dragged position in the data-x/data-y attributes
            const x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx
            const y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy

            this.updatePositionData(target, { x, y })

            // translate the element
            target.style.transform = 'translate(' + x + 'px, ' + y + 'px)'

            // update the posiion attributes
            target.setAttribute('data-x', x)
            target.setAttribute('data-y', y)
        },

        resizeHandler(event) {
            const target = event.target
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

        convertToPdf(){
            const doc = new jsPDF();
            const pdfDoc = document.getElementById("pdfTemplate");

            try{
                doc.html(pdfDoc, {
                    callback: doc => {
                        const blobPDF =  new Blob([ doc.output() ], { type : 'application/pdf'});
                        const blobUrl = URL.createObjectURL(blobPDF);

                        window.open(blobUrl);
                    },
                    width: 210, // 210mm 
                    windowWidth: pdfDoc.offsetWidth // width of the "div.pdfTemplate"
                    
                });
            }catch(err){
                console.log(err)
            }
        },

        /* ------------------------------------------------------------ */
        
        shiftFocus(targetElement){
            const elementList = document.querySelectorAll("div.pdfTemplate>div");

            Array.from(elementList).forEach(element => {
                let zIndex = element.style.zIndex || 0;
                element.style.zIndex = (zIndex - 1) < 0? 0: (zIndex - 1);
            })

            const topZIndex = elementList.length;
            targetElement.style.zIndex = topZIndex;
        },

        // Temporary for testing 
        addElement(){           
            const newElement = document.createElement("div");

            newElement.dataset.x = 0;
            newElement.dataset.y = 0;

            newElement.style.width = "200px";
            newElement.style.height = "200px";
            newElement.style.position = "absolute";
            newElement.style.backgroundColor = "#" + Math.floor(Math.random()*16777215).toString(16);

            newElement.classList.add("draggable");
            newElement.dataset.index = this.elementList.length;

            this.elementList.push({
                type: "element",
                index: this.elementList.length,
                name: "Element #" + this.elementList.length,
                elementRef: newElement,
                positionData: { x: 0, y: 0, width: 200, height: 200 }
            })

            document.getElementById("pdfTemplate").appendChild(newElement)
        },

        makeSelection(){
            if(!this.isSelectionFinished) return;
            this.isSelectionFinished = false;

            const updateSelection = this.updateSelection; // change later

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

        togglePointerEvents(parentElement, boolValue){
            Array.from(parentElement.children).forEach(child => child.style.pointerEvents = boolValue? "auto": "none")
        },

        createSelection(coordinates){
            const newSelection = document.createElement("div");
            
            newSelection.dataset.x = coordinates.x;
            newSelection.dataset.y = coordinates.y;
            
            newSelection.style.transform = "translate(" + coordinates.x + "px, " + coordinates.y + "px)";

            newSelection.style.position = "absolute";
            newSelection.style.backgroundColor = "rgba(173,216,230, 0.25)";

            document.getElementById("pdfTemplate").appendChild(newSelection)

            return newSelection;
        },

        updateSelection(selection, dimesions){
            selection.style.width = dimesions.width + "px";
            selection.style.height = dimesions.height + "px";
        },

        saveNewSelection(selection, positionData){
            selection.dataset.index = this.selectionList.length;

            this.selectionList.push({
                type: "selection",
                index: this.selectionList.length,
                name: "Selection #" + this.selectionList.length,
                elementRef: selection,
                positionData: positionData,
            })
        },

        updateElementList(updatedList){
            this.selectedElement = null;
            this.elementList = updatedList;
        },

        updateList(element){
            document.getElementById("pdfTemplate").removeChild(element.elementRef);

            let updatedList = this[element.type + "List"].filter(elem => elem != element);

            updatedList.forEach((elem, i) => {
                elem.elementRef.dataset.index = i;
                elem.index = i;
            });
            this[element.type + "List"] = updatedList;

            this.selectedElement = null;
        },

        elementSelected(element){
            this.selectedElement = element
        },

        updatePositionData(elem, newData){
            const list = elem.classList.contains("selection")? this.selectionList: this.elementList;

            const element = list[elem.dataset.index];
            Object.keys(newData).forEach(key => element.positionData[key] = newData[key])
        }
    },
    mounted(){
        this.interaction();
    }
}
</script>

<style lang="scss" scoped>
@import "../assets/style.scss";

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