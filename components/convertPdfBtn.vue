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
                return {
                    positionData: element.positionData,
                    variable: element.variable,
                    type: element.type
                }
            })
        }
    }
}
</script>

<style>

</style>