<template>
    <div
        :id="id"
        class="imageUploadComponent internalComponent"
        @dragover.prevent="dragOver" 
        @dragleave.prevent="dragLeave"
        @drop.prevent="drop($event)"

        :class="{ 
            fail: convertRes == 'fail'
        }"
    >
        <div v-if="wrongFile">
            Wrong file type
        </div>
        
        <img v-else-if="imageSource" :src="imageSource" alt="uploaded image"/>
        
        <div v-else-if="!wrongFile">
            Drag and drop an image
        </div>
    </div>
</template>

<script>
export default {
    props:{
        id: {
            type: String,
            required: true
        }
    },
    data(){
        return{
            isDragging: false,
            wrongFile: false,
            imageSource: null,

            convertRes: null,
            resetTimeout: null,
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

            if(file.type.indexOf('image/') < 0) 
                return this.wrongFileFormat();

            const reader = new FileReader()
            reader.onload = (f) => {
                this.imageSource = f.target.result
                this.isDragging = false
            }
            reader.readAsDataURL(file)
        },

        wrongFileFormat(){
            this.convertRes = "fail"

            this.wrongFile = true
            this.imageSource = null
            this.isDragging = false
        },

        setImageURL(imageURL){
            if(this.imageSource && !imageURL) return;
            this.imageSource = imageURL
        },

        getImageSource(){
            return this.imageSource;
        },

        resetState(){
            if(this.resetTimeout) clearTimeout(this.resetTimeout)

            this.wrongFile = false,
            this.convertRes = null
        }
    },
    watch:{
        isDragging(){
            const dragAndDrop = document.getElementById(this.id);
            if(this.isDragging)
                dragAndDrop.classList.add("draggingOver")
            else
                dragAndDrop.classList.remove("draggingOver")
        },
        convertRes(){
            this.resetTimeout = setTimeout(() => this.resetState(), 3000)
        },
    }
}
</script>

<style lang="scss" scoped>
@import "./styles/style.scss";

.imageUploadComponent{
    @include flex(column, center, center);

    width: 100%;
    height: 100%;

    padding: 1px;

    overflow: hidden;
    position: absolute;

    transition: 0.2s ease-in-out;

    & > *{
        pointer-events: none;
    }

    img{
        @include flex(column, center, center);
        
        width: 100%;
        height: 100%;
    }
}

.draggingOver{
    color: $primaryColor;
    background: $blueHighlight;
}

.fail{
    color: $primaryColor;
    background: $redHighlight
}

</style>