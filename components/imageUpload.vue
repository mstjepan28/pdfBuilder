<template>
    <div
        class="imageUploadComponent internalComponent"
        @dragover.prevent="dragOver" 
        @dragleave.prevent="dragLeave"
        @drop.prevent="drop($event)"
    >
        <img :src="imageSource" v-if="imageSource" />
        <h1 v-if="wrongFile">Wrong file type</h1>
        <h1 v-if="!imageSource && !isDragging && !wrongFile">Drop an image</h1>
    </div>
</template>

<script>
export default {
    data(){
        return{
            isDragging: false,
            wrongFile: false,
            imageSource: null
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

            if(file.type.indexOf('image/') < 0) return this.wrongFileFormat();

            const reader = new FileReader()
            reader.onload = (f) => {
                this.imageSource = f.target.result
                this.isDragging = false
            }
            reader.readAsDataURL(file)
        },

        wrongFileFormat(){
            console.log("wrong format");

            this.wrongFile = true
            this.imageSource = null
            this.isDragging = false
        },

        setImageURL(imageURL){
            this.imageSource = imageURL
        }
    }
}
</script>

<style lang="scss" scoped>
@import "../assets/style.scss";

.imageUploadComponent{
    @include flex(column, center, center);

    width: 100%;
    height: 100%;

    padding: 5px;

    overflow: hidden;
    position: absolute;

    pointer-events: none;

    img{
        @include flex(column, center, center);
        
        width: 100%;
        height: 100%;
    }
}

</style>