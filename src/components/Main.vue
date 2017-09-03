<template>
    <div class='main'>

        <nav class='navbar navbar-toggleable navbar-inverse bg-inverse'>
            <a class='navbar-brand' href='#'>Travelmap Data Generator</a>
            <button class='btn navar-buttons btn-outline-info' type='submit' v-on:click="saveButtonClick()">Save</button>
            <button class='btn navar-buttons btn-outline-secondary' type='submit' v-on:click="showLoadModal = true">Load</button>
        </nav>

        <div v-if="showLoadModal" class="modalBackgdrop">
            <div class="modal fade">
                <div class="modal-dialog">
                    <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Load .json-File</h5>
                        <button type="button" class="close" v-on:click="showLoadModal = false">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <p>Please select a .json-File.</p>

                        <form id="jsonFile" name="jsonFile" enctype="multipart/form-data" method="post">
                            <fieldset>
                                <input type='file' id='fileinput'>
                            </fieldset>
                        </form>
                    </div>

                    <div class="modal-footer">
                        <button type="button" class="btn btn-info" v-on:click="loadFile()">Load</button>
                    </div>
                    </div>
                </div>
            </div>
        </div>

        <div class='row panelRow'>
            <div class='col-2 panel placeListPanel'>
                <h3>Places - List</h3>

                <h6 v-if="travelData.places.length === 0">Load a .json-File or add a new place via the button at the bottom!</h6>

                <div class="placeListWrapper">
                    <div class="list-group">
                        <draggable v-model="travelData.places" @start="drag=true" @end="drag=false">
                            <div v-on:click="listPlaceClicked(place)" v-bind:key="index" v-for="(place, index) in travelData.places" class="list-group-item list-group-item-action flex-column align-items-start placeListItem">
                                <div class="d-flex w-100 justify-content-between">
                                    <h5 class="mb-1">{{ place.name }}</h5>
                                    <small># {{ index + 1 }}</small>
                                </div>
                                <p class="mb-1 placeListDescription">{{ place.description }}</p>
                                <span class="badge badge-info">ID: {{ place.id }}</span>
                            </div>
                        </draggable>
                    </div>
                </div>

                <button class='btn floating-button btn-info btn-lg' type='submit' v-on:click="createNewListPlace()" >+</button>
            </div>
            <div class='col-3 panel editPlacePanel'>
                <h3>Edit Place</h3>

                <h6 v-if="!selectedPlace">Select a place on the left to edit it!</h6>

                <form v-if="selectedPlace">
                    <div class="form-group">
                        <label for="idInput">ID</label>
                        <input type="text" class="form-control" id="idInput" placeholder="Enter place ID" v-model="selectedPlace.id">
                        <small id="emailHelp" class="form-text text-muted">This will uniquely identify the place (e.g. for the URL)</small>
                    </div>

                    <div class="form-group">
                        <label for="nameInput">Name</label>
                        <input type="text" class="form-control" id="nameInput" placeholder="Enter name of the place" v-model="selectedPlace.name">
                        <small id="emailHelp" class="form-text text-muted">Can contain spaces, special characters etc.</small>
                    </div>

                    <div class="form-group">
                        <label for="descriptionTextArea">Description</label>
                        <textarea class="form-control" id="descriptionTextArea" rows="20" placeholder="Enter a description for the place" v-model="selectedPlace.description"></textarea>
                    </div>

                    <div class="form-group">
                        <label for="latitudeInput">Latitude & Longitude</label>
                        <input type="text" class="form-control" id="latitudeInput" placeholder="Enter latitude" v-model="selectedPlace.coordinates.lat">
                        <input type="text" class="form-control" id="longitudeInput" placeholder="Enter longitude" v-model="selectedPlace.coordinates.lng">
                        <small id="emailHelp" class="form-text text-muted">Of geographic location</small>
                    </div>

                    <br>
                    <br>
                    <button type="button" class="btn btn-block btn-outline-danger" v-on:click="deleteSelectedPlace()">Delete this place</button>

                </form>

            </div>
            <div class='col-7 panel placePhotosPanel'>
                <h3>Place - Photos</h3>

                <h6 v-if="!selectedPlace">Select a place on the left to add/edit photos!</h6>

                <div v-if="selectedPlace" class="photoTableContainer">
                    <table v-if="selectedPlace" class="table table-striped table-bordered table-sm">
                        <thead class="thead-inverse">
                            <tr>
                                <th class="text-center">#</th>
                                <th class="text-center">Preview</th>
                                <th class="text-center">File Name</th>
                                <th class="text-center">Description</th>
                                <th class="text-center">Image / Video</th>
                                <th class="text-center">Delete</th>
                            </tr>
                        </thead>
                        <draggable :element="'tbody'" v-model="selectedPlace.photos" @start="drag=true" @end="drag=false">
                            <tr class="tableRowPhoto" v-bind:key="index" v-for="(photo, index) in selectedPlace.photos">
                                <td class="text-center">{{ index + 1 }}</th>
                                <td class="text-center">
                                    <span v-if="!photo.base64data">N/A</span>
                                    <img v-if="photo.base64data" v-bind:src="photo.base64data"></img>

                                </td>
                                <td class="text-center">{{ photo.fileName }}</td>
                                <td class="text-center">
                                    <input type="text" class="form-control" id="idInput" placeholder="Enter description" v-model="photo.description">
                                </td>

                                <td class="text-center">
                                    <select class="custom-select" v-model="photo.type">
                                        <option>image</option>
                                        <option>video</option>
                                    </select>
                                </td>

                                <td class="text-center">
                                    <button v-on:click="deletePhoto(index)" type="button" class="btn btn-outline-danger">X</button>
                                </td>
                            </tr>
                        </draggable>
                    </table>
                </div>

                <!-- <span>{{ JSON.stringify(this.travelData) }}</span> -->

                <!-- <div class='photoDropOff'>
                    <h4 class='photoDropOffText'>Drag & Drop Pictures/Videos here to add them to this place!</h4>
                </div> -->
                <div v-if="selectedPlace" class="photoDrop">
                    <div class="photoDropInputWrapper">
                        <span>Select or Drag/Drop Photos & Videos <b>here</b> to add them to this place!</span>
                        <input @change="handlePhotoDrop" class="photoDropInput" type="file" multiple>
                    </div>
                </div>

            </div>
        </div>

        <a id="downloadAnchorElem" style="display:none"></a>
    </div>
</template>

<script>
    import draggable from 'vuedraggable'

    export default {

        name: 'main',
        components: {draggable},
        data () {
            return {
                imageSrc: undefined,
                selectedPlace: undefined,
                showLoadModal: false,
                travelData: {
                    places: []
                }
            }
        },
        created: function () {

        },
        methods: {
            saveButtonClick: function () {
                const dataStr = 'data:text/json;charset=utf-8,' + encodeURIComponent(JSON.stringify(this.travelData, null, 4))
                const dlAnchorElem = document.getElementById('downloadAnchorElem')
                dlAnchorElem.setAttribute('href', dataStr)
                dlAnchorElem.setAttribute('download', 'travelData.json')
                dlAnchorElem.click()
            },
            loadFile: function () {
                let file
                let fileReader

                if (typeof window.FileReader !== 'function') {
                    alert("The file API isn't supported on this browser yet.")

                    return
                }

                const input = document.getElementById('fileinput')
                if (!input) {
                    alert("Couldn't find the fileinput element.")
                } else if (!input.files) {
                    alert("This browser doesn't seem to support the `files` property of file inputs.")
                } else if (!input.files[0]) {
                    alert("Please select a file before clicking 'Load'")
                } else {
                    file = input.files[0]
                    fileReader = new FileReader()
                    fileReader.onload = this.receivedText
                    fileReader.readAsText(file)

                    this.showLoadModal = false
                }
            },
            receivedText: function (e) {
                const lines = e.target.result
                this.travelData = JSON.parse(lines)
            },
            listPlaceClicked: function (place) {
                this.selectedPlace = place
            },
            deletePhoto: function (index) {
                this.selectedPlace.photos.splice(index, 1)
            },
            handlePhotoDrop: function (event) {
                const input = event.target

                for (let i = 0; i < input.files.length; i++) {
                    const file = input.files[i]

                    const reader = new FileReader()

                    const vm = this

                    reader.onload = function (e) {
                        const newImg = document.createElement('img')
                        newImg.src = e.target.result

                        newImg.onload = function () {
                            const imgData = vm.resizedataURL(newImg, 100, 100)
                            vm.selectedPlace.photos.push({
                                fileName: file.name,
                                description: file.name,
                                type: 'image',
                                base64data: imgData
                            })
                        }
                    }

                    reader.readAsDataURL(file)
                }
            },
            resizedataURL: function (img, maxWidth, maxHeight) {
                const canvas = document.createElement('canvas')
                const ctx = canvas.getContext('2d')

                if (img.width > img.height) {
                    const verh = img.height / img.width
                    img.width = 100
                    img.height = 100 * verh
                    canvas.width = 100
                    canvas.height = 100 * verh
                }

                if (img.height > img.width) {
                    const verh = img.width / img.height
                    img.width = 100 * verh
                    img.height = 100
                    canvas.width = 100 * verh
                    canvas.height = 100
                }

                ctx.drawImage(img, 0, 0, maxWidth, maxHeight)

                return canvas.toDataURL()
            },
            createNewListPlace: function () {
                this.travelData.places.push({
                    id: '',
                    name: '',
                    description: '',
                    coordinates: {
                        lat: null,
                        lng: null
                    },
                    photos: []
                })
            },
            deleteSelectedPlace: function () {
                for (let i = 0; i < this.travelData.places.length; i++) {
                    const place = this.travelData.places[i]

                    if (place === this.selectedPlace) {
                        this.travelData.places.splice(i, 1)

                        break
                    }
                }
                this.selectedPlace = undefined
            }
        }
    }
</script>

<style scoped>

.photoDrop {
  display: flex;
  height: 10em;
  padding-top: 1em;
  border-radius: 40px;
}

.photoDropInputWrapper {
  overflow: hidden;
  position: relative;
  background: #eee;
  border-radius: 1px;
  float: left;
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  color: rgba(0, 0, 0, 0.2);
  transition: 0.4s background;
  border-radius: 40px;
  border: 0.3em dashed white;
}

.photoDropInputWrapper:hover {
  background: #e0e0e0;
}

.photoDropInput {
  cursor: inherit;
  display: block;
  font-size: 999px;
  min-height: 100%;
  opacity: 0;
  position: absolute;
  right: 0;
  text-align: right;
  top: 0;
  cursor: pointer;
}

h3 {
    margin-bottom: 0.5em;
}

.photoTableContainer {
    /* background-color: red; */
    width: 100%;
    height: calc(100% - 19em);
    overflow-y: scroll;
}

.table > thead > tr > th {
     vertical-align: middle;
}

.table > tbody > tr > td {
     vertical-align: middle;
}

.fade {
    opacity: 1;
    background-color: rgba(0,0,0,0) !important;
}

.modal {
    position: absolute;
    top: calc(50% - 200px);
    left: 0;
    display: inline-block;
    overflow: visible;
}

.modalBackgdrop {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: rgba(0,0,0,0.75);
    z-index: 1000;
}

.btn {
    cursor: pointer;
}

.placeListWrapper {
    overflow-y: auto;
    max-height: calc(100vh - 15em);
}

.panelRow {
    height: calc(100vh - 54px);
    width: 100%;
    margin-left: 0px;
    margin-right: 0px;
}

.panel {
    padding-top: 1em;
}

.navar-buttons {
    margin-left: 4em;
    margin-right: 4em;
}

.floating-button {
    position: absolute;
    bottom: 2em;
    left: 10%;
    width: 80%;
}

.placeListItem {
    cursor: pointer;
}

.tableRowPhoto {
    cursor: pointer;
}

.placeListDescription {
    text-overflow: ellipsis;
    overflow: hidden;
    width: 100%;
    white-space: nowrap;
}

.arrowSpan {
    font-size: 2em;
}

.placeListPanel {
    /* background-color: #c2dfb8; */
    position: relative;
}
.editPlacePanel {
    /* background-color: #efbfa3; */
    border-left: 1px solid #292b2c;
    border-right: 1px solid #292b2c;
}
.placePhotosPanel {
    /* background-color: #cdbded; */
    position: relative;
}

.photoDropOff {
    position: absolute;
    bottom: 1em;
    right: 0;

    height: 12em;
    width: calc(100% - 2em);
    margin-right: 1em;

    background-color: rgba(0, 0, 0, 0.2);
    border: 4px dashed black;
    border-radius: 30px;

    padding: 1em;

    line-height: 10em;
    text-align: center;
}

.photoDropOffText {
    display: inline-block;
    vertical-align: middle;
    line-height: 35px;
}

</style>
