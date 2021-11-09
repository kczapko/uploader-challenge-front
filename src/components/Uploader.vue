<template lang="pug">
.uploader
  transition(name="zoom" mode="out-in" appear)
    section.uploader__upload(v-if="state === 'new'")
      .uploader__error(v-if="error") {{ error }}
      header.uploader__header 
        h2.uploader__heading Upload your image
        p.uploader__heading-text File should be jpg, png, webp. Max 10 MB
      .uploader__drop-zone(@dragover="$event.preventDefault(); overDropZone = true" @dragenter="overDropZone = true" @dragleave="overDropZone = false" @drop="upload" :class="{ 'uploader__drop-zone--over': overDropZone }")
        p.uploader__text Drag &amp; Drop your image here
      p.uploader__text Or
      .uploader__file-input-container
        input.uploader__file-input(type="file" accept="image/jpg,image/jpeg,image/png,image/webp" @change="upload")
        button.uploader__button Choose a file
    section.uploader__progress(v-else-if="state === 'uploading'")
      .uploader__error(v-if="error") {{ error }}
      header.uploader__header 
        h2.uploader__heading Uploading...
      .uploader__progress-bar
        .uploader__progress-bar__progress(:style="{width: `${progress}%`}")
    section.uploader__result(v-else-if="state === 'result'")
      header.uploader__header 
        svg.uploader__success-icon(xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24")
          path(d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z")
        h2.uploader__heading Uploaded Successfully!
      .uploader__image
        img.uploader__img(:src="filepath")
      .uploader__error(v-if="error") {{ error }}
      p.uploader__text(v-if="copyMessage") {{ copyMessage }}
      .uploader__copy
        input.uploader__input(type="text" :value="filepath" readonly ref="copyInput")
        button.uploader__button(@click="copyLink") Copy Link
</template>

<script>
import axios from 'axios';

export default {
  name: 'Uploader',
  data() {
    return {
      error: '',
      state: 'new',
      overDropZone: false,
      progress: 0,
      filepath: '',
      copyMessage: '',
    };
  },
  methods: {
    async upload(ev) {
      ev.preventDefault();

      const files = [...((ev.dataTransfer && ev.dataTransfer.files) || ev.target.files)];
      const filteredFiles = files.filter(
        file =>
          file.type === 'image/jpg' ||
          file.type === 'image/jpeg' ||
          file.type === 'image/png' ||
          file.type === 'image/webp'
      );
      const file = filteredFiles[0];

      if (file) {
        const formData = new FormData();
        formData.append('image', file);

        try {
          this.progress = 0;
          this.state = 'uploading';

          const res = await axios.post('', formData, {
            onUploadProgress: pEv => {
              this.progress = (pEv.loaded / pEv.total) * 100;
            },
          });

          this.filepath = res.data.filepath;
          this.state = 'result';
        } catch (err) {
          this.state = 'new';
          this.error = err.response ? err.response.data.message : err.message;
        }
      }
    },
    async copyLink() {
      try {
        await navigator.clipboard.writeText(this.filepath);
        this.copyMessage = 'Link copied to clipboard';
      } catch (err) {
        this.error = 'Unable to copy. Please copy manually';
        this.$refs.copyInput.select();
      }
    },
  },
};
</script>

<style lang="scss">
@mixin upload-section-base-layout() {
  background-color: #fff;
  width: calc(100vw - 4rem);
  max-width: 40rem;
  padding: 3.6rem 3.2rem;
  border-radius: 1.2rem;
  box-shadow: 0 0.4rem 1.2rem rgba(0, 0, 0, 0.1);
}

.uploader {
  margin: 2rem;

  &__error {
    text-align: center;
    color: #fff;
    background: red;
    padding: 0.5rem;
    margin-bottom: 1.5rem;
  }

  &__upload {
    @include upload-section-base-layout;
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  &__header {
    margin-bottom: 3rem;
  }

  &__heading {
    font-size: 1.8rem;
    color: #4f4f4f;
  }

  &__heading-text {
    font-size: 1rem;
    color: #828282;
    margin-top: 1.5rem;
  }

  &__drop-zone {
    height: 22rem;
    width: 100%;
    background: #f6f8fb url('/img/drop-zone.png') no-repeat 50% 3.5rem;
    border-radius: 1.2rem;
    border: 1px dashed #97bef4;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    padding: 2rem;

    &--over {
      background-color: darken(#f6f8fb, 5%);
    }
  }

  &__text {
    margin: 2rem 0;
    color: #bdbdbd;
    font-size: 1.2rem;
  }

  &__file-input-container {
    position: relative;
    overflow: hidden;
  }

  &__file-input {
    font-size: 10rem;
    opacity: 0;
    position: absolute;
    cursor: pointer;
  }

  &__file-input:hover + &__button {
    background: darken(#2f80ed, 10%);
  }

  &__button {
    border: 0;
    border-radius: 8px;
    background: #2f80ed;
    padding: 0.8rem 1.6rem;
    color: #fff;
    font-size: 1.2rem;
    cursor: pointer;

    &:hover {
      background: darken(#2f80ed, 10%);
    }
  }

  &__progress {
    @include upload-section-base-layout;
  }

  &__progress-bar {
    background: #f2f2f2;
    height: 0.6rem;
    position: relative;
    border-radius: 100vw;
    overflow: hidden;
    margin-bottom: 1rem;

    &__progress {
      position: absolute;
      left: 0;
      top: 0;
      height: 100%;
      background: #2f80ed;
      border-radius: 100vw;
      transition: all 250ms;
    }
  }

  &__result {
    @include upload-section-base-layout;
    text-align: center;
  }

  &__success-icon {
    fill: #219653;
    width: 35px;
  }

  &__image {
    border-radius: 1.2rem;
    overflow: hidden;
    margin-bottom: 2.5rem;
  }

  &__img {
    width: 100%;
    display: block;
  }

  &__copy {
    position: relative;
  }

  &__input {
    border: 1px solid #e0e0e0;
    background: #f6f8fb;
    border-radius: 0.8rem;
    height: 3.4rem;
    width: 100%;
    padding: 0 9rem 0 0.8rem;
    font-size: 0.8rem;
    line-height: 3.4rem;
    color: #4f4f4f;
    font-family: inherit;
  }

  &__copy &__button {
    position: absolute;
    right: 2px;
    top: 2px;
  }
}

.zoom-enter-from,
.zoom-leave-to {
  transform: scale3d(0, 0, 0);
  opacity: 0;
}

.zoom-enter-active {
  transition: all 200ms ease-out;
}

.zoom-leave-active {
  transition: all 200ms ease-in;
}
</style>
