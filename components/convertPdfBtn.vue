<template>
    <button @click="generatePDF()"> Convert </button>
</template>

<script>
import jsPDF from "jspdf";
import axios from "axios";

export default {
    props:{
        selectionList: Array,
    },
    methods:{
        generatePDF(){
            const doc = new jsPDF();
            const pdfDoc = document.getElementById("pdfTemplate");
            const postPDFTemplate = this.postPDFTemplate;

            try{
                doc.html(pdfDoc, {
                    callback: doc => {
                        const pdfBlob = new Blob([ doc.output() ], { type : 'application/pdf'})
                        postPDFTemplate(pdfBlob)
                    },
                    width: 210, // 210mm 
                    windowWidth: pdfDoc.offsetWidth // width of the "div.pdfTemplate"
                });
            }catch(err){
                console.log(err)
            }
        },

        async postPDFTemplate(pdfTemplate){
            const route = "http://localhost:8080/postTemplate";
            const extension = pdfTemplate.type.split('/')[1];  

            const selectionList = this.filterList();
            const blobList = new Blob([JSON.stringify(selectionList)]);
            const blobListExtension = pdfTemplate.type.split('/')[1];

            let data = new FormData();
            data.append("selectionList", blobList, "selectionList" + blobListExtension, { type: "application/json" })
            data.append("pdfTemplate", pdfTemplate, "pdfTemplate." + extension)

            const config = { header : {'Content-Type': `multipart/form-data; boundary=${data._boundary}`,} }

            try{
                const responce = await axios.post( route, data, config );
                console.log(responce)
            }catch(error){
                console.log(error);
            }
        
        },
        
        // Filter data od list elements so it contains only relevent data
        filterList(){
            return this.selectionList.map(element => {
                const normalisedPositionData = this.normalisePositionData(element.positionData)
                console.log(normalisedPositionData)
                return {
                    positionData: normalisedPositionData,
                    variable: element.variable,
                    type: element.type
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
            
        }
    }
}
</script>

<style>

</style>