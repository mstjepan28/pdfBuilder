<template>
    <div>
        <h4>{{title}}</h4>
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
            selectedElement: ""
        }
    },
    methods:{
        editElement(selectedElement){
            this.selectedElement = selectedElement;

            console.log(this.elementList[selectedElement])
        },
        deleteElement(index){
            const element = this.elementList[index].elementRef
            document.getElementById("pdfTemplate").removeChild(element);

            let updatedList = this.elementList.filter(elem => elem.index != index)
            updatedList.forEach((elem, i) => elem.index = i);

            this.$emit("listUpdate", updatedList);
        }
    },
}
</script>

<style lang="scss" scoped>

ul{
    list-style-type: none;
}


</style>