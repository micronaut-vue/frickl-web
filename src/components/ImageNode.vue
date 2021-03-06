<template>
  <b-card no-body :class="`image-card ${imageDetailsMode}`">
    <router-link :to="'/images/' + image.id">
      <div class="card-img-wrap">
        <img :src="getSrc('SMALL')" class="card-img" :style="'height:' + imageHeight + 'px'"/>
      </div>
    </router-link>
    <div class="card-img-overlay flex-column justify-content-end" v-if="imageDetailsMode === 'overlay'">
      <div class="image-actions">
        <div class="mb-2">{{ image.name }}</div>
        <div><small class="text-light" v-if="image.exif && (image.exif.dateTimeOriginal || image.exif.dateTime)"><CalendarClockIcon class="mdi-sm" /> {{ getTime() | toDateTime }}</small></div>
        <div><small class="text-light" v-if="image.exif && (image.exif.cameraMake || image.exif.cameraModel)"><CameraIcon class="mdi-sm" /> <span v-if="image.exif.cameraMake"> {{ image.exif.cameraMake }}</span><span v-if="image.exif.cameraModel"> {{ image.exif.cameraModel }}</span></small></div>
        <div class="mt-2">
          <span v-if="authEnabled === false || token">
            <HeartIcon v-b-tooltip.hover.bottom="'Mark as favourite'" v-if="image.isFavorite" @click="onToggleFavorite($event)"/>
            <HeartOutlineIcon v-b-tooltip.hover.bottom="'Unmark as favourite'" v-else  @click="onToggleFavorite($event)"/>
            <template v-if="authEnabled === true && token">
              <LockOpenVariantIcon v-b-tooltip.hover.bottom="'Make private'" @click="onTogglePublic($event)" v-if="image.isPublic === 1" />
              <LockIcon v-b-tooltip.hover.bottom="'Make public'" @click="onTogglePublic($event)" v-else />
            </template>
            <FolderImageIcon v-b-tooltip.hover.bottom="'Set image as album cover'" @click="onSetImageAsAlbumCover($event)" v-if="albumId"/>
          </span>
          <a class="baguettebox" :href="getSrc('ORIGINAL')" v-b-tooltip.hover.bottom="'Open large preview'" @click.stop.prevent><MagnifyPlusIcon /></a>
        </div>
      </div>
    </div>
    <div v-else class="card-image-details">
      <b-card-body>
        <div>
          <div class="mb-2">{{ image.name }}</div>
          <div><small class="text-muted" v-if="image.exif && (image.exif.dateTimeOriginal || image.exif.dateTime)"><CalendarClockIcon class="text-muted mdi-sm" /> {{ getTime() | toDateTime }}</small></div>
          <div><small class="text-muted" v-if="image.exif && (image.exif.cameraMake || image.exif.cameraModel)"><CameraIcon class="mdi-sm" /> <span v-if="image.exif.cameraMake"> {{ image.exif.cameraMake }}</span><span v-if="image.exif.cameraModel"> {{ image.exif.cameraModel }}</span></small></div>
        </div>
      </b-card-body>

      <b-button-group class="image-actions mt-2 w-100">
        <template v-if="authEnabled === false || token">
          <b-button v-b-tooltip.hover.bottom="'Mark as favourite'" v-if="image.isFavorite" @click="onToggleFavorite($event)"><HeartIcon /></b-button>
          <b-button v-b-tooltip.hover.bottom="'Unmark as favourite'" v-else  @click="onToggleFavorite($event)"><HeartOutlineIcon /></b-button>
          <template v-if="authEnabled === true && token">
            <b-button v-b-tooltip.hover.bottom="'Make private'" @click="onTogglePublic($event)" v-if="image.isPublic === 1"><LockOpenVariantIcon /></b-button>
            <b-button v-b-tooltip.hover.bottom="'Make public'" @click="onTogglePublic($event)" v-else><LockIcon /></b-button>
          </template>
          <b-button v-b-tooltip.hover.bottom="'Set image as album cover'" @click="onSetImageAsAlbumCover($event)" v-if="albumId"><FolderImageIcon/></b-button>
        </template>
        <b-button class="baguettebox" :href="getSrc('ORIGINAL')" v-b-tooltip.hover.bottom="'Open large preview'" @click.stop.prevent><MagnifyPlusIcon /></b-button>
      </b-button-group>
    </div>
  </b-card>
</template>

<script>
import CalendarClockIcon from 'vue-material-design-icons/CalendarClock.vue'
import CameraIcon from 'vue-material-design-icons/Camera.vue'
import HeartIcon from 'vue-material-design-icons/Heart.vue'
import HeartOutlineIcon from 'vue-material-design-icons/HeartOutline.vue'
import FolderImageIcon from 'vue-material-design-icons/FolderImage.vue'
import LockIcon from 'vue-material-design-icons/Lock.vue'
import LockOpenVariantIcon from 'vue-material-design-icons/LockOpenVariant.vue'
import MagnifyPlusIcon from 'vue-material-design-icons/MagnifyPlus.vue'

import { mapGetters } from 'vuex'

export default {
  data: function () {
    return {
    }
  },
  computed: {
    ...mapGetters([
      'authEnabled',
      'imageDetailsMode',
      'baseUrl',
      'token'
    ])
  },
  props: {
    image: {
      type: Object,
      default: null
    },
    albumId: {
      type: String,
      default: null
    },
    imageHeight: {
      type: Number,
      default: 300
    }
  },
  components: {
    CalendarClockIcon,
    CameraIcon,
    HeartIcon,
    HeartOutlineIcon,
    FolderImageIcon,
    LockIcon,
    LockOpenVariantIcon,
    MagnifyPlusIcon
  },
  methods: {
    getSrc: function (size) {
      var result = `${this.baseUrl}image/${this.image.id}/img?size=${size}`

      if (this.token && this.token.imageToken) {
        result = `${result}&token=${this.token.imageToken}`
      }

      return result
    },
    getTime: function () {
      if (this.image.exif.dateTime) {
        return this.image.exif.dateTime
      } else if (this.image.exif.dateTimeOriginal) {
        return this.image.exif.dateTimeOriginal
      } else {
        return ''
      }
    },
    onSetImageAsAlbumCover: function (event) {
      event.stopPropagation()
      event.preventDefault()

      var vm = this
      this.$bvModal.msgBoxConfirm('Set image as album cover?', {
        title: 'Album cover',
        okTitle: 'Yes',
        okVariant: 'success',
        cancelTitle: 'No',
        cancelVariant: 'secondary'
      })
        .then(value => {
          if (value) {
            const album = {
              id: vm.albumId,
              bannerImageId: vm.image.id
            }

            vm.apiPatchAlbum(album, function (result) {
              vm.$bvToast.toast('Image set as album cover.', {
                title: 'Success',
                autoHideDelay: 5000,
                appendToast: true
              })
            })
          }
        })
        .catch(err => {
          console.error(err)
        })
    },
    onTogglePublic: function (event) {
      event.stopPropagation()
      event.preventDefault()
      this.image.isPublic = Math.abs(this.image.isPublic - 1)

      this.apiPatchImage(this.image)
    },
    onToggleFavorite: function (event) {
      event.stopPropagation()
      event.preventDefault()
      this.image.isFavorite = Math.abs(this.image.isFavorite - 1)

      this.apiPatchImage(this.image)
    }
  }
}
</script>

<style>
  .image-card .card-img {
    width: 100%;
    object-fit: cover;
    transition: transform .2s ease-in-out;
  }
  .image-card.below .card-img {
    border-bottom-left-radius: 0;
    border-bottom-right-radius: 0;
  }
  .image-card .mdi-sm,
  .image-card .card-img-overlay .image-actions .material-design-icon {
    margin-right: 0.75rem;
  }
  .image-card .card-img-overlay {
    padding: 0;
    top: inherit;
  }
  .image-card .card-img-overlay .material-design-icon {
    height: 1.5rem;
    width: 1.5rem;
    opacity: 0;
    transition: opacity .2s ease-in-out;
  }
  .image-card .card-image-details .material-design-icon {
    height: 1.5rem;
    width: 1.5rem;
  }
  .image-actions svg:hover {
    cursor: pointer;
  }
  .image-card .card-img-overlay .image-actions {
    opacity: 0;
    padding: 1rem;
    background-color: rgba(1,1,1,.5);
    margin:0 !important;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: white;
    transition: opacity .2s ease-in-out;
  }
  .image-card .card-image-details .image-actions {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  .image-card .card-image-details .image-actions .btn {
    border-top-left-radius: 0;
    border-top-right-radius: 0;
  }
  .image-card .card-img-overlay .image-actions > div {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    font-size: 11pt;
  }
  .image-card:hover .card-img-overlay .image-actions {
    opacity: 1;
  }
  .image-card .card-img-overlay .material-design-icon.heart-icon {
    opacity: 1;
  }
  .image-card:hover .card-img-overlay .material-design-icon {
    opacity: 1;
  }
  .image-card .card-img-overlay .material-design-icon > .material-design-icon__svg {
    height: 1.5rem;
    width: 1.5rem;
    fill: white;
  }
  .image-card .card-image-details .material-design-icon > .material-design-icon__svg {
    height: 1.2rem;
    width: 1.2rem;
    fill: white;
  }
  .image-card .card-image-details .card-body .material-design-icon > .material-design-icon__svg {
    fill: #6c757d;
  }
  .image-card .card-img-overlay .material-design-icon.heart-icon > .material-design-icon__svg,
  .image-card .card-img-overlay .material-design-icon.heart-outline-icon:hover > .material-design-icon__svg,
  .image-card .card-image-details .material-design-icon.heart-icon > .material-design-icon__svg,
  .image-card .card-image-details .material-design-icon.heart-outline-icon:hover > .material-design-icon__svg {
      fill: #EA2027;
  }
  .image-card .image-actions .material-design-icon.folder-image-icon:hover > .material-design-icon__svg,
  .image-card .card-image-details .material-design-icon.folder-image-icon:hover > .material-design-icon__svg {
      fill: #FFC312;
  }
  .image-card .image-actions .material-design-icon.magnify-plus-icon:hover > .material-design-icon__svg,
  .image-card .card-image-details .material-design-icon.magnify-plus-icon:hover > .material-design-icon__svg {
      fill: #A3CB38;
  }
  .image-card .card-img-overlay .material-design-icon.lock-icon:hover > .material-design-icon__svg,
  .image-card .card-img-overlay .material-design-icon.lock-open-variant-icon > .material-design-icon__svg,
  .image-card .card-image-details .material-design-icon.lock-icon:hover > .material-design-icon__svg,
  .image-card .card-image-details .material-design-icon.lock-open-variant-icon > .material-design-icon__svg {
      fill: #3498db;
  }

  .col-xxl-1 .card-body {
    padding: 1rem;
  }
  .image-card .mdi-sm.material-design-icon,
  .image-card .mdi-sm.material-design-icon .material-design-icon__svg {
    height: 1rem;
    width: 1rem;
  }
</style>
