<template>
    <div class="errorPopup" @click="closePopup()">
        <div class="feedbackWrapper"
            :class="{
                fail: requestResult == 'fail',
                success: requestResult == 'success'
            }"
        >
            {{feedbackMsg}}
        </div>
    </div>
</template>

<script>
export default {
    data(){
        return{
            isActive: false,
            closeTimeout: null,

            statusCode: "",
            customSuccessMsg: "",
        }
    },
    computed:{
        requestResult(){
            if(this.statusCode >= 400) 
                return "fail";

            return "success";
        },
        feedbackMsg(){
            const statusHandler = {
                400: "Bad request",
                401: "Unauthorized",
                402: "Payment Required",
                403: "Forbidden",
                404: "Not Found",
                405: "Method Not Allowed",
                406: "Not Acceptable",
                407: "Proxy Authentication Required",
                408: "Request Timeout",
                409: "Conflict",
                410: "Gone",
                411: "Length Required",
                412: "Precondition Failed",
                413: "Request Entity Too Large",
                414: "Request-URI Too Long",
                415: "Unsupported Media Type",
                416: "Requested Range Not Satisfiable",
                417: "Expectation Failed",
                500: "Internal Server Error",
                501: "Not Implemented",
                502: "Bad Gateway",
                503: "Service Unavailable",
                504: "Gateway Timeout",
                505: "HTTP Version Not Supported",
                511: "Network Authentication Required",
            }
            const message = statusHandler[this.statusCode] || "Successfully generated PDF files!"
            return this.customSuccessMsg? this.customSuccessMsg: message;
        }
    },
    methods:{
        setResponseData(newResult, customSuccessMsg){
            this.statusCode = newResult;
            this.customSuccessMsg = customSuccessMsg;
            this.openPopup();
        },

        openPopup(){
            const popup = document.querySelector("div.errorPopup");

            popup.classList.add("slideDown");
            popup.classList.remove("slideUp");

            if(!this.closeTimeout) setTimeout(() => this.closePopup(), 5000)
        },
        closePopup(){
            if(this.closeTimeout){
                clearTimeout(this.closeTimeout);
                this.closeTimeout = null;
            }

            const popup = document.querySelector("div.errorPopup");

            popup.classList.add("slideUp");
            popup.classList.remove("slideDown");
        }
    }
}
</script>

<style lang="scss" scoped>
@import "./styles/style.scss";

$slideUpDistance: -100px;
$slideDownDistance: 20px;
$animationDuration: 0.4s;

.success{
    border: 2px solid $greenHighlight;
    background: $greenHighlight;
}

.fail{
    border: 2px solid $redHighlight;
    background: $redHighlight;
}

.errorPopup{
    @include flex(row, center, stretch);

    width: 400px;
    min-height: 50px;

    position: absolute;
    left: 20px;

    overflow: hidden;

    border-radius: 8px;
    background: $secondaryColor;

    transform: translateY($slideUpDistance);

    &>.feedbackWrapper{
        @include flex(row, center, center);

        width: 100%;

        font-size: 16px;
        font-weight: 700;
        color: $primaryColor;

        border-radius: 8px;
    }
}

.slideUp{
    animation: slideUp $animationDuration ease-in-out;
    animation-fill-mode: forwards;
}
.slideDown{
    animation: slideDown $animationDuration ease-in-out;
    animation-fill-mode: forwards;
}

@keyframes slideUp {
    from{
        transform: translateY($slideDownDistance);
    }
    to{
        transform: translateY($slideUpDistance);
    }
}

@keyframes slideDown {
    from{
        transform: translateY($slideUpDistance);
    }
    to{
        transform: translateY($slideDownDistance);
    }
}


</style>