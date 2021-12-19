<template>
    <div>
        <h2>{{title}}</h2>
        <ul v-if="elementList" class="elementList">
            <li :key="element.index" :element="element" v-for="element in elementList">
                <button   @click="selectElement(element.index)">
                    {{element.name}}
                </button>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    props:{
        elementList: Array,
        title: String,
        selectedElement: Object
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

.active{
    border: 1px solid $highlightColor !important;
}
.elementList{
    @include flex(column, center, initial);
    row-gap: 0.5rem;
    list-style-type: none;
    
    & > li{
        @include flex(row, center, center);

        & button{
            width: 90%;

            font-weight: bold;
            word-break: normal;
            text-align: start;

            padding: 0.5rem;

            cursor: pointer;

            outline: none;
            border: none;
            border-radius: 8px;

            background: $secondaryColor;
        
            &:focus{
                outline: none;
                text-decoration: underline;
            }
        }
    }

}


</style>