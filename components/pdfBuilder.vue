<template>
    <div class="container">
        <div id="pdfDocument" class="pdfBuilder">
            <div class="draggable exampleCube3"></div>
            <div class="draggable exampleCube2"></div>
            <div class="draggable exampleCube1"></div>
        </div>
        <div id="editor"></div>
        <div class="buttonContainer">
            <button class="addElement" @click="addElement()"> Add Element </button>
            <button class="saveBtn" @click="convertToPdf()"> Convert </button>
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
            snapGrid: {
                x: 15,
                y: 15
            },
        }
    },
    methods:{
        addElement(){           
            const newElement = document.createElement("div");

            newElement.style.width = "200px";
            newElement.style.height = "200px";
            newElement.style.position = "absolute";
            newElement.style.backgroundColor = "#" + Math.floor(Math.random()*16777215).toString(16);

            newElement.classList.add("draggable");

            document.getElementById("pdfDocument").appendChild(newElement)
        },

        convertToPdf(){
            const doc = new jsPDF({ 
                unit: "px" 
            });
            const pdfDoc = document.getElementById("pdfDocument");

            try{
                doc.html(pdfDoc, {
                    callback: doc => {
                        const blobPDF =  new Blob([ doc.output() ], { type : 'application/pdf'});
                        const blobUrl = URL.createObjectURL(blobPDF);

                        window.open(blobUrl);
                    },
                    width: pdfDoc.offsetWidth,
                    windowWidth: 1260 
                    
                });
            }catch(err){
                console.log(err)
            }
        },

        interaction(){
            const thisRef = this;

            interact('.draggable')
            .resizable({
                // resize from all edges and corners
                edges: { left: true, right: true, bottom: true, top: true },

                listeners: {
                    move (event) {
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
                    }
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

                inertia: true
            })
            .draggable({
                inertia: true,

                modifiers: [
                    interact.modifiers.snap({
                        targets: [
                            interact.snappers.grid(this.snapGrid)
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
                // enable autoScroll
                //autoScroll: true,

                listeners: {
                    move: dragMoveListener,
                }
            })
            .on('tap', (event) => thisRef.shiftFocus(event.target))

            function dragMoveListener (event) {
                const target = event.target
                
                // keep the dragged position in the data-x/data-y attributes
                const x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx
                const y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy

                // translate the element
                target.style.transform = 'translate(' + x + 'px, ' + y + 'px)'

                // update the posiion attributes
                target.setAttribute('data-x', x)
                target.setAttribute('data-y', y)
            }

            // this function is used later in the resizing and gesture demos
            window.dragMoveListener = dragMoveListener

        },

        shiftFocus(targetElement){
            const elementList = document.querySelectorAll("div.pdfBuilder>div");

            Array.from(elementList).forEach(element => {
                let zIndex = element.style.zIndex || 0;
                element.style.zIndex = (zIndex - 1) < 0? 0: (zIndex - 1);
                element.style.border = "";
            })

            const topZIndex = elementList.length;
            targetElement.style.zIndex = topZIndex;
            targetElement.style.border = "2px solid black";
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
    justify-content: center;
    align-items: center;

    background: darkslategray;

    & > .pdfBuilder{
        width: 750px;
        aspect-ratio: 1 / 1.4142;

        position: relative;

        background-color: #FFFFFF;
    }
}

@mixin button($baseColor, $secondaryColor){
    width: 100%;

    font-weight: 600;
    font-size: 20px;
    color: white;

    margin: 0 1rem;
    padding: 0.75rem 0;

    border: 4px solid $secondaryColor;
    border-radius: 10px;

    background-color: $baseColor;

    &:hover{
        cursor: pointer;
        border-color: $baseColor;
        background-color: $secondaryColor;
    }
}

.buttonContainer{
    width: 30vw;
    display: flex;

    margin-top: 2rem;

    .saveBtn{
        @include button(#1EA823, #27DB2C);
    }
    .addElement{
        @include button(#0f00dd, #2c1ee6);
    }
}
</style>