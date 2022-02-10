<template>
    <div id="app" class="app">
        <dm-pdf-viewer :name="'document'" :footer="true" :url="'./docs.pdf'"/>
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
    width: 50vw;
    height: 60vh;
}
</style>