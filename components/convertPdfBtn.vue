<template>
    <button class="primaryButton" :class="{progress: isProcessing}" @click="postPDFTemplate()" :disabled="isProcessing"> 
        <div v-if="isProcessing" class="progress">
            <LoadingIndicator 
                id="convertingIndicator" 
                size="30px" 
                thickness="5px"
                color="255, 255, 255"
            />
        </div>
        <div v-else>Convert</div>
    </button>
</template>

<script>
import axios from "axios";

import LoadingIndicator from "./LoadingIndicator.vue";

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
    components: { LoadingIndicator },
    data(){
        return{
            isProcessing: false,
        }
    },
    methods:{
        async postPDFTemplate(){
            if(this.isProcessing) return;
            this.isProcessing = true;

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
                
                this.$emit("openResponse", response.status);
            }catch(error){
                const status = error.message == "Network Error"? 408: 500;
                this.$emit("openResponse", status);
            }

            document.documentElement.style.cursor = "";
            this.isProcessing = false;
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
        },
        
        
    }
}
</script>

<style lang="scss">
@import "./styles/style.scss";

.primaryButton{
    min-height: 45px;
    max-height: 45px;
}

button.primaryButton.progress{
    cursor: initial !important;

    display: flex;
    align-items: center;
    justify-content: center;

    padding: 0;

    color: $secondaryColor;
    background: $blueHighlight;

    &:hover{
        background: $blueHighlight;
    }
}
    
</style>