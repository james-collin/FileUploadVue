<script setup>
import { ref } from 'vue';
import { Upload } from "@aws-sdk/lib-storage";
import { S3Client } from "@aws-sdk/client-s3";

const startingCellType = ref('');
const targetCellType = ref('');
const cellFile = ref(null);
const selectedFileName = ref('');

const message = ref('');
const isLoading = ref(false);
const uploadProgress = ref(0);

const onFileChange = (event) => {
  if (event.target.files.length > 0) {
    cellFile.value = event.target.files[0];
    selectedFileName.value = event.target.files[0].name;
  }
};

function onFileDrop(event) {
  event.preventDefault();
  const files = event.dataTransfer.files;
  handleDroppedFiles(files);
}
function handleDroppedFiles(files) {
  if (files.length > 0) {
    cellFile.value = files[0];
    selectedFileName.value = files[0].name;
  }
}

function removeSelectedFile() {
  selectedFileName.value = '';
  cellFile.value = null;
}

const save = async () => {
  try {
    isLoading.value = true;

    await uploadToS3(cellFile.value);

    startingCellType.value = '';
    targetCellType.value = '';
    cellFile.value = null;
    selectedFileName.value = '';

    message.value = 'Files uploaded successfully!';

  } catch (error) {
    message.value = 'Error uploading files. Please try again.';
  }
  finally {
    isLoading.value = false;
  }
};


const uploadToS3 = async (file) => {
  const accessKeyId = 'AKIA47CRV2GAKBFDWMKS'
  const secret = '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';

  const s3 = new S3Client({
    credentials: {
      accessKeyId: accessKeyId,
      secretAccessKey: secret
    },
    region:"us-east-1"
  });


  const uniqueFileName = `${Date.now()}_${file.name}`;

  const params = {
    Bucket: 'ronakfileupload',
    Key: uniqueFileName,
    Body: file,
    ACL: "public-read"
  };

  const uploadPromise = new Upload({
      client: s3,
      params: params,
    });

  uploadPromise.on('httpUploadProgress', function (progress) {
    uploadProgress.value = (progress.loaded / progress.total) * 100;
  });

  try {
    const data = await uploadPromise.done();
    console.log('Upload completed. Location:', data.Location);
  } catch (err) {
    console.error(':', err);
  }
  finally{
    isLoading.value = false;
  }
};

</script>

<template>
  <div class="flex justify-center items-center h-screen bg-gray-100">
    <div class="bg-white p-8 rounded-lg shadow-lg max-w-lg w-full">
      <h1 class="text-2xl font-bold mb-4">File Upload</h1>
      <div class="mb-4">
        <div class="flex justify-between">
          <label for="startingCellType" class="block mb-2 me-2 text-lg font-bold">Starting cell type:</label>
          <select v-model="startingCellType"
            class="block px-4 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:border-indigo-500">
            <option value="" disabled selected hidden>Select an option</option>
            <option class="py-1">Stem</option>
            <option class="py-1">Endo</option>
            <option class="py-1">Meso</option>
          </select>
        </div>
        <div class="min-h-[30px] ml-2">
          <span v-if="startingCellType">{{ startingCellType.name }}</span>
        </div>
      </div>

      <div class="mb-4">
        <div class="flex justify-between">
          <label for="targetCellType" class="block mb-2 text-lg font-bold">Target cell type:</label>
          <select v-model="targetCellType"
            class="block px-4 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:border-indigo-500">
            <option value="" disabled selected hidden>Select an option</option>
            <option class="py-1">Stem</option>
            <option class="py-1">Endo</option>
            <option class="py-1">Meso</option>
          </select>

        </div>
        <div class="min-h-[30px] ml-2">
          <span v-if="targetCellType">{{ targetCellType.name }}</span>
        </div>
      </div>
      <div class="mb-4 border-dashed border-2 border-gray-400 p-4 rounded-lg text-center min-h-40" @dragover.prevent
        @drop="onFileDrop">

        <div v-if="selectedFileName">
          <span class="me-2">{{ selectedFileName }}</span>
          <button @click="removeSelectedFile"
            class="mt-2 bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded inline-block">
            Remove
          </button>
        </div>
        <div v-else>
          <p class="text-lg mb-2">Drop files here or</p>
          <label for="fileInput"
            class="cursor-pointer bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded inline-block">
            Click to Upload
          </label>
          <input type="file" id="fileInput" @change="onFileChange" class="hidden">
        </div>
      </div>
      <button @click="save" :disabled="!startingCellType || !targetCellType || !cellFile || isLoading"
        class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded w-full mt-10"
        :class="{ 'cursor-not-allowed opacity-50': !startingCellType || !targetCellType || !cellFile || isLoading }">Continue</button>

      <div v-if="isLoading" class="w-full bg-gray-200 rounded-full dark:bg-gray-700">
        <div class="bg-blue-600 text-xs font-medium text-blue-100 text-center p-0.5 leading-none rounded-full"
          :style="{ width: `${uploadProgress}%` }">{{ uploadProgress.toFixed(0) }}%</div>
      </div>
      <div v-if="message" class="mt-4 text-green-600">{{ message }}</div>
    </div>
  </div>
</template>

<style scoped>
.border-dashed {
  border-style: dashed;
}
</style>
