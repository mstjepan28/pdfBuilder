<template>
    <div
        class="dragAndDropPdf"
        @dragover.prevent="dragOver" 
        @dragleave.prevent="dragLeave"
        @drop.prevent="drop($event)"

        :class="{
            success: convertRes == 'success',
            fail: convertRes == 'fail',
            processing: isConverting
        }"
    >
        <div v-if="wrongFile">
            Wrong file type
        </div>
        <div v-else-if="isConverting">
            <LoadingIndicator 
                id="pdfToImgLoading" 
                size="64px" 
                thickness="8px"
                color="255, 255, 255"
            />
        </div>
        <div v-else-if="!isConverting && convertRes">
            {{feedbackMsg}}
        </div>
        <div v-else-if="!wrongFile">
            Drop a PDF to set as <br> base template
        </div>
    </div>
</template>

<script>
import LoadingIndicator from "./LoadingIndicator.vue";

import axios from "axios";

export default {
    props:{
        apiUrl: {
            type: String,
            required: true
        },
    },
    components:{ LoadingIndicator },
    data(){
        return{
            isConverting: false,
            isDragging: false,

            pdfSource: null,
            wrongFile: false,

            convertRes: null,
            resetTimeout: null,

            feedbackMsg: null,
        }
    },
    methods:{
        dragOver(){
            this.resetState();
            this.isDragging = true
        },
        dragLeave(){
            this.isDragging = false
        },
        drop(event){
            this.resetState();
            
            const file = event.dataTransfer.files[0]

            if(file.type != "application/pdf") 
                return this.wrongFileFormat();

            const reader = new FileReader()
            reader.onload = (file) => {
                this.pdfSource = file.target.result
                this.isDragging = false
                
                this.convertToImage();
            }
            reader.readAsArrayBuffer(file)
        },

        wrongFileFormat(){
            this.convertRes = "fail"

            this.wrongFile = true
            this.pdfSource = null
            this.isDragging = false
        },

        // Convert the uploaded PDF to a BLOB and send it as a part of a multiform. Set the response 
        //  image as the pdfTemplate background and get its width and height
        async convertToImage(){
            if(!this.pdfSource || this.isConverting) return;
            this.isConverting = true;

            const pdfBlob = new Blob([ this.pdfSource ], { type : 'application/pdf'})

            const data = new FormData();
            data.append("pdfTemplate", pdfBlob, "pdfTemplate.pdf")

            const config = { header : {'Content-Type': `multipart/form-data; boundary=${data._boundary}`,} }
            let imageSize = {}

            document.documentElement.style.cursor = "wait";

            try{
                const response = await axios.post(`${this.apiUrl}/convertPdfToImg`, data, config );
                const pdfTemplate = document.getElementById("pdfTemplate");
                
                pdfTemplate.style.backgroundImage = `url(${response.data.attachment_url})`;

                imageSize = this.getImageSize(response.data.attachment_url);
                
                this.convertRes = "success";
                this.feedbackMsg = "Success!"
            }catch(error){
                this.feedbackMsg = error.response == undefined? "Error: server isn't responding": "Something went wrong"
                this.convertRes = "fail";
            }

            document.documentElement.style.cursor = "";

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
        },

        resetState(){
            if(this.resetTimeout) clearTimeout(this.resetTimeout);

            this.wrongFile = false,
            this.convertRes = null
        }
    },
    watch:{
        isDragging(){
            const dragAndDrop = document.querySelector("div.dragAndDropPdf");
            if(this.isDragging)
                dragAndDrop.classList.add("draggingOver")
            else
                dragAndDrop.classList.remove("draggingOver")
        },
        convertRes(){
            this.resetTimeout = setTimeout(() => this.resetState(), 3000)
        }
    }
}
</script>

<style lang="scss" scoped>
@import "./styles/style.scss";

.dragAndDropPdf{
    @include flex(column, center, center);

    width: 100%;
    max-height: 10rem;

    font-size: 20px;
    font-weight: bold;

    padding: 4rem 0;

    overflow: hidden;

    transition: 0.2s ease-in-out;

    border-radius: 8px;
    background: $secondaryColor;

    img{
        @include flex(column, center, center);
        
        width: 100%;
        height: 100%;
    }
    div{
        text-align: center;
    }
}

.draggingOver, .processing{
    color: $primaryColor;
    background: $blueHighlight;
}

.success{
    color: $primaryColor;
    background: $greenHighlight;
}

.fail{
    color: $primaryColor;
    background: $redHighlight;
}

</style>