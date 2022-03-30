<template>
    <div
        @webkitfullscreenchange="onPdfFullScreenChange"
        @fullscreenchange="onPdfFullScreenChange"
        id="pdf-document"
        class="pdf-document">
        <div class="viewer-header">
            <div class="header-toolbar">
                <div class="header-toolbar__name">
                    <span> {{ pdfName }} </span>
                </div>
                <div class="header-toolbar__items">
                    <span
                        v-if="!isEdit && edit"
                        @click="editFile"
                        :style="{color: toolbarItemColor}"
                        class="header-toolbar__item header-toolbar__icon_edit"/>
                    <span
                        class="header-toolbar__item header-toolbar__icon_fullscreen header-toolbar__icon_desktop"
                        v-if="!isFullScreen && fullscreen"
                        :style="{color: toolbarItemColor}"
                        @click="openFullScreen"/>
                    <span
                        v-if="isFullScreen && fullscreen"
                        @click="closeFullScreen"
                        :style="{color: toolbarItemColor}"
                        class="header-toolbar__item header-toolbar__icon_fullscreen-exit header-toolbar__icon_desktop"/>
                    <span
                        v-if="isFullScreen && zoom"
                        :style="{color: toolbarItemColor}"
                        class="header-toolbar__item header-toolbar__icon_zoomIn"
                        @click="zoomIn"/>
                    <span
                        v-if="isFullScreen && zoom"
                        :style="{color: toolbarItemColor}"
                        class="header-toolbar__item header-toolbar__icon_zoomOut"
                        @click="zoomOut"/>
                    <span
                        v-if="isImportPrintJs && print"
                        @click="printPdf"
                        :style="{color: toolbarItemColor}"
                        class="header-toolbar__item header-toolbar__icon_print header-toolbar__icon_desktop"
                        src="@/assets/icons/print.svg"/>
                    <span
                        v-if="download && !selectiveDownload"
                        @click="downloadPdf"
                        :style="{color: toolbarItemColor}"
                        class="header-toolbar__item header-toolbar__icon_download"/>
                    <span
                        v-if="download && selectiveDownload"
                        @click="downloadButtonsVisible = !downloadButtonsVisible"
                        :style="{color: toolbarItemColor}"
                        class="header-toolbar__item header-toolbar__icon_download"/>
                    <div v-if="downloadButtonsVisible" class="download_btns">
                        <span @click="downloadOrig">{{ downloadOrigTitle }}</span>
                        <span @click="downloadPdf">{{ downloadPdfTitle }}</span>
                    </div>
                </div>
            </div>
        </div>
        <div
            class="viewer-content"
            id="pdf-document-content"
            :style="{background: backgroundColor}">
            <pinch-zoom
                disable-zoom-control="disable"
                :background-color="backgroundColor"
                :wheel="false"
                overflow="visible">
                <div class="pdf-canvas-wraper" id="pdf-canvas-wraper"/>
            </pinch-zoom>
        </div>
        <div v-if="footer" class="viewer-footer"/>
    </div>
</template>

<script>
    import * as pdfjsLib from "pdfjs-dist/legacy/build/pdf";
    import PDFJSWorker from 'pdfjs-dist/legacy/build/pdf.worker.entry'
    import PinchZoom from 'vue-pinch-zoom'

    pdfjsLib.GlobalWorkerOptions.workerSrc = PDFJSWorker
    export default {
        components: {
            PinchZoom
        },
        props: {
            name: {
                type: String,
                default: ''
            },
            docId: {
                type: String,
                default: null
            },
            fileIndex: {
                type: [String, Number],
                default: 0
            },
            docFile: {
                type: Object,
                default: null
            },
            file: {},
            url: {
                type: String,
                default: ''
            },
            filePath: {
                type: String,
                default: ''
            },
            selectiveDownload: {
                type: Boolean,
                default: false
            },
            downloadOrigTitle: {
                type: String,
                default: 'Download'
            },
            downloadPdfTitle: {
                type: String,
                default: 'Download as PDF'
            },
            zoom: {
                type: Boolean,
                default: true
            },
            print: {
                type: Boolean,
                default: true
            },
            download: {
                type: Boolean,
                default: true
            },
            fullscreen: {
                type: Boolean,
                default: true
            },
            footer: {
                type: Boolean,
                default: false
            },
            backgroundColor: {
                type: String,
                default: '#fff'
            },
            toolbarItemColor: {
                type: String,
                default: '#5956e0'
            },
            edit: {
                type: Boolean,
                default: false
            }
        },
        data() {
            return {
                pdfViewer: null,
                scale: 1,
                isFullScreen: false,
                isImportPrintJs: false,
                downloadButtonsVisible: false,
                isEdit: false
            }
        },
        computed: {
            pdfName() {
                return this.name ? this.name?.split('.').slice(0, -1).join('.') : null;
            },
        },
        mounted() {
            if (this.url) {
                this.generatedPdfDocument(this.url)
            }
            import('print-js').then(() => {
                this.isImportPrintJs = true
            })
            if (!this.includesOneOf(this.name.toLowerCase())) {
                this.isEdit = true
            }
        },
        methods: {
            async onPdfFullScreenChange(event) {
                this.isFullScreen = document.fullscreenElement !== null
                this.isFullScreen = document.webkitFullscreenElement !== null
                const pdfWraper = document.getElementById('pdf-document-content')
                const calcScale = (pdfWraper.offsetWidth * 0.59) / 373

                if (this.scale > calcScale || this.scale < calcScale) {
                    if (pdfWraper.offsetWidth < 1200 && pdfWraper.offsetWidth > 768) {
                        this.scale = calcScale + 1
                    } else {
                        this.scale = 1.5
                    }
                    for (let i = 1; i <= this.pdfViewer.numPages; i++) {
                        const page = await this.pdfViewer.getPage(i)
                        await this.zoomRenderPage(page, this.scale)
                    }
                }
            },
            async generatedPdfDocument(link) {
                const loadingTask = pdfjsLib.getDocument(link)
                pdfjsLib.disableWorker = true
                const pdf = await loadingTask.promise
                this.pdfViewer = pdf
                await this.renderAllPage(pdf)
            },
            async renderPage(page, scale) {
                const wrapper = document.getElementById('pdf-canvas-wraper')
                const canvas = document.createElement('canvas')
                canvas.classList.toggle('pdf-canvas')
                const context = canvas.getContext('2d')
                context.clearRect(0, 0, canvas.width, canvas.height)
                const viewport = page.getViewport({scale})
                canvas.height = viewport.height
                canvas.width = viewport.width
                const renderContext = {
                    canvasContext: context,
                    viewport
                }
                wrapper.appendChild(canvas)
                await page.render(renderContext)
            },
            async zoomRenderPage(page, scale) {
                const wrapper = document.getElementById('pdf-canvas-wraper')
                const canvas = document.querySelector('.pdf-canvas')
                const context = canvas.getContext('2d')
                context.clearRect(0, 0, canvas.width, canvas.height)
                const viewport = page.getViewport({scale})
                canvas.height = viewport.height
                canvas.width = viewport.width
                const renderContext = {
                    canvasContext: context,
                    viewport
                }
                wrapper.appendChild(canvas)
                await page.render(renderContext)
            },
            async renderAllPage(pdf) {
                const pdfWraper = document.getElementById('pdf-document-content')
                let scale
                if (pdfWraper.offsetWidth < 768) {
                    scale = 2.3
                } else if (pdfWraper.offsetWidth < 1200 && pdfWraper.offsetWidth > 768) {
                    scale = (pdfWraper.offsetWidth * 0.59) / 373 + 1
                } else {
                    scale = 1.6
                }
                this.scale = scale
                for (let i = 1; i <= pdf.numPages; i++) {
                    const page = await pdf.getPage(i)
                    await this.renderPage(page, scale)
                }
            },
            async zoomIn() {
                if (this.scale >= 2.5) {
                    return
                }
                this.scale = this.scale + 0.4
                for (let i = 1; i <= this.pdfViewer.numPages; i++) {
                    const page = await this.pdfViewer.getPage(i)
                    await this.zoomRenderPage(page, this.scale)
                }
            },
            async zoomOut() {
                if (this.scale <= 0.8) {
                    return
                }
                this.scale = this.scale - 0.4
                for (let i = 1; i <= this.pdfViewer.numPages; i++) {
                    const page = await this.pdfViewer.getPage(i)
                    await this.zoomRenderPage(page, this.scale)
                }
            },
            openFullScreen() {
                const pdf = document.getElementById('pdf-document')
                if (pdf.requestFullscreen) {
                    pdf.requestFullscreen()
                } else if (pdf.webkitRequestFullscreen) {
                    pdf.webkitRequestFullscreen()
                } else if (pdf.mozRequestFullscreen) {
                    pdf.mozRequestFullScreen()
                }
            },
            closeFullScreen() {
                if (document.exitFullscreen) {
                    document.exitFullscreen()
                } else if (document.webkitExitFullscreen) {
                    document.webkitExitFullscreen()
                } else if (document.mozExitFullscreen) {
                    document.mozExitFullscreen()
                }
            },
            printPdf() {
                printJS(this.url)
            },
            downloadPdf() {
                const a = document.createElement('a')
                a.href = this.url
                a.download = this.name
                a.click()
            },
            downloadOrig() {
                this.$emit("download-orig", this.filePath);
            },
            editFile() {
                let docType = 'main';
                let docId = this.docId;
                let index = this.fileIndex;
                let docFile = this.docFile;

                if (this.docFile.path.includes('files:files')) {
                    docType = 'attachment';
                    index = index - 1;
                }

                let digest = docFile.digest

                let route = this.$router.resolve({
                    path: `/onlyoffice/${docId}`, query: {
                        mode: 'edit',
                        index: index,
                        digest: digest,
                        docType: docType
                    }
                });

                window.open(route.href, '_blank');
            },
            includesOneOf(val) {
                let arrayOfVal = ['doc','pptx','xls','txt'];
                return arrayOfVal.some(str => val.includes(str));
            },
        }
    }
</script>

<style lang="scss">
    .pdf-document {
        box-shadow: 0px 0px 12px 0px rgba(51, 51, 61, 0.08);
        height: 100%;

        .viewer-header {
            background: #fff;
            display: flex;
            align-items: center;
            justify-content: space-between;
            box-shadow: 0px 0px 12px 0px rgba(51, 51, 61, 0.08);
            border-radius: 12px 12px 0 0;
            padding: 16px 24px 16px 24px;
            position: relative;
            z-index: 20;
            @media (max-width: 768px) {
                padding: 13px 16px 13px 16px;
            }

            .header-toolbar {
                width: 898px;
                margin: 0 auto;
                display: flex;
                align-items: center;
                justify-content: space-between;

                &__name span {
                    font-size: 18px;
                    @media (max-width: 768px) {
                        font-size: 14px;
                    }
                    font-weight: bold;
                }

                &__items {
                    display: flex;
                    align-items: center;
                }

                &__item {
                    cursor: pointer;
                }

                &__icon {
                    &_desktop {
                        display: none;
                        @media (min-width: 768px) {
                            display: block;
                        }
                    }

                    &_fullscreen {
                        -webkit-mask-image: url('../assets/icons/fullscreen.svg');
                        mask-image: url('../assets/icons/fullscreen.svg');
                        width: 18px;
                        height: 18px;
                        -webkit-mask-size: cover;
                        mask-size: cover;
                        -webkit-mask-position: 50% 50%;
                        mask-position: 50% 50%;
                        margin-right: 1rem;
                        background: currentColor;
                        @media (min-width: 768px) {
                            display: inline-block;
                        }
                    }

                    &_fullscreen-exit {
                        -webkit-mask-image: url('../assets/icons/fullscreen_exit.svg');
                        mask-image: url('../assets/icons/fullscreen_exit.svg');
                        width: 24px;
                        height: 24px;
                        -webkit-mask-size: cover;
                        mask-size: cover;
                        -webkit-mask-position: 50% 50%;
                        mask-position: 50% 50%;
                        margin-right: 1rem;
                        background: currentColor;
                        display: inline-block;
                    }

                    &_zoomIn {
                        -webkit-mask-image: url('../assets/icons/zoomIn.svg');
                        mask-image: url('../assets/icons/zoomIn.svg');
                        width: 24px;
                        height: 24px;
                        -webkit-mask-size: cover;
                        mask-size: cover;
                        -webkit-mask-position: 50% 50%;
                        mask-position: 50% 50%;
                        margin-right: 1rem;
                        background: currentColor;
                        display: inline-block;
                    }

                    &_zoomOut {
                        -webkit-mask-image: url('../assets/icons/zoomOut.svg');
                        mask-image: url('../assets/icons/zoomOut.svg');
                        width: 24px;
                        height: 24px;
                        -webkit-mask-size: cover;
                        mask-size: cover;
                        -webkit-mask-position: 50% 50%;
                        mask-position: 50% 50%;
                        margin-right: 1rem;
                        background: currentColor;
                        display: inline-block;
                    }

                    &_print {
                        -webkit-mask-image: url('../assets/icons/print.svg');
                        mask-image: url('../assets/icons/print.svg');
                        width: 18px;
                        height: 18px;
                        -webkit-mask-size: cover;
                        mask-size: cover;
                        -webkit-mask-position: 50% 50%;
                        mask-position: 50% 50%;
                        background: currentColor;
                        margin-right: 0.8rem;
                        @media (min-width: 768px) {
                            display: inline-block;
                        }
                    }

                    &_download {
                        -webkit-mask-image: url('../assets/icons/download.svg');
                        mask-image: url('../assets/icons/download.svg');
                        width: 24px;
                        height: 24px;
                        -webkit-mask-size: cover;
                        mask-size: cover;
                        -webkit-mask-position: 50% 50%;
                        mask-position: 50% 50%;
                        background: currentColor;
                        display: inline-block;
                    }

                    &_edit {
                        -webkit-mask-image: url('../assets/icons/edit.svg');
                        mask-image: url('../assets/icons/edit.svg');
                        width: 21px;
                        height: 21px;
                        -webkit-mask-size: cover;
                        mask-size: cover;
                        -webkit-mask-position: 50% 50%;
                        mask-position: 50% 50%;
                        margin-right: 1rem;
                        background: currentColor;
                        display: inline-block;
                    }
                }
                .download_btns {
                    display: inline-block;
                    position: absolute;
                    top: 45px;
                    right: 5px;
                    span {
                        padding: 5px;
                        display: block;
                        background: #fff;
                        &:hover {
                            cursor: pointer;
                            background: #edebeb;
                        }
                    }
                }
            }
        }

        .viewer-content {
            width: 100%;
            height: 100%;
            overflow-y: scroll;
            border-radius: 0px 0px 12px 12px;

            .pdf-canvas-wraper {
                .pdf-canvas {
                    display: block;
                    box-shadow: 0px 0px 12px 0px rgba(51, 51, 61, 0.08);
                    margin: 0 auto;
                    width: 100% !important;
                    @media (max-width: 768px) {
                        width: 100% !important;
                    }
                }
            }
        }

        .viewer-footer {
            background: #fff;
            box-shadow: 0px 0px 12px 0px rgba(51, 51, 61, 0.08);
            border-radius: 0 0 12px 12px;
            height: 56px;
            position: relative;
            z-index: 20;
            @media (max-width: 768px) {
                height: 50px;
            }
        }
    }
</style>
