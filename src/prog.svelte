<script>
    import {
        DialogOverlay,
        DialogContent
    } from 'svelte-accessible-dialog';
    import {
        ENDPOINT_API
    } from './utils/consts';
    import TableRow from './TableRow.svelte';
    import {
        sizesApiData,
        colorsApiData,
        productStore
    } from './stores/stores';
    import {
        NotificationDisplay,
        notifier
    } from '@beyonk/svelte-notifications';

    export let catalog_image_id = '167';
    let notification_timeout = 3500;
    let rows = [];
    let username, password;
    let isModalOpen;
    const openModal = () => {
        isModalOpen = true;
    };

    const closeModal = () => {
        isModalOpen = false;
    };

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
        console.log('saveing');
        console.log('rows: ', rows)
        for (let i = 0; i < rows.length; i++) {
            let row = rows[i];
            let json = row.to_json();
            console.log('row json: ', json)
            var myHeaders = new Headers();
            myHeaders.append("Content-Type", "application/json");
            let token = window.localStorage.getItem('token');
            myHeaders.append("Authorization", "Bearer " + token);
            var raw = JSON.stringify(json);
            var requestOptions = {
                method: 'PUT',
                headers: myHeaders,
                body: raw,
                redirect: 'follow'
            };
            let url = ENDPOINT_API + "svelte/api/productTabel/" + json.id + "/"
            fetch(url, requestOptions)
                .then(response => response.json())
                .then(result => {
                    debugger;
                    console.log("result of " + url + " is: " + result);
                    if (result.code == 'token_not_valid') {
                        notifier.danger('token_not_valid', notification_timeout);

                        openModal();
                    } else {
                        notifier.success(`POST to ${url}, ${JSON.stringify(result, null, 2)}`, 10000);
                    }
                })
                .catch(error => {
                    console.log('error', error)
                    notifier.danger(`POST to ${url} failed`, notification_timeout);
                });
        }



    }

    function handle_login() {
        var myHeaders = new Headers();
        let url = ENDPOINT_API + "api/token/";
        var formdata = new FormData();
        formdata.append("username", username);
        formdata.append("password", password);
        var requestOptions = {
            method: 'POST',
            headers: myHeaders,
            body: formdata,
            redirect: 'follow'
        };
        fetch(url, requestOptions)
            .then(response => response.json())
            .then(result => {
                debugger;
                console.log("result of " + url + " is: " + result);
                if(result.access) {
                    notifier.success(`success obtaining token ${result.access}`, notification_timeout);
                    window.localStorage.setItem('token', result.access);
                }else {
                    notifier.danger(`failed to get token: ${result}`, notification_timeout);
                }
            })
            .catch(error => {
                console.log('error', error)
                notifier.danger(`POST to ${url} failed`, notification_timeout);
            });
    }
    let n;
</script>

<DialogOverlay isOpen={isModalOpen} onDismiss={closeModal}>
    <DialogContent aria-label="Announcement">
        <button on:click={closeModal}>Close</button>
        <form id="login_form" action="{ENDPOINT_API + 'api/token/'}" method="POST">
            <label for="username">שם משתמש: </label>
            <input type="text" name="username" bind:value={username}>
            <label for="password">סיסמא: </label>
            <input type="password" name="password" bind:value={password}>
            <button type="button" on:click|preventDefault={handle_login}>Get Token</button>
        </form>
    </DialogContent>
</DialogOverlay>

<main>
    <img class="sm-img" src={$productStore.image} alt={$productStore.title} />
    <h1>{$productStore.title}</h1>
    {#if $productStore.detailTabel}
        <table>
            <tr>
                <th>id</th>
                <th>ספק</th>
                <th>מידות</th>
                <th>צבעים</th>
                <th>מחיר עלות</th>
                <th>מחיר לחנות</th>
                <th>מחיר מומלץ</th>
                <th>מקט ספק</th>
            </tr>
            {#each $productStore.detailTabel as entryId, i}
                <TableRow entry_id={entryId} bind:this={rows[i]}/>
            {/each}
        </table>
        <button type="button" on:click={saveToServer}>שמור שינויים בתת טבלה</button>
    {/if}
</main>
<NotificationDisplay bind:this={n} />
<style lang="scss">
    .sm-img{
        width:150px;
    }
    main {
        direction: rtl;
    }
    th {
        font-size: x-large;
        text-decoration: underline;
        font-weight: bold;
        border-bottom: 1px solid black;
    }
    table, tr {
        text-align: center;
        border-bottom: 1px solid black;
        max-width: 100vw;
    }
    table tr {
        border-bottom: 1px solid black;
    }

    #login_form {
        display:flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        direction:rtl;
    }
</style>