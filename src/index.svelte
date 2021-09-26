<svelte:options tag="svelte-custom-element" />

<script>
    import Prog from './prog.svelte'
    export let catalog_image_id;

</script>

<Prog catalog_image_id={catalog_image_id}/>


<style global type="text/scss" lang="scss" src="./index.scss">
</style>
