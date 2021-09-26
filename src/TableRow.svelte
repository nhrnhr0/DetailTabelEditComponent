
<script>
    import { writable } from "svelte/store";
    import {ENDPOINT_API} from './utils/consts'
    import Select from 'svelte-select';

    import {sizesApiData, colorsApiData, productStore} from './stores/stores'

    export let entry_id;
    let sizesValue = new writable([]);
    let colorsValue = new writable([]);
    let sizes_options = [];
    let colors_options = [];
    let dataStore = new writable([]);

    function convert_api_to_options(all, part) {
        let temp = []
        for(let i =0; i < all.length; i++) {
            for(let j = 0; j < part.length;j++) {
                if ('' + part[j] == all[i]['value']) {
                    temp.push(all[i]);
                }
            }
        }
        return temp;
    }

    function get_details_from_server() {
        let tabelDataUrl = ENDPOINT_API + 'svelte/api/productTabel/' + entry_id + '/';
        fetch(tabelDataUrl)
            .then(response => response.json())
            .then(result => {
                console.log('got data from url ', tabelDataUrl, ' result: ', result);
                dataStore.set(result);
                debugger;
                let sizes = convert_api_to_options($sizesApiData, $dataStore.sizes)
                sizesValue.set(sizes);

                let colors = convert_api_to_options($colorsApiData, $dataStore.colors)
                colorsValue.set(colors);
            })
            .catch(error => console.log('error', error));
    }
    get_details_from_server();

    // limit options to parent sizes
    $: {
        /*let temp = []
        for(let i =0; i < $sizesApiData.length; i++) {
            for(let j = 0; j < $productStore.sizes.length;j++) {
                if ('' + $productStore.sizes[j] == $sizesApiData[i]['value']) {
                    temp.push($sizesApiData[i]);
                }
            }
        }
        sizes_options = temp;*/
        let sizes = convert_api_to_options($sizesApiData, $productStore.sizes);
        sizes_options = sizes;
        let colors = convert_api_to_options($colorsApiData, $productStore.colors);
        colors_options = colors;
    }
</script>
<tr>
    <td>
        {entry_id}
    </td>
    <td>
        {$dataStore.providerName}
    </td>
    <td>
        <Select items={sizes_options} bind:value={$sizesValue} isMulti={true} ></Select>
    </td>
    <td>
        <Select items={colors_options} bind:value={$colorsValue} isMulti={true} ></Select>
    </td>
</tr>

<style>
    table, th, td {
  border: 1px solid black;
}
td {
    margin: 25px;
    padding: 25px;
}
</style>