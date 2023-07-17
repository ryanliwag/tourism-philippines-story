<script>
    export let button_label
    export let button_callback

</script>

<button class="button_style" on:click={button_callback()}>
    {button_label}
</button>

<style>
    .button_style {
        /* position: absolute; */
        background-color: tomato;
        opacity: 0.8;
        padding: 10px;
        margin:2px 10px;
        color: whitesmoke;
        width: 8rem;
        border-radius: 5px;
    }
</style>