<template>
<div class="container">

    <div class="row">

        <div class="col-md-7">

            <h2>Camera test</h2>
            <div class="col-md-12">
                <button type="button" class="btn btn-primary" @click="onCapture">CapturePhoto</button>
                <button type="button" class="btn btn-danger" @click="onStop">Stop Camera</button>
                <button type="button" class="btn btn-success" @click="onStart">Start Camera</button>
                <button type="button" class="btn btn-primary" @click="submitImage">Send Photo Analyzed</button>
            </div>
            <div class="col-md-12">
                <select v-model="camera">
                    <option>-- Select Device --</option>
                    <option v-for="device in devices" :key="device.deviceId" :value="device.deviceId">{{ device.label }}</option>
                </select>
            </div>
            <code v-if="device"></code>
            <div class="border">
                <vue-web-cam ref="webcam" :device-id="deviceId" width="90%" height="50%" @stopped="onStopped" @error="onError" @cameras="onCameras" @camera-change="onCameraChange" />
            </div>

        </div>
        <!--Display Image  -->
        <div class="col-md-5">
            <h2>Captured Image</h2>
            <b-container fluid class="bg-dark">
                <b-row>
                    <b-col>
                        <b-img :src="img"></b-img>
                    </b-col>

                </b-row>
            </b-container>
            <b-container fluid class="p-4 my-4 bg-info">
                <h2>Analyzed Results</h2>
                <b-row>
                    <b-col>
                        <div v-for="(item, index) in items.detected_objects" :key="index">
                            <b-progress show-progress :value="item.confidence*100" variant="success" striped :animated="true"></b-progress>
                            <div>
                                {{item.parent}}
                            </div>
                        </div>
                    </b-col>
                </b-row>
            </b-container>

        </div>

    </div>
    <div class="row-my-4">
        <div class="col-md-12 bg-info text-center">
            <h1>Response</h1>
            <p>{{items}}</p>
        </div>
    </div>
</div>
</template>

<script>
export default {
    name: "Cameratest",

    data() {
        return {
            img: null,
            camera: null,
            deviceId: null,
            devices: [],
            items: {},

        };
    },
    computed: {
        device: function () {
            return this.devices.find(n => n.deviceId === this.deviceId);
        },
        decodeBase64() {

            try {
                let x = this.img.split("base64,");
                return x[1]

            } catch (error) {
                return null

            }

        }
    },
    watch: {
        camera: function (id) {
            this.deviceId = id;
        },
        devices: function () {
            // Once we have a list select the first one
            const [first, ...tail] = this.devices;
            if (first) {
                this.camera = first.deviceId;
                this.deviceId = first.deviceId;
            }
        }
    },
    methods: {
        onCapture() {
            this.img = this.$refs.webcam.capture();
        },
        onStarted(stream) {
            console.log("On Started Event", stream);
        },
        onStopped(stream) {
            console.log("On Stopped Event", stream);
        },
        onStop() {
            this.$refs.webcam.stop();
        },
        onStart() {
            this.$refs.webcam.start();
        },
        onError(error) {
            console.log("On Error Event", error);
        },
        onCameras(cameras) {
            this.devices = cameras;
            console.log("On Cameras Event", cameras);
        },
        onCameraChange(deviceId) {
            this.deviceId = deviceId;
            this.camera = deviceId;
            console.log("On Camera Change Event", deviceId);
        },
        async submitImage() {
            this.items = await this.$axios({
                    method: "post",
                    url: "https://nvision.nipa.cloud/api/v1/object-detection",
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': 'ApiKey cdb29f355cb4059995e05420dc8d963f657898bf3a5f2f5e7a88c58279f5e4a0a1c4c4cf874594b42e413fc45c425425ac'
                    },
                    data: {
                        "configurations": [{
                                "parameter": "OutputCroppedImage",
                                "value": "false"
                            },
                            {
                                "parameter": "ConfidenceThreshold",
                                "value": "0.2"
                            },
                            // {
                            //     "parameter": "OutputVisualizedImage",
                            //     "value": "false"
                            // }
                        ],
                        "raw_data": this.decodeBase64
                    }
                })
                .then((response) => {
                    return response.data
                })
                .catch((error) => {
                    return {

                    }
                });
        },

    }
};
</script>

<style scoped>
img {
    margin: 5px;
    padding: 5px;
    border-radius: 4px;
    width: 400px;
}
</style>
