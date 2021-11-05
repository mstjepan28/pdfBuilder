<template>
    <div class="container">
        <div class="buttonContainer">
            <button @click="addElement()"> Add Element </button>
            <button @click="makeSelection()">Make selection</button>
            <button @click="convertToPdf()"> Convert </button>
        </div>

        <div id="pdfTemplate" class="pdfTemplate">
            <!--
            <div class="draggable exampleCube3"></div>
            <div class="draggable exampleCube2"></div>
            <div class="draggable exampleCube1"></div>
            -->
        </div>
    </div>
</template>

<script>
import interact from "interactjs";
import jsPDF from "jspdf";

export default {
    components: {  },
    data(){
        return{
            snapGrid: { x: 15, y: 15 },
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
                    interact.modifiers.restrictEdges({
                        outer: 'parent'
                    }),

                    // Smallest width/height of the element
                    interact.modifiers.restrictSize({
                        min: { width: 100, height: 50 }
                    })
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
                    interact.modifiers.restrictRect({
                        restriction: 'parent',
                        endOnly: true
                    })
                ],

                listeners: {
                    move: event => thisRef.dragMoveListener(event),
                },

            })
            .on('tap', (event) => thisRef.shiftFocus(event.target))
        },
        
        dragMoveListener (event) {
            const target = event.target
            
            // keep the dragged position in the data-x/data-y attributes
            const x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx
            const y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy

            // translate the element
            target.style.transform = 'translate(' + x + 'px, ' + y + 'px)'

            // update the posiion attributes
            target.setAttribute('data-x', x)
            target.setAttribute('data-y', y)
        },

        resizeHandler(event) {
            var target = event.target
            var x = (parseFloat(target.getAttribute('data-x')) || 0)
            var y = (parseFloat(target.getAttribute('data-y')) || 0)

            // update the element's style
            target.style.width = event.rect.width + 'px'
            target.style.height = event.rect.height + 'px'

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
                element.style.border = "";
            })

            const topZIndex = elementList.length;
            targetElement.style.zIndex = topZIndex;
            targetElement.style.border = "2px solid black";
        },

        addElement(){           
            const newElement = document.createElement("div");

            newElement.style.width = "200px";
            newElement.style.height = "200px";
            newElement.style.position = "absolute";
            newElement.style.backgroundColor = "#" + Math.floor(Math.random()*16777215).toString(16);

            newElement.classList.add("draggable");

            document.getElementById("pdfTemplate").appendChild(newElement)
        },

        makeSelection(){
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
                    "width": width < 10? 10: width,
                    "height": height < 10? 10: height
                }
                updateSelection(selection, dimesions)
            }
            
            const pdfTemplate = document.getElementById("pdfTemplate");
            
            let coordinates = { x: 0, y: 0 };
            let dimesions = { width: 0, height: 0 };
            let selection = "";

            const controller = new AbortController(); 
            let selectionStarted = false;

            pdfTemplate.addEventListener("click", (event) => {
                if(selectionStarted){
                    pdfTemplate.removeEventListener("mousemove", getDimensions);
                    selection.classList.add("draggable", "selection");
                    
                    console.log({...coordinates, ...dimesions})

                    controller.abort();
                    return 
                }
                selectionStarted = true;
                
                getCoordinates(event);
                selection = this.createSelection(coordinates);

                pdfTemplate.addEventListener('mousemove',  getDimensions);
            }, { signal: controller.signal })
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
        }
    },
    mounted(){
        this.interaction();
    }
}
</script>

<style lang="scss" scoped>

*{
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

/* -------------------------------------------------------------------- */

@mixin cube($color, $left){
    width: 200px;
    height: 200px;

    position: absolute;
    left: $left;
    background: $color;
}

.exampleCube1{
    @include cube(red, 0px);
}
.exampleCube2{
    @include cube(blue, 100px);
}
.exampleCube3{
    @include cube(green, 200px);
}

/* -------------------------------------------------------------------- */

.container{
    width: 100vw;
    height: 100vh;

    display: flex;
    flex-direction: column;

    align-items: center;
    justify-content: center;

    background: darkslategray;

    & > .pdfTemplate{
        width: 210mm;
        min-height: 297mm;

        position: relative;
        //left: 15vw;

        background-color: #FFFFFF;
    }
}

@mixin button($baseColor, $secondaryColor){
    width: 100%;

    font-weight: 600;
    font-size: 20px;
    color: white;

    padding: 0.75rem 0;
    margin-bottom: 1rem;

    border: 4px solid $secondaryColor;
    border-radius: 10px;

    background-color: $baseColor;

    &:hover{
        cursor: pointer;
        border-color: $baseColor;
        background-color: $secondaryColor;
    }

    &:disabled{
        border-color: #666666;
        background-color: #888888;
    }
    &:disabled:hover{
        cursor: initial;
    }
}

.buttonContainer{
    width: 15vw;
    height: 100vh;

    display: flex;
    flex-direction: column;

    padding: 1rem;

    position: absolute;
    top: 0;
    left: 0;

    button{
        @include button(#1EA823, #27DB2C);
    }
}
</style>