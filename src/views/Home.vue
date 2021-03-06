<template>
  <div>
    <v-container fluid>
      <v-layout row wrap>
        <v-flex xs12 class="mb-5">
          <input type="file" accept="image/*" v-on:change="handleImageUpload" />

          <span v-if="isCompressing">
            COMPRESSING&nbsp;&nbsp;&nbsp;&nbsp;<v-progress-circular
              indeterminate
              color="red"
              :size="25"
            ></v-progress-circular>
          </span>
        </v-flex>
        <v-flex xs12 v-if="isReadyToDisplay">
          <div>
            <h1>Compressed Image:</h1>
            <div>
              <img :src="compressedFile" id="image" />
            </div>
            <div>
              <v-btn
                color="blue-grey"
                class="white--text"
                :href="compressedFile"
                :download="newFilename"
              >
                Download
                <v-icon right dark>cloud_download</v-icon>
              </v-btn>

              <div class="mt-2 ml-3">
                Original: {{ originalFileSize | friendlyBytes }}&nbsp;MB<br />
                Compressed:
                {{
                  compressedFileSize | friendlyBytes
                }}&nbsp;MB<br />Compression ratio (original / compressed):
                {{ compressionRatio }}
                <br /><br />

                <span style="font-weight: bold">{{ newFilename }}</span>
                <br /><br />
              </div>
            </div>
          </div>
        </v-flex>
      </v-layout>
    </v-container>
  </div>
</template>

<script>
import imageCompression from "browser-image-compression";
const blobToBase64 = require("blob-to-base64");
export default {
  components: {},
  mounted() {
    this.options = {
      maxSizeMB: 1,
      useWebWorker: true,
      maxIteration: 20
    };
  },
  data() {
    return {
      options: {},
      isCompressing: false,
      originalFileInfo: null,
      compressedFileInfo: null,
      isReadyToDisplay: false,
      compressedFile: null,
      compressedThumbnail: 500,
      originalSize: null,
      compressedSize: null,
      compressedFileName: null,
      compressionPercentage: null,
      compressedFileWidth: null,
      compressedFileHeight: null,
      filenameSuffix: "-min",
      newFilename: ""
    };
  },
  filters: {
    friendlyBytes: function(value) {
      let bytes = value / 1024 / 1024;
      return bytes.toFixed(2);
    }
  },
  methods: {
    slugify(text) {
      return text
        .toString()
        .toLowerCase()
        .replace(/\s+/g, "-") // Replace spaces with -

        .replace(/\-\-+/g, "-") // Replace multiple - with single -
        .replace(/^-+/, "") // Trim - from start of text
        .replace(/-+$/, ""); // Trim - from end of text
    },
    async handleImageUpload(event) {
      this.isReadyToDisplay = false;
      this.isCompressing = true;

      const imageFile = event.target.files[0];
      //console.log(imageFile);
      let filenameInfo = imageFile.name.split(".");
      let newFilename =
        filenameInfo[0] + this.filenameSuffix + "." + filenameInfo[1];

      this.newFilename = this.slugify(newFilename);
      //console.log(this.newFilename);
      this.originalFileSize = imageFile.size;
      try {
        const compressedFile = await imageCompression(imageFile, this.options);
        this.isCompressing = false;
        //console.log(imageFile);
        this.compressedFileSize = compressedFile.size;
        this.displayCompressedFile(compressedFile);
        this.compressionRatio = (imageFile.size / compressedFile.size).toFixed(
          2
        );
      } catch (error) {
        this.isCompressing = false;
        console.log(error);
      }
    },

    displayCompressedFile(file) {
      this.isReadyToDisplay = true;
      let vm = this;
      blobToBase64(file, function(error, base64) {
        if (!error) {
          vm.compressedFile = base64;
        }
      });
    }
  },
  computed: {}
};
</script>

<style>
img {
  width: 100%;
  height: auto;
  max-width: 100%;
}
</style>
