<template>
    <div class="background">
         

        <!-- <h1>Load Data</h1> -->
        <div class="main-container">
            
            <div class="import-container">
                <div class="container-heading">
                    Data Import
                </div>
                
                <div class="file-list">
                    <div v-for="(file, index) in load_files" :key="file.id">
                        <span>{{file.name}}</span> -
                        <!-- <span>{{file.size}}</span> - -->
                        <span>{{file.response}}</span> 
                        <span v-if="file.error">{{file.error}}</span>
                        <!-- <span v-else-if="file.success">success</span>
                        <span v-else-if="file.active">active</span>
                        <span v-else-if="file.active">active</span> -->
                        <!-- <span v-else></span> -->
                    </div>
                </div>

                <div class="load-button-container">
                    <span class="hoverable">
                    <file-upload
                        class="load_upload"
                        post-action="http://localhost:5000/upload/load_data"
                        extensions="gif,jpg,jpeg,png,webp, csv"
                        accept="image/png,image/gif,image/jpeg,image/webp, text/csv"
                        :multiple="true"
                        :size="1024 * 1024 * 10"
                        v-model="load_files"
                        @input-filter="inputFilter"
                        @input-file="inputFile"
                        ref="upload">
                        <div class="load-file-button">Select File</div>
                    </file-upload>
                    </span>
                    <div class="import-button" v-if="!$refs.upload || !$refs.upload.active" @click.prevent="$refs.upload.active = true">
                        Start Import
                    </div>
                    <div class="import-button importing" v-else @click.prevent="$refs.upload.active = false">
                        Stop Import
                    </div>
                </div>
            </div>
            <div class="separator">
                <font-awesome-icon class="fa-icon" icon="chevron-right" />  
            </div>
            <div class="available-container">
                <div class="container-heading">
                    Available Load Files
                </div>
                
                <div class="load-files-list-container">
                    <div class="load-files-list-item" v-for="item in files_lists.load_files_list">{{ item }}</div>
                </div>
            </div>
            
        </div>
        
    </div>
</template>

<script>
    // Note this is adapted from:
    // "https://github.com/lian-yue/vue-upload-component/blob/master/docs/views/examples/Simple.vue"

    import FileUpload from 'vue-upload-component';
    import moment from 'moment';
    export default {
        name: "ModelDataLoad",
        components: {
            FileUpload
        },

        data () {
            return {
                view_name: this.$options.name,
                model_page_name: "LoadUpload",
                is_connected: false,

                files_lists: {
                    solar_files_list: [],
                    load_files_list: [],
                },
                load_files: [],
            }
        },

        created() {
            if (this.model_page_name in this.$store.state.frontend_state) {
                this.input_data = this.$store.state.frontend_state[this.model_page_name]
            }
            this.get_solar_files();
            this.get_load_files();
        },

        methods: {

            get_solar_files() {
                console.log("Getting solar files");
                this.$socket.emit('get_solar_files')
            },

            get_load_files() {
                this.$socket.emit('get_load_files')
            },

            inputFilter(newFile, oldFile, prevent) {
                if (newFile && !oldFile) {
                    // Before adding a file
                    // Filter system files or hide files
                    if (/(\/|^)(Thumbs\.db|desktop\.ini|\..+)$/.test(newFile.name)) {
                        return prevent()
                    }
                    // Filter php html js file
                    if (/\.(php5?|html?|jsx?)$/i.test(newFile.name)) {
                        return prevent()
                    }
                }
            },

            inputFile(newFile, oldFile) {
                if (newFile && !oldFile) {
                    // add
                    console.log('add', newFile)
                }
                if (newFile && oldFile) {
                    // update
                    this.get_solar_files()
                    console.log('update', newFile)
                }
                if (!newFile && oldFile) {
                    // remove
                    console.log('remove', oldFile)
                }
            }
        },

        sockets: {
            filesChannel: function(response) {
                console.log('filesChannel Response', response)
                this.is_connected = true;
                this.files_lists[response.key] = response.data;
            },
            uploadsChannel: function(response) {
                console.log('HI THIS IS AN UPLOAD ALERT')
                alert(response)
            },
        }
    }
</script>

<style scoped>
    .background {

    }

    .main-container {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        animation-name: fade-in;
        animation-duration: 1s;
        
        height:100%;
        max-width:70vw;
    }

    .load-title {
        width: 100%;
        height: 10%;
        /*background-color: pink;*/
    }

    .load-button-container {
        background-color: #f8f8f8;

        display:flex;
        flex-direction:row;
        justify-content:space-around;
        align-items:center;
        height: 5vh;
        width: 30vw;
        
    }

    .load-file-button {
        
        background-color:#6BC5E8;
        padding:0 1vw 0 1vw;
        border-radius:2px;
        color:white;
        cursor: pointer !important;
    }

    .load_upload{
        cursor: pointer !important;
    }

    .import-button{
        background-color:#6BC5E8;
        padding:0 1vw 0 1vw;
        border-radius:2px;
        color:white;
        cursor: pointer !important;
    }

    .importing{
        background-color:#FFD670;
    }

    


    .import-container{
        display:flex;
        flex-direction: column;
        justify-content:space-between;
        align-items:center;
        width:30vw;
        height:40vh;
        margin: 2vh 0 0 0;

        /* border:1px solid #f8f8f8; */
        border-radius:2px;
        
        background-color:white;
        /* padding: 0 1vw 1vh 1vw;       */
    }

    .file-list{
        height: 30vh;
        width:100%;
        overflow:auto;
        display:flex;
        flex-direction:column;
        justify-content:flex-start;
        align-items:flex-start;
        padding-left: 1vw;
        color:#7a7a7b;
        
    }

    .container-heading{
        width:100%;
        border-bottom:1px solid #f8f8f8;
        background-color:#f8f8f8;
        color: #1ca6db;
        font-size:1.2em;
        /* font-weight:bold; */
    }

    .separator{
        margin: 0 2vw 0 2vw;
        font-size: 2em;
    }

    .available-container{
        justify-content: space-around;
        display:flex;
        flex-direction: column;
        justify-content:space-between;
        align-items:center;
        width:30vw;
        height:40vh;
        margin: 2vh 0 0 0;

        /* border:1px solid grey; */

        border-radius:2px;
    }

    .load-files-list-container {
        display: flex;
        flex-direction:column;
        justify-content: flex-start;
        align-items:flex-start;
        width: 100%;
        height:100%;
        overflow:auto;
        /* background-color:#6BC5E8; */
        background-color:white;
        color:#7a7a7b;
        padding-top:0.5vh;
    }

    .load-files-list-item{
        padding-left:1vw;
    }

    .hoverable{
        cursor:pointer !important;
        display:flex;
        align-items:center;
        height:100%;

    }

    button{
        cursor:pointer;
    }

    
</style>
