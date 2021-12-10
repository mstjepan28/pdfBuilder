<template>
  <div class="loading"></div>
</template>

<script>
export default {

}
</script>

<style lang="scss" scoped>
@import "../assets/style.scss";

.loading{
    $spinerSize: 120px;
    $spinerThickness: 12px;

    width: $spinerSize;
    height: $spinerSize;

    border-radius: 100%;
    border: $spinerThickness solid rgba(0,0,0,.2);

    animation: roation 2s infinite linear;

    &:before{
        

        content:"";

        width:100%;
        height:100%;

        display:block;
        
        position:absolute;
        left: -$spinerThickness;
        top: -$spinerThickness;
        
        border-top:    $spinerThickness solid rgba(0,0,0,.8);
        border-left:   $spinerThickness solid transparent;
        border-bottom: $spinerThickness solid transparent;
        border-right:  $spinerThickness solid transparent;
        border-radius:100%;
    }
}

@keyframes roation{
    from{
        transform: rotate(0deg);
    }
    to{
        transform: rotate(359deg);
    }
}

</style>