<template>
    <div class='main'>

        <nav class='navbar navbar-toggleable navbar-inverse bg-inverse'>
            <a class='navbar-brand' href='#'>Travelmap Data Generator</a>
            <button class='btn navar-buttons btn-outline-info' type='submit' v-on:click="saveButtonClick()">Save</button>
            <button class='btn navar-buttons btn-outline-secondary' type='submit'>Load</button>
        </nav>

        <div class='row panelRow'>
            <div class='col-2 panel placeListPanel'>
                <h1>Places - List</h1>

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

                <button class='btn floating-button btn-info btn-lg' type='submit' v-on:click="listPlaceClicked(null)" >+</button>
            </div>
            <div class='col-3 panel editPlacePanel'>
                <h1>Edit Place</h1>

                <h2 v-if="!selectedPlace">Select a place on the left to edit it!</h2>

                <form v-if="selectedPlace">
                    <div class="form-group">
                        <label for="idInput">ID</label>
                        <input type="email" class="form-control" id="idInput" placeholder="Enter place ID" v-model="selectedPlace.id">
                        <small id="emailHelp" class="form-text text-muted">This will uniquely identify the place (e.g. for the URL)</small>
                    </div>

                    <div class="form-group">
                        <label for="nameInput">Name</label>
                        <input type="email" class="form-control" id="nameInput" placeholder="Enter name of the place" v-model="selectedPlace.name">
                        <small id="emailHelp" class="form-text text-muted">Can contain spaces, special characters etc.</small>
                    </div>

                    <div class="form-group">
                        <label for="descriptionTextArea">Description</label>
                        <textarea class="form-control" id="descriptionTextArea" rows="20" placeholder="Enter a description for the place" v-model="selectedPlace.description"></textarea>
                    </div>

                    <div class="form-group">
                        <label for="latitudeInput">Latitude & Longitude</label>
                        <input type="email" class="form-control" id="latitudeInput" placeholder="Enter latitude" v-model="selectedPlace.coordinates.lat">
                        <input type="email" class="form-control" id="longitudeInput" placeholder="Enter longitude" v-model="selectedPlace.coordinates.lng">
                        <small id="emailHelp" class="form-text text-muted">Of geographic location</small>
                    </div>
                </form>


            </div>
            <div class='col-7 panel placePhotosPanel'>
                <h1>Place - Photos</h1>

                <form id="jsonFile" name="jsonFile" enctype="multipart/form-data" method="post">
                    <fieldset>
                        <h2>Json File</h2>
                        <input type='file' id='fileinput'>
                        <input type='button' id='btnLoad' value='Load' v-on:click="loadFile()">
                    </fieldset>
                </form>

                <br>

                 <h2 v-if="!selectedPlace">Select a place on the left to add/edit photos!</h2>

                <table v-if="selectedPlace" class="table table-striped table-bordered table-sm table">
                    <thead class="thead-inverse">
                        <tr>
                            <th>#</th>
                            <th>Preview</th>
                            <th>File Name</th>
                            <th>Description</th>
                            <th>Image / Video</th>
                            <th>Delete</th>
                        </tr>
                    </thead>
                    <!-- <tbody> -->
                        <draggable :element="'tbody'" v-model="selectedPlace.photos" @start="drag=true" @end="drag=false">
                            <tr class="tableRowPhoto" v-bind:key="index" v-for="(photo, index) in selectedPlace.photos">
                                <th scope="row">{{ index + 1 }}</th>
                                <td>N/A</td>
                                <td>{{ photo.fileName }}</td>
                                <td>{{ photo.description }}</td>
                                <td>Image</td>
                                <td><button v-on:click="deletePhoto(index)" type="button" class="btn btn-outline-danger">X</button></td>
                            </tr>
                        </draggable>
                    <!-- </tbody> -->
                </table>

                <br>

                <span>{{ JSON.stringify(this.travelData) }}</span>
                <div class='photoDropOff'>
                    <h4 class='photoDropOffText'>Drag & Drop Pictures/Videos here to add them to this place!</h4>
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
                selectedPlace: undefined,
                travelData: {
                    places: [
                        {
                            id: 'sydney',
                            name: 'Sydney',
                            description: 'Sydney ist cool.',
                            coordinates: {
                                lat: -33.865143,
                                lng: 151.209900
                            },
                            photos: [
                                {
                                    fileName: 'DSC03096.jpg',
                                    description: 'Waterfall along the Kuranda Scenic Railway Trail'
                                },
                                {
                                    fileName: 'DSC00067.jpg',
                                    description: 'Sydney Opera House at dusk'
                                },
                                {
                                    fileName: 'DSC00120.jpg',
                                    description: 'Bondi Bay'
                                },
                                {
                                    fileName: 'DSC00012.jpg',
                                    description: 'St Marys Cathedral'
                                },
                                {
                                    fileName: 'DSC09946.jpg',
                                    description: 'Sydney Opera House Roof'
                                },
                                {
                                    fileName: 'DSC09906.jpg',
                                    description: 'Harbour Bridge'
                                },
                                {
                                    fileName: 'DSC_2038.JPG',
                                    description: 'DSC_2038.JPG'
                                }
                            ]
                        }
                    ]
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
                    alert("Um, couldn't find the fileinput element.")
                } else if (!input.files) {
                    alert("This browser doesn't seem to support the `files` property of file inputs.")
                } else if (!input.files[0]) {
                    alert("Please select a file before clicking 'Load'")
                } else {
                    file = input.files[0]
                    fileReader = new FileReader()
                    fileReader.onload = this.receivedText
                    fileReader.readAsText(file)
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
            }
        }
    }
</script>

<style scoped>

.btn {
    cursor: pointer;
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
    right: 2em;
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
