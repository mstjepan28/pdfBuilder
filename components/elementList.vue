<template>
    <div>
        <h2>{{title}}</h2>
        <ul v-if="elementList" class="elementList">
            <li :key="element.index" :element="element" v-for="element in elementList">
                <button @click="editElement(element.index)">{{element.name}}</button>
                <button @click="deleteElement(element.index)"> Delete </button>
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
        editElement(index){
            const element = this.elementList[index]
            this.$emit("elementSelected", element);
        },
        deleteElement(index){
            const element = this.elementList[index].elementRef
            document.getElementById("pdfTemplate").removeChild(element);

            let updatedList = this.elementList.filter(elem => elem.index != index)
            updatedList.forEach((elem, i) => {
                elem.elementRef.dataset.index = i;
                elem.index = i;
            });

            this.$emit("listUpdate", updatedList);
        }
    },
}
</script>

<style lang="scss" scoped>
@import "../assets/style.scss";

.elementList{
    
    list-style-type: none;
    
    & > li{
        @include flex(row, space-between, initial);
        
        margin: 1rem 0;
        padding: 0 1rem;
    }
}


</style>