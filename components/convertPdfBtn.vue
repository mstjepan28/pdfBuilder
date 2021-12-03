<template>
    <button @click="postPDFTemplate()"> Convert </button>
</template>

<script>
import axios from "axios";

export default {
    props:{
        selectionList: Array,
        pdfTemplate: ArrayBuffer,
        pdfDimensions: Object
    },
    methods:{
        async postPDFTemplate(){
            const selectionList = this.filterList();
            const pdfDimensions = this.pdfDimensions || {width: 596, height: 842}

            const blobPdfDimensions = new Blob([ JSON.stringify(pdfDimensions) ], { type: "application/json" });
            const blobSelectionList = new Blob([ JSON.stringify(selectionList) ], { type: "application/json" });
            const blobPdfTemplate = new Blob([ this.pdfTemplate ], { type : 'application/pdf'})

            let data = new FormData();
            data.append("pdfDimensions", blobPdfDimensions, "pdfDimensions")
            data.append("selectionList", blobSelectionList, "selectionList")
            data.append("pdfTemplate", blobPdfTemplate, "pdfTemplate.pdf")

            try{
                const responce = await axios.post( "http://localhost:8080/postTemplate", data, {
                    header : {
                        'Content-Type': `multipart/form-data; boundary=${data._boundary}`
                    }
                });
                console.log(responce)
            }catch(error){
                console.log(error);
            }
        },

        // Filter data od list elements so it contains only relevent data
        filterList(){
            return this.selectionList.map(element => {
                const normalisedPositionData = this.normalisePositionData(element.positionData);
                const staticContent = this.formatParagraph(element);

            console.log(staticContent)

                return {
                    positionData: normalisedPositionData,
                    type: element.type,
                    variable: element.variable,
                    staticContent: staticContent || false
                }
            })
        },

        // This PDF and the one on the frontend do not match in dimensions so the positionData is offset. 
        //  To fix that the positionData is the percentage of the width/height of the PDF
        normalisePositionData(positionData){
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

        // If the element type is paragraph, break up the given string into substring of assumed row width.
        //  Join the substring back with the separator <br/> 
        formatParagraph(element){
            if(element.type != "paragraph") return element.staticContent;

            const width = element.positionData.width;
            const avgLetterWidth = 8;
            const regex = new RegExp(`.{1,${Math.floor(width / avgLetterWidth)}}`, 'g');

            return element.staticContent.match(regex).join("<br />")
        },
    }
}
</script>

<style>

</style>