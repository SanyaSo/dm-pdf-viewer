# Pinch zoom for Vue

vue library for viewing pdf files based on pdfJs 

## Installation

Install the npm package.

	npm i dm-pdf-viewer

Import module:

	import DmPdfViewer from 'dm-pdf-viewer';

	Vue.component('dm-pdf-viewer', DmPdfViewer);

## Usage
Pass the url to the component and it will render the pdf

	<dm-pdf-viewer name="document" url="./doc.pdf" />

## Properties

| name             | type            | default | description                                 |
|------------------|-----------------|---------|---------------------------------------------|
| name | string       | ''     | Document title.|
| url       | string | '' | Document url |
| zoom         | boolean          | true       | Zoom buttons |
| print    | boolean         | true   | Print button|
| download       | boolean         | true    | Download button|
| fullscreen         | boolean         | true   | fullscreen button |
| backgroundColor       | string         | '#fff'   | The background color of the container. |
| toolbarItemColor         | string | '#5956e0' | The color of toolbar items |
