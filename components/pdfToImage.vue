<template>
    <div
        class="dragAndDropPdf"
        @dragover.prevent="dragOver" 
        @dragleave.prevent="dragLeave"
        @drop.prevent="drop($event)"
    >
        <h4 v-if="wrongFile">Wrong file type</h4>
        <h4 v-if="!pdfSource && !isDragging && !wrongFile">Drop an PDF</h4>
    </div>
</template>

<script>
import axios from "axios";

export default {
    props:{
        apiUrl: String,
    },
    data(){
        return{
            isDragging: false,
            wrongFile: false,
            pdfSource: null,

            isConverting: false
        }
    },
    methods:{
        dragOver(){
            this.isDragging = true
        },
        dragLeave(){
            this.isDragging = false
        },
        drop(event){
            const file = event.dataTransfer.files[0]
            this.wrongFile = false

            if(file.type != "application/pdf") return this.wrongFileFormat();

            const reader = new FileReader()
            reader.onload = (f) => {
                this.pdfSource = f.target.result
                this.isDragging = false
                
                this.convertToImage();
            }
            reader.readAsArrayBuffer(file)
        },

        wrongFileFormat(){
            console.log("wrong format");

            this.wrongFile = true
            this.pdfSource = null
            this.isDragging = false
        },

        // Convert the uploaded PDF to a BLOB and send it as a part of a multiform. Set the responce 
        //  image as the pdfTemplate background and get its width and height
        async convertToImage(){
            if(!this.pdfSource || this.isConverting) return;
            this.isConverting = true;

            const pdfBlob = new Blob([ this.pdfSource ], { type : 'application/pdf'})

            const data = new FormData();
            data.append("pdfTemplate", pdfBlob, "pdfTemplate.pdf")

            const config = { header : {'Content-Type': `multipart/form-data; boundary=${data._boundary}`,} }
            let imageSize = {}

            try{
                const responce = await axios.post(`${this.apiUrl}/convertPdfToImg`, data, config );
                
                const pdfTemplate = document.querySelector(".pdfTemplate");
                pdfTemplate.style.backgroundImage = `url(${responce.data.attachment_url})`;

                imageSize = this.getImageSize(responce.data.attachment_url);
                
            }catch(error){
                console.log(error);
            }

            this.$emit("pdfUploaded", this.pdfSource, imageSize)
            this.isConverting = false;
        },

        getImageSize(imageSrc){
            const imageSize = {}
            const img = new Image()

            img.src = imageSrc
            img.onload = function(){
                imageSize.width = this.width;
                imageSize.height = this.height;
            }

            return imageSize;
        }
    }
}
</script>

<style lang="scss" scoped>
@import "../assets/style.scss";

.dragAndDropPdf{
    @include flex(column, center, center);

    width: 100%;
    min-height: 75px;

    padding: 20px 5px;

    overflow: hidden;

    border-radius: 10px;
    background: white;

    img{
        @include flex(column, center, center);
        
        width: 100%;
        height: 100%;
    }
}

</style>