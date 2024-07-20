<template>
  <v-container>
    <v-row>
      <v-col>
        <h1 class="mb-3 mt-3">File Upload</h1>
        <v-form @submit.prevent="uploadFile">
          <v-file-input
            v-model="file"
            label="Choose file"
            prepend-icon="mdi-paperclip"
            outlined
          ></v-file-input>
          <v-btn type="submit" color="primary" :loading="isUploading">
            Upload
          </v-btn>
        </v-form>
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <h2 class="mb-3 mt-3">Uploaded Files</h2>
        <v-alert
          v-if="files.length === 0"
          border="top"
          type="warning"
          variant="outlined"
        >
          You haven't uploaded any files. When you do, it will appear here.
        </v-alert>
        <v-list v-else>
          <v-list-item v-for="(file, index) in files" :key="index">
            <v-list-item-content>
              <v-list-item-title>{{ file.name }}</v-list-item-title>
            </v-list-item-content>
            <v-list-item-action>
              <v-btn icon @click="deleteFile(file.name)">
                <v-icon color="error">mdi-delete</v-icon>
              </v-btn>
            </v-list-item-action>
          </v-list-item>
        </v-list>
      </v-col>
    </v-row>

    <v-snackbar
      v-model="snackbar"
      :timeout="2000"
      :color="snackbarColor"
      top
      center
      class="mt-12"
    >
      {{ snackbarText }}
      <template v-slot:action="{ attrs }">
        <v-btn color="white" text v-bind="attrs" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      file: null,
      files: [],
      snackbar: false,
      snackbarColor: 'success',
      snackbarText: '',
      isUploading: false,
    }
  },
  mounted() {
    this.getUploadedFiles()
  },
  methods: {
    async getUploadedFiles() {
      try {
        const result = await this.$axios.get('/api/files')
        this.files = result.data.files
      } catch (error) {
        this.files = []
      }
    },
    async uploadFile() {
      if (!this.file) {
        this.snackbarColor = 'error'
        this.snackbar = true
        this.snackbarText = 'Please choose file before uploading'
        return
      }

      const formData = new FormData()
      formData.append('file', this.file)
      this.isUploading = true

      try {
        await this.$axios.post('/api/files', formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
          },
        })

        this.snackbarText = 'File uploaded successfully!'
        this.snackbarColor = 'success'
        this.snackbar = true

        this.getUploadedFiles()
      } catch (error) {
        this.snackbarText = 'Failed to upload file'
        if (error.response?.data?.error) {
          this.snackbarText =
            this.snackbarText + ' : ' + error.response.data.error
        }
        this.snackbarColor = 'error'
        this.snackbar = true
      } finally {
        this.file = null // Reset the input after upload
        this.isUploading = false
      }
    },
    async deleteFile(filename) {
      try {
        await this.$axios.delete('/api/files/' + filename)

        this.getUploadedFiles()
      } catch (error) {
        this.snackbarText = 'Failed to delete file'
        if (error.response?.data?.error) {
          this.snackbarText =
            this.snackbarText + ' : ' + error.response.data.error
        }
        this.snackbarColor = 'error'
        this.snackbar = true
      }
    },
  },
}
</script>
