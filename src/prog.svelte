
<script>
    import {ENDPOINT_API} from './utils/consts'
    import TableRow from './TableRow.svelte'
    import {sizesApiData, colorsApiData, productStore} from './stores/stores'
    export let catalog_image_id = '77';


    function get_product_from_server() {
        let catalogImageUrl = ENDPOINT_API + 'svelte/api/products/' + catalog_image_id + '/';
        fetch(catalogImageUrl)
            .then(response => response.json())
            .then(result => {
                console.log('got data from url ', catalogImageUrl, ' result: ', result);
                productStore.set(result);
            })
            .catch(error => console.log('error', error));
            
    }
    function get_all_sizes() {
        let url = ENDPOINT_API + 'svelte/api/sizes/'
        fetch(url)
            .then(response => response.json())
            .then(result => {
                console.log('got data from url ', url, ' result: ', result);
                sizesApiData.set(result);
            })
            .catch(error => console.log('error', error));
    }
    function get_all_colors() {
        let url = ENDPOINT_API + 'svelte/api/colors/'
        fetch(url)
            .then(response => response.json())
            .then(result => {
                console.log('got data from url ', url, ' result: ', result);
                colorsApiData.set(result);
            })
            .catch(error => console.log('error', error));
    }
    get_all_colors();
    get_all_sizes();
    get_product_from_server();
    function saveToServer() {
        console.log('saveing')
    }
</script>

<main>
    <img class="sm-img" src={$productStore.image} alt={$productStore.title}/>
    <h1>{$productStore.title}</h1>
    {#if $productStore.detailTabel}
        <table>
            <tr>
                <td>id</td>
                <td>ספק</td>
                <td>מידות</td>
                <td>צבעים</td>
            </tr>
            {#each $productStore.detailTabel as entryId}
                <TableRow entry_id={entryId}/>
            {/each}
        </table>
        <button type="button" on:click={saveToServer}>שמור</button>
    {/if}
</main>

<style>
    .sm-img{
        width:150px;
    }
    main {
        direction: rtl;
    }
    table tr {
        border-bottom: 1px solid black;
    }
</style>