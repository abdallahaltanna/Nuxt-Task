<template>
  <div class="pt-[88px] pb-[140px]">
    <div class="container mx-auto">
      <div class="border-b border-b-[#9C9C9C]">
        <nav class="flex justify-between h-[56px] items-center">
          <span class="text-[20px] font-bold">Gallery</span>
          <div>
            <button
              type="button"
              class="w-[106px] h-[44px] bg-[#72A8CA] rounded-[24px]"
              @click="triggerFileInput"
            >
              Upload
            </button>
            <input
              type="file"
              @change="handleFileChange"
              multiple
              hidden
              ref="fileInput"
            />
          </div>
        </nav>
      </div>

      <div
        class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 xl:grid-cols-4 gap-3 pt-[96px]"
      >
        <div
          v-for="(image, index) in uploadedImages"
          :key="index"
          class="image-container h-[520px] relative"
        >
          <button
            @click="deleteImage(index)"
            class="close-btn w-[25px] h-[25px] rounded-full flex justify-center items-center bg-white absolute top-[15px] right-[15px]"
          >
            <Icon name="ic:baseline-close" color="black" />
          </button>
          <img
            :src="image"
            alt="Uploaded Image"
            class="object-cover w-full h-full"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import axios from 'axios';

const uploadedImages = ref<string[]>([]);
const fileInput = ref<HTMLInputElement | null>(null);

const triggerFileInput = () => {
  fileInput.value?.click();
};

const handleFileChange = async (event: Event) => {
  const files = (event.target as HTMLInputElement).files;
  if (!files) return;

  for (let file of files) {
    if (file.size > 2097152) {
      alert('File size should not exceed 2MB');
      return;
    }

    if (!file.type.startsWith('image/')) {
      alert('Invalid file type');
      return;
    }

    const reader = new FileReader();
    reader.onload = async e => {
      const base64Image = e.target?.result;
      if (base64Image) {
        uploadedImages.value.push(base64Image as string);
        // Upload the image to the server
        await uploadImageToServer(base64Image as string);
      }
    };
    reader.readAsDataURL(file);
  }
};

const uploadImageToServer = async (base64Image: string) => {
  try {
    const response = await axios.post(
      'https://vintrackers.buildonlinestaging.com/upload/images/',
      { images: base64Image }
    );
    console.log('Upload successful', response.data);
  } catch (error) {
    console.error('Upload failed', error);
    alert('Failed to upload image');
  }
};

const deleteImage = (index: number) => {
  uploadedImages.value.splice(index, 1);
};
</script>

<style scoped>
.image-container {
  transition: opacity 0.3s;
}
.image-container .close-btn {
  transition: opacity 0.3s ease;
  opacity: 0;
  visibility: hidden;
}
.image-container:hover .close-btn {
  opacity: 1;
  visibility: visible;
  transition-delay: 0s, 0.3s;
}
</style>
