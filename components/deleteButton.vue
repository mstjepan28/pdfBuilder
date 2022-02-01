<template>
    <div class="deleteContainer">
        <button class="deleteButton" @click="openChoices"> Delete </button>

        <div class="choiceButtons">
            <button class="confirmDelete" type="button" @click="confirmDelete">
                <img src="./svg/SuccessIndicator.svg" alt="Green check mark">
            </button>
            <button class="stopDelete" type="button" @click="closeChoices">
                <img src="./svg/FailIndicator.svg" alt="Red x">
            </button>
        </div>
    </div>
</template>

<script>
export default {
    props:{
        confirmBeforeDelete: {
            type: Boolean,
            required: false,
        }
    },
    data(){
        return{
            closeTimeout: null,
        }
    },
    methods: {
        openChoices(){
            if(!this.confirmBeforeDelete) return this.confirmDelete();

            const deleteButton = document.querySelector("button.deleteButton");
            if(deleteButton.style.width) 
                return this.closeChoices();

            deleteButton.style.width = "60%";
            this.closeTimeout = setTimeout(() => deleteButton.style.width = "", 5000);
        },
        closeChoices(){
            const deleteButton = document.querySelector("button.deleteButton");
            deleteButton.style.width = ""

            if(this.closeTimeout) clearTimeout(this.closeTimeout); // Clear the timeout from openChoices()
        },
        confirmDelete(){
            this.$emit("delete")
        }
    },
}
</script>

<style lang="scss" scoped>
@import "./styles/style.scss";

button, button:hover, button:focus{
    outline: none;
    border: none;
}

.deleteContainer{
    @include flex(row, space-between, center);
    width: 100%;

    padding: 0.5rem 0;
    position: relative;

    border: 2px solid $redHighlight;
    border-radius: 8px;
    background: $primaryColor;

    &>.deleteButton{
        @include flex(row, center, center);
        width: 100%;

        font-size: 20px;
        font-weight: bold;
        color: $primaryColor;

        cursor: pointer;

        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;

        transition: 0.4s ease-in-out;

        border-radius: 6px;
        background: red;
    }

    .choiceButtons{
        @include flex(row, initial, initial);

        width: 40%;
        margin-left: auto;

        & > button{
            @include flex(row, center, center);
            width: 100%;

            cursor: pointer;

            background: none;

            &>img{
                $buttonSize: 1.75rem;
                width: $buttonSize;
                height: $buttonSize;
            }
        }
    }
}

</style>