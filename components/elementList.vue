<template>
    <div>
        <h2>{{title}}</h2>
        <ul v-if="elementList" class="elementList">
            <li :key="element.index" :element="element" v-for="element in elementList">
                <button @click="selectElement(element.index)">{{element.name}}</button>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    props:{
        elementList: Array,
        title: String
    },
    data(){
        return{
            
        }
    },
    methods:{
        selectElement(index){
            const element = this.elementList[index];
            this.$emit("elementSelected", element);
        },
    },
}
</script>

<style lang="scss" scoped>
@import "./styles/style.scss";

.elementList{
    
    list-style-type: none;
    
    & > li{
        @include flex(row, space-between, initial);
        
        margin: 1rem 0;
        padding: 0 1rem;
    }
}


</style>