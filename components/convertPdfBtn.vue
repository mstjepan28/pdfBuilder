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
            const selectionList = this.filterList();

            try{
                const responce = await axios.post( route, { pdfTemplate, selectionList });
                console.log(responce)
            }catch(error){
                console.log(error);
            }
        
        },
        
        filterList(){
            return this.selectionList.map(element => {
                const normalisedPositionData = this.normalisePositionData(element.positionData)
                return {
                    positionData: normalisedPositionData,
                    variable: element.variable,
                    type: element.type
                }
            })
        },

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