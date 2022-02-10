<template>
    <div id="app" class="app">
        <dm-pdf-viewer :name="'document'" :url="'./docs.pdf'"/>
    </div>
</template>

<script>
import DmPdfViewer from './DmPdfViewer.vue'
    export default {
        name: 'App',
        components: {DmPdfViewer}
    }
</script>

<style scoped>
.app {
    margin: 0 auto;
    width: 15vw;
    height: 40vh;
}
</style>