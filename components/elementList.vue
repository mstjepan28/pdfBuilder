<template>
    <div>
        <h2 v-if="title">{{title}}</h2>
        <ul v-if="elementList" class="elementList">
            <li :key="element.index" :element="element" v-for="element in elementList">
                <button class="primaryButton" @click="selectElement(element.index)">
                    {{element.name}}
                </button>
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    props:{
        title: {
            type: String,
            required: false,
        },
        elementList: {
            type: Array,
            required: true,
        },
    },
    methods:{
        selectElement(index){
            const element = this.elementList[index];
            this.$emit("elementSelected", element);
        },
    }
}
</script>

<style lang="scss" scoped>
@import "./styles/style.scss";

.active{
    border: 1px solid $highlightColor !important;
}
.elementList{
    @include flex(column, center, initial);
    max-height: 15rem;

    row-gap: 0.5rem;
    list-style-type: none;

    overflow-y: auto;

    margin-top: 1rem;

    & > li{
        @include flex(row, center, center);
    }
}

.primaryButton{
    padding: 0.25rem 0;
}


</style>