<template>
    <button class="primaryButton" @click="postPDFTemplate()"> Convert </button>
</template>

<script>
import axios from "axios";

export default {
    props:{
        apiUrl: {
            type: String,
            required: true
        },
        selectionList: {
            type: Array,
            required: true
        },
        pdfTemplate: {
            type: ArrayBuffer,
            required: false

        },
        pdfDimensions: {
            type: Object,
            required: false
        }
    },
    methods:{
        async postPDFTemplate(){
            this.openResponseModal();

            const selectionList = this.filterList();
            const pdfDimensions = this.pdfDimensions || {width: 596, height: 842}

            const blobPdfDimensions = new Blob([ JSON.stringify(pdfDimensions) ], { type: "application/json" });
            const blobSelectionList = new Blob([ JSON.stringify(selectionList) ], { type: "application/json" });
            const blobPdfTemplate = new Blob([ this.pdfTemplate ], { type : 'application/pdf'})

            let data = new FormData();
            data.append("pdfDimensions", blobPdfDimensions, "pdfDimensions")
            data.append("selectionList", blobSelectionList, "selectionList")
            data.append("pdfTemplate", blobPdfTemplate, "pdfTemplate.pdf")

            document.documentElement.style.cursor = "wait"

            try{
                const response = await axios.post(`${this.apiUrl}/postTemplate`, data, {
                    header : {
                        'Content-Type': `multipart/form-data; boundary=${data._boundary}`
                    }
                });
                
                this.$emit("converterResponse", response.status);
            }catch(error){
                this.$emit("converterResponse", 500);
            }

            document.documentElement.style.cursor = "";
        },

        openResponseModal(){
            const modal = document.querySelector(".modalBackground");
            modal.style.display = "flex";
        },

        // Filter data od list elements so it contains only relevant data
        filterList(){
            return this.selectionList.map(element => {
                const normalizedPositionData = this.normalizePositionData(element.positionData);
                const staticContent = this.getStaticContent(element);
                
                return {
                    positionData: 
                    normalizedPositionData,
                    type: element.type,
                    variable: element.variable,
                    staticContent: staticContent
                }
            })
        },

        // This PDF and the one on the frontend do not match in dimensions so the positionData is offset. 
        //  To fix that the positionData is the percentage of the width/height of the PDF
        normalizePositionData(positionData){
            const pdfTemplate = document.querySelector("div.pdfTemplate")
            const width = pdfTemplate.offsetWidth;
            const height = pdfTemplate.offsetHeight;

            return{
                x: positionData.x / width,
                y: positionData.y / height, 
                width: positionData.width / width, 
                height: positionData.height / height, 
            }
            
        },

        // Get the base64 image from the internal component. If it doesn't exist return the existing static 
        //  content or false
        getStaticContent(element){
            if(!element.internalComponent) return element.staticContent || false;
            return element.internalComponent.getImageSource();
        }   
    }
}
</script>