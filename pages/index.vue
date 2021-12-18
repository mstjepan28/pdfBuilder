<template>
    <div class="wrapper">
        <PdfBuilder :apiUrl="'http://localhost:8080'"/>
    </div>    
</template>

<script>
import PdfBuilder from "@/components/pdfBuilder.vue"

export default {
    components: { PdfBuilder }
}
</script>

<style>
    *{
        margin: 0;
        padding: 0;
    }
    body{
        background-color: #DDDDDD;
    }
    .wrapper{
        width: 100vw;
        height: 100vh;
        
        display: flex;
    }
</style>