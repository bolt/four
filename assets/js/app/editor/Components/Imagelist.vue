<template>
  <div>
    <div
      v-for="(child, index) in containerImages"
      :key="child.id"
      class="form-fieldsgroup"
    >
      <editor-image
        v-if="child.hidden !== true"
        :filename="child.filename"
        :thumbnail="child.thumbnail"
        :title="child.title"
        :alt="child.alt"
        :attributes-link="attributesLink"
        :media="child.media"
        :directory="directory"
        :filelist="filelist"
        :csrf-token="csrfToken"
        :labels="labels"
        :in-imagelist="true"
        :name="fieldName(index)"
        :extensions="extensions"
        :is-first-in-imagelist="isFirstInImagelist(index)"
        :is-last-in-imagelist="isLastInImagelist(index)"
        @remove="onRemoveImage"
        @moveImageUp="onMoveImageUp"
        @moveImageDown="onMoveImageDown"
      ></editor-image>
    </div>

    <button class="btn btn-secondary" type="button" @click="addImage">
      {{ labels.add_new_image }}
    </button>
  </div>
</template>

<script>
import Image from './Image';

export default {
  name: 'EditorImage',
  components: { 'editor-image': Image },
  props: [
    'images',
    'directory',
    'name',
    'filelist',
    'csrfToken',
    'labels',
    'extensions',
    'attributesLink',
  ],
  data: function() {
    let counter = 0;
    let containerImages = this.images;
    containerImages.forEach(function(file, index, theContainerImagesArray) {
      theContainerImagesArray[index].id = index;
      counter++;
    });

    return {
      counter,
      containerImages: this.images,
    };
  },
  methods: {
    isFirstInImagelist(index) {
      return index === 0;
    },
    isLastInImagelist(index) {
      return index === this.getActiveImageFields().length - 1;
    },
    getActiveImageFields() {
      return this.containerImages.filter(function(image) {
        return image.hidden !== true;
      });
    },
    getFieldNumberFromElement(elem) {
      return parseInt(elem.fieldName.match(/\d+/)[0]);
    },
    onMoveImageDown(elem) {
      let fieldNumber = this.getFieldNumberFromElement(elem);

      if (fieldNumber < this.containerImages.length - 1) {
        let imageToMoveDown = this.containerImages[fieldNumber];
        let imageToMoveUp = this.containerImages[fieldNumber + 1];

        this.containerImages.splice(
          fieldNumber,
          2,
          imageToMoveUp,
          imageToMoveDown,
        );
      }
    },
    onMoveImageUp(elem) {
      let fieldNumber = this.getFieldNumberFromElement(elem);

      if (fieldNumber > 0) {
        let imageToMoveUp = this.containerImages[fieldNumber];
        let imageToMoveDown = this.containerImages[fieldNumber - 1];

        this.containerImages.splice(
          fieldNumber - 1,
          2,
          imageToMoveUp,
          imageToMoveDown,
        );
      }
    },
    onRemoveImage(elem) {
      let fieldNumber = this.getFieldNumberFromElement(elem);
      let updatedImage = this.containerImages[fieldNumber];
      updatedImage.hidden = true;
      this.$set(this.containerImages, fieldNumber, updatedImage);

      if (this.getActiveImageFields().length === 0) {
        this.addImage();
      }
    },
    fieldName(index) {
      return this.name + '[' + index + ']';
    },
    addImage() {
      let imageField = {
        inImagelist: true,
        directory: this.directory,
        name: this.name,
        filelist: this.filelist,
        csrfToken: this.csrfToken,
        labels: this.labels,
        thumbnail: '',
        extensions: this.extensions,
        id: this.counter,
      };

      this.counter++;
      this.containerImages.push(imageField);
    },
  },
};
</script>