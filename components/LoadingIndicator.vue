<template>
  <div class="loading"></div>
</template>

<script>
export default {

}
</script>

<style lang="scss" scoped>
@import "./styles/style.scss";

.loading{
    $spinnerSize: 120px;
    $spinnerThickness: 12px;

    width: $spinnerSize;
    height: $spinnerSize;

    border-radius: 100%;
    border: $spinnerThickness solid rgba(0,0,0,.2);

    animation: spinning 2s infinite linear;

    &:before{
        

        content:"";

        width:100%;
        height:100%;

        display:block;
        
        position:absolute;
        left: -$spinnerThickness;
        top: -$spinnerThickness;
        
        border-top:    $spinnerThickness solid rgba(0,0,0,.8);
        border-left:   $spinnerThickness solid transparent;
        border-bottom: $spinnerThickness solid transparent;
        border-right:  $spinnerThickness solid transparent;
        border-radius:100%;
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