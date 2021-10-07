
<script>
    import { writable } from "svelte/store";
    import {ENDPOINT_API} from './utils/consts'
    import Select from 'svelte-select';

    import {sizesApiData, colorsApiData, productStore} from './stores/stores'

    export let entry_id;
    export function to_json() {
        let ret = {};
        ret.id = $dataStore.id;
        ret.provider = $dataStore.provider;
        ret.colors = [];
        for (let i= 0; i < $colorsValue.length; i++) {
            ret.colors.push($colorsValue[i]['value']);
        }
        ret.sizes = [];
        for (let i = 0; i < $sizesValue.length; i++) {
            ret.sizes.push($sizesValue[i]['value']);
        }
        ret.cost_price = $costPriceValue
        ret.client_price = $clientPriceValue
        ret.recomended_price = $recomendedPriceValue
        ret.providerMakat = '' + $providerMakat;
        ret.parent = $dataStore.parent
        debugger;

        return ret;
    }
    let costPriceValue = new writable([]);
    let clientPriceValue = new writable([]);
    let recomendedPriceValue = new writable([]);
    let providerMakat = new writable([]);
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
                let sizes = convert_api_to_options($sizesApiData, $dataStore.sizes)
                sizesValue.set(sizes);

                let colors = convert_api_to_options($colorsApiData, $dataStore.colors)
                colorsValue.set(colors);
                get_provider_name();

                costPriceValue.set($dataStore.cost_price);
                clientPriceValue.set($dataStore.client_price);
                recomendedPriceValue.set($dataStore.recomended_price);
                providerMakat.set($dataStore.providerMakat);
            })
            .catch(error => console.log('error', error));
    }
    get_details_from_server();
    let providerName;
    function get_provider_name() {
        let url = ENDPOINT_API + 'svelte/api/providers/' + $dataStore.provider + '/';
        fetch(url)
            .then(response => response.json())
            .then(result => {
                providerName = result.label;
                console.log('providerName: ',providerName);
            })
            .catch(error => console.log('error', error));
    }

    // limit options to parent sizes
    
        /*let temp = []
        for(let i =0; i < $sizesApiData.length; i++) {
            for(let j = 0; j < $productStore.sizes.length;j++) {
                if ('' + $productStore.sizes[j] == $sizesApiData[i]['value']) {
                    temp.push($sizesApiData[i]);
                }
            }
        }
        sizes_options = temp;*/
    $: {
        let sizes = convert_api_to_options($sizesApiData, $productStore.sizes);
        sizes_options = sizes;
    }
    $: {
        let colors = convert_api_to_options($colorsApiData, $productStore.colors);
        colors_options = colors;
    }

    function price_component(buy, sell) {
        let prcent = ((buy / sell) - 1)*100        
        return prcent.toFixed(2);
    }
    let cost_profit;
    let client_profit;
    $: {cost_profit = price_component($clientPriceValue, $costPriceValue);}
    $: {client_profit= price_component($recomendedPriceValue ,$clientPriceValue)}
    
</script>
<tr>
    <td>
        {entry_id}
    </td>
    <td>
        {providerName}
    </td>
    <td>
        <Select items={sizes_options} bind:value={$sizesValue} isMulti={true} ></Select>
    </td>
    <td>
        <Select items={colors_options} bind:value={$colorsValue} isMulti={true} ></Select>
    </td>
    <td>
        <input type="number" bind:value={$costPriceValue}>
    </td>
    <td>
        <input type="number" bind:value={$clientPriceValue}>
        <div class="profit_div" class:red={cost_profit<=0}>
            {cost_profit}%
        </div>
    </td>
    <td>
        <input type="number" bind:value={$recomendedPriceValue}>
        <div class="profit_div" class:red={cost_profit<=0}>
            {client_profit}%
        </div>
    </td>
    <td>
        <input type="text" bind:value={$providerMakat}>
    </td>
</tr>

<style>
    table, th, td {
  border: 1px solid black;
}
td {
    margin: 5px;
    padding: 5px;
}
input[type="number"] {
    width: 80%;
}
.profit_div {
    color: green;
}
.red {
    color: red;
}
</style>