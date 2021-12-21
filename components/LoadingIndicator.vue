<template>
    <div class="loading" :id="id"></div>
</template>

<script>
export default {
    props: {
        id:{
            type: String,
            required: false,
        },
        size: {
            type: String,
            required: false
        },
        thickness: {
            type: String,
            required: false,
        }
    },
    methods:{
        setValues(){
            if(!this.id) return;

            const spinner = document.getElementById(this.id);
            if(this.size) 
                spinner.style.setProperty("--size", this.size)
            if(this.thickness) 
                spinner.style.setProperty("--thickness", this.thickness)
        }
    },
    mounted() {
        this.setValues()
    },
}

</script>

<style lang="scss" scoped>
@import "./styles/style.scss";

.loading{
    --size: 120px;
    --thickness: 12px;

    width: var(--size);
    height: var(--size);

    border-radius: 100%;
    border: var(--thickness) solid rgba(0,0,0,.2);

    animation: spinning 2s infinite linear;

    &:before{
        content:"";

        width:100%;
        height:100%;

        display:block;
        
        position:absolute;
        left: calc(var(--thickness) * -1);
        top: calc(var(--thickness) * -1);
        
        border-top:    var(--thickness) solid rgba(0,0,0,.8);
        border-left:   var(--thickness) solid transparent;
        border-bottom: var(--thickness) solid transparent;
        border-right:  var(--thickness) solid transparent;
        border-radius: 100%;
    }
}

@keyframes spinning{
    from{
        transform: rotate(0deg);
    }
    to{
        transform: rotate(359deg);
    }
}

</style>