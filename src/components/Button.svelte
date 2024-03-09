<script>
    export let data = {};
    import Popup from './Popup.svelte';

    import { getContext } from 'svelte';
    import { fly } from 'svelte/transition';

    const { open } = getContext('simple-modal');

    const showPopup = () => {
        open(Popup, { message: "It's a popup!", data: data});
    };
</script>

<main>
    <button on:click={showPopup}>Visualize Amount</button>
</main>

<style>
    button {
        margin-top: 10px;
        padding: 10px 8px;
        border-radius: 15px;
        background: #0057B7;
        color: #FFF;
        border: 0px;
    }

    button:hover {
        background: #003C8F;
        cursor: pointer;
    }

</style>