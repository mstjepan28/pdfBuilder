<template>
    <div>
        <PdfBuilder/>
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
</style>